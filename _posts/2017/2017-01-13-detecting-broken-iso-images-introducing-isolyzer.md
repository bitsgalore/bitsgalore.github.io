---
layout: post
title: Detecting broken ISO images&#58; introducing Isolyzer
tags: [isolyzer,optical-media,ISO-9660]
comment_id: 25
---

In my [previous blog post]({{ BASE_PATH }}/2017/01/04/breaking-waves-and-some-flacs) I addressed the detection of broken audio files in an automated workflow for ripping audio CDs. For (data) CD-ROMs and DVDs that are imaged to an ISO image, a similar problem exists: how can we be reasonably sure that the created image is complete? In this blog post I will discuss some possible ways of doing this using existing tools, along with their limitations. I then introduce *Isolyzer*, a new tool that might be a useful addition to the existing methods.

<!-- more -->

## Checksums

A  number of techniques exist to verify a newly created ISO image. A seemingly obvious solution would be to do a checksum comparison on both the ISO image and the physical carrier. For instance, the following will work on any Linux system:

```
md5sum myimage.iso
md5sum /dev/sr0
```

The first line computes an MD5 checksum from the ISO image; the second line repeats this for the physical carrier. This method is not completely fail-safe. In some tests I did over a year ago, I ran into a a very strange issue where my attempts to image a CD would sometimes [result in incomplete reads](http://qanda.digipres.org/1076/incomplete-image-after-imaging-rom-prevent-and-detect-this), and, as a result, truncated ISO images. The problem was most likely caused by faulty hardware (the machine on which I ran those tests more or less died shortly afterwards). Most worryingly, the machine would sometimes return incomplete data, both while creating the ISO image as well as during the subsequent checksum calculation on the physical carrier. The result of this was that the computed checksums were identical in both cases, *which meant that the image passed the checksum quality check, even though it was incomplete*!

## Isovfy

The popular [*cdrtools*](https://en.wikipedia.org/wiki/Cdrtools) library includes a tool called [*isovfy*](http://linux.die.net/man/8/isoinfo). Its man page describes it as follows:

> isovfy is a utility to verify the integrity of an iso9660 image. Most of the tests in isovfy were added after bugs were discovered in early versions of mkisofs. It isn't all that clear how useful this is anymore, but it doesn't hurt to have this around. 

I already commented on this tool in an [earlier blog post]({{ BASE_PATH }}/2015/11/13/preserving-optical-media-from-the-command-line/):

> The documentation of the tool isn’t very clear about what specific checks it performs. In one of my tests I fed it an ISO image that had its last 50 MB missing (truncated). This did not result in any error or warning message! Most of the reported isovfy errors that I came across in my tests simply reflected the file system on the physical CD not conforming to ISO 9660 (this seems to be pretty common).

You can try this yourself by running *isovfy* on the following two ISO images:

* [This is a small (350 kB) ISO image](https://github.com/KBNLresearch/verifyISOSize/blob/master/testFiles/minimal.iso?raw=true) that is intact
* [Here's the same image with most of its data truncated](https://github.com/KBNLresearch/verifyISOSize/blob/master/testFiles/minimal_trunc.iso?raw=true) (the image is only 48 KB)

I ran both images through *isovfy* (version 3.02a06); both resulted in the following output:

```
Root at extent 17, 2048 bytes
[0,0]
No errors found
```

This demonstrates that *isovfy* is not very useful for detecting truncated ISO files.

## Digging into the specs

At this point I decided it was time to start digging into some specs. The [ISO 9660 page on the OSDev Wiki](http://wiki.osdev.org/ISO_9660) gives a good explanation of the internal organisation of an ISO 9660 image. From this I learnt that the [Primary Volume Descriptor](http://wiki.osdev.org/ISO_9660#The_Primary_Volume_Descriptor) (which is a data structure that is present on all ISO images) contains two interesting fields:

* *Volume Space Size*, which is the "number of Logical Blocks in which the volume is recorded";
* *Logical Block Size*, which is "the size in bytes of a logical block".

In theory, multiplying both figures should give the expected size of the ISO image, and this would provide a useful way to check if data are missing. To test this, I wrote a Python script that parses an ISO's Primary Volume Descriptor fields, calculates the expected file size and then compares this against the actual file size. Running the script against some 20 ISO images I had lying around showed that for 7 files the expected size was indeed identical to the actual file size. For most images, the actual size turned out to be marginally larger than expected (typically about 300-600 kB). For 3 images, the actual size was about twice the expected size. Digging deeper, I found out that these were hybrid images that contain an Apple partition on top of the ISO 9660 file system. According to [this Wikipedia article](https://en.wikipedia.org/wiki/Hybrid_disc#Multiple_file_systems), these hybrid discs come in two varieties:

1. Hybrid discs that contain an Apple Partition Map (located at 512 bytes into the disc/image).
2. Hybrid discs without a Partition Map. These contain a Master Directory Block (located at 1024 bytes into the disc/image).

In my case all of the 3 hybrid images turned out to be of the first category. Using the information [here](https://en.wikipedia.org/wiki/Apple_Partition_Map#Layout) and [here](https://opensource.apple.com/source/IOStorageFamily/IOStorageFamily-116/IOApplePartitionScheme.h) I was able to add detection of such hybrid images to my code, as well as a simple parser for the 'zero block' structure that contains two fields that define the partition's size: *Block Size* and *Block Count*. For my hybrid images, multiplying both figures resulted in a value that was close to (but again marginally smaller than) the actual file size.

Finally, I also added detection of the second hybrid disc category (no Partition Map, but Master Directory Block). The Master Directory Block also contains Block Size and Block Count fields that allow one to calculate the size of the file system.

## Isolyzer

I wrapped up the results of the above analyses into [*Isolyzer*](https://github.com/KBNLresearch/verifyISOSize), which is a dedicated (Python) tool for checking the size of an ISO image. What it does is this: 

1. Locate the image's Primary Volume Descriptor (PVD).
2. From the PVD, read the Volume Space Size (number of sectors/blocks) and Logical Block Size (number of bytes for each block) fields.
3. Calculate the expected file size as ( Volume Space Size x Logical Block Size ).
4. If the image contains an Apple Partition Map, read the Block Size and Block Count fields from the 'zero block'
5. Calculate the expected file size as ( Block Size x Block Count )
6. If the image contains an Apple Master Directory Block, read its Block Size and Block Count fields
7. Calculate the expected file size as ( Block Size x Block Count )
8. Calculate the final expected file size as the largest value out of any of the above 3 values
9. Compare this against the actual size of the image files.

In addition to this, Isolyzer also extracts and reports technical metadata from the Primary Volume Descriptor and the Zero Block. 

Currently the test results are reported in the following format (this may well change in upcoming releases):

```xml
<tests>
    <containsISO9660Signature>True</containsISO9660Signature>
    <containsApplePartitionMap>False</containsApplePartitionMap>
    <containsAppleHFSHeader>False</containsAppleHFSHeader>
    <containsAppleMasterDirectoryBlock>False</containsAppleMasterDirectoryBlock>
    <parsedPrimaryVolumeDescriptor>True</parsedPrimaryVolumeDescriptor>
    <sizeExpected>358400</sizeExpected>
    <sizeActual>358400</sizeActual>
    <sizeDifference>0</sizeDifference>
    <sizeAsExpected>True</sizeAsExpected>
    <smallerThanExpected>False</smallerThanExpected>
</tests>
```

In the above example the *sizeExpected* field is the size as calculated from the ISO/Apple headers, and *sizeActual* is the actual size. In this case both are identical. Below some output for a truncated ISO:

```xml
<tests>
    <containsISO9660Signature>True</containsISO9660Signature>
    <containsApplePartitionMap>False</containsApplePartitionMap>
    <containsAppleHFSHeader>False</containsAppleHFSHeader>
    <containsAppleMasterDirectoryBlock>False</containsAppleMasterDirectoryBlock>
    <parsedPrimaryVolumeDescriptor>True</parsedPrimaryVolumeDescriptor>
    <sizeExpected>358400</sizeExpected>
    <sizeActual>49157</sizeActual>
    <sizeDifference>-309243</sizeDifference>
    <sizeAsExpected>False</sizeAsExpected>
    <smallerThanExpected>True</smallerThanExpected>
</tests>
```

So, in this case *sizeDifference* is negative, and flag *smallerThanExpected* equals 'True' (which indicates a damaged image).

## Feedback wanted

At this stage Isolyzer is a bit experimental and pretty rough around the edges, and I wouldn't recommend it for production use. Nevertheless I'm curious about any feedback on the tool. Do others find this useful? Are things missing (i.e. other hybrid disc types I'm not aware of), or did I get anything completely wrong?

One thing that puzzles me a bit is that for the majority of ISO images I've come across, the expected size as calculated by Isolyzer is marginally smaller than the actual size. The difference is typically in the order of about 300-600 kB. I'm not quite sure what's causing this, although [this article](http://twiki.org/cgi-bin/view/Wikilearn/CdromMd5sumsAfterBurning) mentions that some CD writing software packages add padding bytes when writing a CD. I wasn't able to verify if this, although [this SuperUser answer](http://superuser.com/a/220353) on validating a burnt DVD suggests it as well. If anyone knows more about this, please let me know!

## Download links

Isolyzer can be found [here on Github](https://github.com/KBNLresearch/verifyISOSize). It can be installed using *pip*; [see the instructions here](https://github.com/KBNLresearch/verifyISOSize#installation-with-pip). For Windows users who cannot/don't want to install Python I also provided stand-alone Windows binaries, which are available for download [here](https://github.com/KBNLresearch/verifyISOSize/releases).


<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2017/01/13/detecting-broken-iso-images-introducing-isolyzer/)
