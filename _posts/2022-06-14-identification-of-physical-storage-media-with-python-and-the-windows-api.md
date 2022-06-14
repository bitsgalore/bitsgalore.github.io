---
layout: post
title: Identification of physical storage media and devices with Python and the Windows API
tags: [disk-imaging, floppy-disks, optical-media]
comment_id: 80
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2022/06/storage-media.jpg" alt="Still life of assorted storage media">
</figure>

This blog post covers some techniques that can be used to identify storage media and storage devices using Python and the Windows API. This can be useful for distinguishing between different types of portable storage media, such as floppy disks and USB thumb drives. It also presents a demo script that integrates these techniques.

<!-- more -->

## Preservation of portable storage media

In 2019 the KB started a [major initiative](https://www.kb.nl/over-ons/projecten/digitalisering-optische-dragers) to safeguard the information on optical data carriers in its collection, such as CD-ROMs, DVDs and audio CDs. The [Iromlab](https://github.com/KBNLresearch/iromlab) software is a central component of the workflow we're using for this. The optical media preservation project will most likely reach its completion near the end of this year. As the KB collection also contains many other types of portable digital storage media (see [this blog post for an overview]({{ BASE_PATH }}/2020/02/20/offline-digital-carriers-kb-deposit-collection)), we're currently looking into ways to preserve those as well. The first candidate will be 3.5 inch floppy disks. I'm currently working on a derivative of the *Iromlab* software that can be used for imaging floppies, but also other types of portable media, such as USB thumb drives. Like *Iromlab*, it wraps around [IsoBuster](https://www.isobuster.com/) to do the actual imaging. Since I'd like the new software to be able to deal with a variety of physical storage media types and devices, it would be useful to have a way to automatically identify the medium type prior to the imaging stage. On Windows (which is the target environment for this work), this kind of hardware identification is possible through the [Win32 API](https://docs.microsoft.com/en-us/windows/win32/api/).

## Challenges

In Python, parts of the Win32 API can be accessed using the [pywin32](https://github.com/mhammond/pywin32) wrapper extensions, but using these extensions can be a bit of a challenge.There are a couple of reasons for this. First, the pywin32 documentation is incomplete and partially outdated. This means you'll often have to rely on Microsoft's documentation of the underlying C++ API. Using the API also involves some fairly low-level operations, such as creating file handles, defining output buffers and parsing binary output. Many of the (few) relevant code examples that can be found online are also outdated, which meant I had to combine examples and documentation from a variety of sources in order to get things working. Taken together, this all makes working with the Win32 API pretty daunting.

## Purpose of this post

In this post I'll try to document how I made basic media and device identification work for me. Given any device attached to a logical Windows drive (e.g. the "A drive", "D drive", etcetera), the objectives here are to identify:

1. the storage media type (i.e. hard disk, floppy drive, etc.);
2. the hardware device, and the storage media types that are supported by it.

I'll link to the relevant documentation throughout. At the end of this post I also present a simple demo script that ties everything together, and which could be used as a basis for your own code.

## Preparation

In order to use the techniques described here, you'll [pywin32](https://github.com/mhammond/pywin32), which you can install using:

```bash
pip install pywin32
```

Then create a new Python file, and add the following imports:

```python
import sys
import struct
import argparse
import win32api
import win32file
import winioctlcon
```

## Device name

First of all we need the device name that corresponds to the logical drive we're interested in (e.g. drive A, C, D, and so on). For this we use (using the "A" drive as an example here):

```python
drive="A"
# Low-level device name of device assigned to logical drive
driveDevice =  "\\\\.\\" + drive + ":"
```

## Create file handle

Next we create a file handle for this device, using the [win32file.CreateFile](http://timgolden.me.uk/pywin32-docs/win32file__CreateFile_meth.html) method:

```python
handle = win32file.CreateFile(driveDevice,
                              0,
                              win32file.FILE_SHARE_READ,
                              None,
                              win32file.OPEN_EXISTING,
                              0,
                              None)
```

## Retrieve disk geometry info

We can now use the [win32file.DeviceIoControl](http://timgolden.me.uk/pywin32-docs/win32file__DeviceIoControl_meth.html) function to obtain information about a physical disk:

```python
diskGeometry = win32file.DeviceIoControl(handle,
                            winioctlcon.IOCTL_DISK_GET_DRIVE_GEOMETRY,
                            None,
                            24)
```

In this case, the *DeviceIoControl* call contains four arguments:

- the file handle for the device;
- the control code `winioctlcon.IOCTL_DISK_GET_DRIVE_GEOMETRY` (documented [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ni-winioctl-ioctl_disk_get_drive_geometry)), which tells *DeviceIoControl* to retrieves information about a physical disk's geometry;
- an input buffer, which is set to `None` in this case because it is not used;
- the output buffer size in bytes. 

The output buffer size must be equal to (or larger than) the output that is returned by the function call. The output (a string of raw bytes) is defined by the `DISK_GEOMETRY` structure, which is [documented here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ns-winioctl-disk_geometry). It contains 5 fields. The first field is a [large integer](https://docs.microsoft.com/en-us/windows/win32/api/winnt/ns-winnt-large_integer-r1) (8 bytes); the remaining fields are all [4-byte unsigned integers](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/262627d8-3418-4627-9218-4ffe110850b2). This means the total size of the `DISK_GEOMETRY` structure is 24 bytes, so we use this as the buffer size.

### Parsing DISK_GEOMETRY

The *MediaType* field is the second item of the `DISK_GEOMETRY` structure. It is an unsigned integer (4 bytes) that starts at byte offset 8 of our *diskGeometry* variable. We can use Python's [struct module](https://docs.python.org/3/library/struct.html) to interpret the raw bytes into an integer value:

```python
offset = 8
mediaTypeCode = struct.unpack("<I", diskGeometry[offset:offset + 4])[0]
```

Here the "`<I`" format string informs unpack that the bytes represent a little-Endian unsigned integer. Note that `struct.unpack` always returns a tuple, hence the "`\[0\]`" index at the end.

The resulting *mediaTypeCode* value is an integer number. These numbers can be mapped back to media type strings using the *MEDIA_TYPE* enumeration in [winioctlcon.py](https://github.com/mhammond/pywin32/blob/main/win32/Lib/winioctlcon.py). These strings are in turn documented [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ne-winioctl-media_type).

## Additional device info

Although the above method already allows us to identify, as an example, many types of floppy disks, it cannot distinguish between a USB thumb drive and a CD-ROM drive, both of which are simply identified as "RemovableMedia". For some more granularity, we can call [win32file.DeviceIoControl](http://timgolden.me.uk/pywin32-docs/win32file__DeviceIoControl_meth.html) with the `IOCTL_STORAGE_GET_MEDIA_TYPES_EX` control code (documented [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ni-winioctl-ioctl_storage_get_media_types_ex)):


```python
getMediaTypes = win32file.DeviceIoControl(handle,
                            winioctlcon.IOCTL_STORAGE_GET_MEDIA_TYPES_EX,
                            None,
                            2048)
```

The function arguments are largely identical to the earlier (disk geometry) call, but this time we use a 2048 byte output buffer. This is a somewhat arbitrary value. Unlike `IOCTL_DISK_GET_DRIVE_GEOMETRY`, the output size of `IOCTL_STORAGE_GET_MEDIA_TYPES_EX` is not fixed (see also the explanation that follows below), so I'm simply using a fairly large value to ensure the buffer will be large enough to fit the output under all circumstances. The output (again a string of raw bytes) is defined by the `GET_MEDIA_TYPES` structure, which is [documented here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ns-winioctl-get_media_types).

### Parsing GET_MEDIA_TYPES

The `GET_MEDIA_TYPES` structure is made up of the following fields:

1. a 4-byte unsigned integer that represents a device type code;
2. another 4-byte unsigned integer that represents the number of `DEVICE_MEDIA_INFO` structures
3. a pointer to the first `DEVICE_MEDIA_INFO` structure.

We can read the device type code and the number of `DEVICE_MEDIA_INFO` structures like this:

```python
deviceCode = struct.unpack("<I", getMediaTypes[0:4])[0]
mediaInfoCount = struct.unpack("<I", getMediaTypes[4:8])[0]
```

The resulting *deviceCode* value is an integer number, which again can be mapped back to a device type string using an enumeration in [winioctlcon.py](https://github.com/mhammond/pywin32/blob/main/win32/Lib/winioctlcon.py). The value of *mediaInfoCount* tells us the number of `DEVICE_MEDIA_INFO` structures we need to parse.

### Parsing DEVICE_MEDIA_INFO

The `DEVICE_MEDIA_INFO` structure itself is documented [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ns-winioctl-device_media_info). At first sight this may look a little intimidating, but essentially it just describes a "union" of 3 possible 32-byte structures. This means that each `DEVICE_MEDIA_INFO` instance is made up of either of those structures. For the purpose of this post, we're only interested in the *MediaType* field here, and this field can be at an identical location (the second item of the `DEVICE_MEDIA_INFO` structure) for two of the three possible variants. Only in case of a tape device, *MediaType* is the first item. Tape devices can be identified from the value of *deviceCode*. Using this information, we can use the code below to iterate over all `DEVICE_MEDIA_INFO` structures, and extract their respective media type codes: 

```python
# Start position in GET_MEDIA_TYPES structure
# (remember we already read two 4-byte integers from it, 
# hence the 8 byte start offset!)
offset = 8

# Loop over DEVICE_MEDIA_INFO structures
for _ in range(mediaInfoCount):
    if deviceCode in [31, 32]:
        # Tape device, mediaTypeCode is first item
        mediaTypeCode = struct.unpack("<I",
                          getMediaTypes[offset:offset + 4])[0]
        offset +=8
    else:
        # Not a tape device, so skip 8 byte cylinders value
        offset += 8
        mediaTypeCode = struct.unpack("<I",
                          getMediaTypes[offset:offset + 4])[0]
```

The resulting *mediaTypeCode* values are a integer numbers that can be mapped back to media type strings using the *MEDIA_TYPE* and *STORAGE_MEDIA_TYPE* enumerations in [winioctlcon.py](https://github.com/mhammond/pywin32/blob/main/win32/Lib/winioctlcon.py). These strings are in turn documented [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ne-winioctl-media_type) and [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ne-winioctl-storage_media_type).

## Putting it all together

I created a simple [demo script](https://github.com/KBNLresearch/detectStorageMediaType/blob/main/detectStorageMediaType.py) that ties everything discussed here together. It also contains lookup functions that translate the media type and device codes into human-readable strings. You can run it from the command prompt with one or more logical drive names a command-line arguments. For example:

```
python detectStorageMediaType.py A D E
```

The script then tries to establish the media type (from the disk geometry), the device type and the media types that are supported by the device, and reports the results back like this:

```
Drive:                   A
Media type:              F3_1Pt44_512

Drive:                   D
Media type:              RemovableMedia
Device type:             FILE_DEVICE_CD_ROM
Supported media types:
                         CD_ROM
                         RemovableMedia

Drive:                   E
Media type:              RemovableMedia
Device type:             FILE_DEVICE_DISK
Supported media types:
                         RemovableMedia
```

You may note that the output is incomplete in some cases, which is because the API calls do not work for all devices. In the above example, the `IOCTL_STORAGE_GET_MEDIA_TYPES_EX` control code could not be used to access the floppy drive attached to logical drive *A*, so no device output is produced for that drive.

This example also showcases the level of detail that is reported for floppy disks. As can be seen [here](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ne-winioctl-media_type), the code `F3_1Pt44_512` indicates a 3.5" floppy disk, with 1.44MB and 512 bytes per sector.

Finally, the logical drive *D* in the above example was a virtual optical drive with an ISO image of a DVD-ROM attached to it. Despite that, the device is identified as "FILE_DEVICE_CD_ROM" (not "FILE_DEVICE_DVD"!), and DVDs are not listed as a supported media type. This could be a limitation of my test approach, which was based on Windows 10 running in VirtualBox inside a Linux host environment[^1]. This would need additional testing with physical hardware.

## Final note

This post only scratches the surface of what's possible with the Windows API, but hopefully it will be useful to others to start their own explorations. I've only tested the code presented here with a limited number of devices (both physical and virtual ones) attached to a virtual machine running Windows 10. If you run into any surprises, or have suggestions for improvements, feel free to leave a comment here, or open a pull request for the demo script. 

## Further resources

- [Demo script](https://github.com/KBNLresearch/detectStorageMediaType)
- [PyWin32 Documentation](http://timgolden.me.uk/pywin32-docs/)
- [Programming reference for the Win32 API (Microsoft)](https://docs.microsoft.com/en-us/windows/win32/api/)

[^1]: For reasons unknown to me I was unable to attach a physical DVD drive to this virtual machine for my tests.