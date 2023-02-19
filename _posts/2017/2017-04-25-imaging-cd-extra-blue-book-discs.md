---
layout: post
title: Imaging CD-Extra / Blue Book discs
tags: [isolyzer,optical-media,ISO-9660,HFS,UDF]
comment_id: 24
---

The development work on an imaging/ripping workflow for optical media is shaping up steadily, and you can expect a write-up with more information about our software and hardware setup here in the near future (you can get a sneak peek [here](https://github.com/KBNLresearch/iromlab)). However, this blog is about a very specific problem that we ran into while testing the workflow with a selection of discs from our collection. This selection included a few discs that follow the [*Blue Book*](https://en.wikipedia.org/wiki/Blue_Book_(CD_standard)) standard (also known as *CD-Extra*). This standard defines a method for combining audio and data tracks on one disc. A *CD-Extra* disc contains two sessions, where the first session holds all audio tracks, and the second session holds a data track. *Blue Book* was (and still is) widely used for audio CD's with bonus videos or software.

<!-- more -->

In our workflow, these discs are handled in the following way:

1. Identify the audio and data sessions with the [*cd-info*](https://linux.die.net/man/1/cd-info) tool (part of the [*libcdio*](https://www.gnu.org/software/libcdio/) library).
2. Rip the audio tracks in the first session to *WAVE* or *FLAC* files using [*dBpoweramp*](https://www.dbpoweramp.com/).
3. Verify the audio files for completeness with [*shntool*](http://www.etree.org/shnutils/shntool/) (*WAVE*) or [*flac*](https://xiph.org/flac/) (*FLAC*).
4. Extract the data track in the second session to an ISO image with [*IsoBuster*](https://www.isobuster.com/).
5. Verify the ISO image for completeness with [*isolyzer*](https://github.com/KBNLresearch/isolyzer).

## Size of ISO image smaller than expected

All *Blue Book* discs in our test passed steps 1 through 4 without any issues, but failed the final *isolyzer* check. This *isolyzer* check involves a comparison of the file size of the ISO image against the *expected* size, as calculated from the image's Primary Volume Descriptor fields (and Apple HFS blocks, if present). If the actual size is smaller than the expected size, this indicates the image is incomplete. For *all* ISO images that were extracted from a *Blue Book* disc, the actual file size was significantly smaller than  expected. Moreover, the images couldn't be mounted in Linux, or opened in file archiver software (e.g. 7-Zip). Below is an excerpt from the *isolyzer* output of one of the offending images:

```xml
<tests>
    <containsISO9660Signature>True</containsISO9660Signature>
    <containsApplePartitionMap>True</containsApplePartitionMap>
    <containsAppleHFSHeader>False</containsAppleHFSHeader>
    <containsAppleMasterDirectoryBlock>False</containsAppleMasterDirectoryBlock>
    <parsedAppleZeroBlock>True</parsedAppleZeroBlock>
    <parsedPrimaryVolumeDescriptor>True</parsedPrimaryVolumeDescriptor>
    <sizeExpected>609912832</sizeExpected>
    <sizeActual>554373120</sizeActual>
    <sizeDifference>-55539712</sizeDifference>
    <sizeAsExpected>False</sizeAsExpected>
    <smallerThanExpected>True</smallerThanExpected>
</tests>
```
So, in this case the ISO image is about 56 MB smaller than expected, even though *IsoBuster* did not report any errors during the extraction process. One thing that caught my eye after running a few of these problematic images through *isolyzer*, was that the value of *sizeDifference* always roughly corresponded to the size of the uncompressed audio on the disc. The *sizeExpected* value is calculated from the *Volume Space Size* field (which defines the number of logical blocks in the ISO 9660 file system) in the ISO's Primary Volume Descriptor. This made me wonder: does the *Volume Space Size* value really reflect the number of sectors occupied by the *data track* (which I was quietly assuming), or does it perhaps reflect *all sectors on the disc* (including those in the audio session)?

Finding the answer to this question was more difficult than I expected, but my initial suspicions were confirmed by this entry on the Debian mailing list:

[Re: reading the raw iso from a CD-Extra (multisession CD)](https://lists.debian.org/debian-user/2005/01/msg02339.html)

Which states:

> [T]he sector numbers in the file system refer[s] to sectors of the original CD rather than sectors of session2.iso.

Similarly, from this thread on the *libcdio* mailing list:  

[[Libcdio-devel] Retrieving DATA session from multisession audio disc](https://lists.gnu.org/archive/html/libcdio-devel/2010-02/msg00048.html)

> Remember, the path table and directory structure of the iso reflect the fact that the ISO filesystem starts on sector 222145 (49:23:70) of the CD.  If it is burned to another CD at a different position, it won't work.  Likewise, any program that reads the iso will need to be able to compensate for the offset.

This confirms that all references to blocks of data (sectors) in these ISO images are defined *relative to the start of the physical CD*, and **not** relative to the start of the ISO image! So the questions are:

1. How can we verify that these images are complete?
2. How can we access these images at all?

Using the information from (mostly) the aforementioned *libdio* mailing list thread I was able to answer both questions. The steps below will work on most Linux-based systems (and possibly some Windows-based ones as well).

## Cd sector layout

As a first step we need some information on the sector layout of the physical disc, and in particular the start sector of the second session (which contains the data track). You can do this by running *cd-info* while the disc is in the drive: 

```bash
cd-info /dev/sr0 > leesleeuw_cdinfo.txt
```

In the resulting output, the start sesssion of the data session is listed twice. First look at the *CD-ROM Track List*  section:

```
CD-ROM Track List (1 - 3)
    #: MSF       LSN    Type   Green? Copy? Channels Premphasis?
    1: 00:02:00  000000 audio  false  no    2        no
    2: 01:25:24  006249 audio  false  no    2        no
    3: 06:05:46  027271 data   false  no   
170: 66:14:61  297961 leadout (668 MB raw, 668 MB formatted)
Media Catalog Number (MCN): 0000000000000
TRACK  1 ISRC: 000000000000
TRACK  2 ISRC: 000000000000
TRACK  3 ISRC: 000000000000
Last CD Session LSN: 27271
```

This tells us that the CD contains three tracks, where track 1 and track 2 are audio tracks, and track 3 is a data track. The second column (heading: LSN) shows the start sector of each track; for track 3 this is 027271. This value is repeated in the *Last CD Session LSN* (sector number of last session). Finally it can be found again in the *CD Analysis Report* at the bottom of the file:

```
session #2 starts at track  3, LSN: 27271, ISO 9660 blocks: 297809
```

So, in this case session 2 (which contains the data track) starts on sector 27271 of the disc.

## Verify ISO image for completeness

Now that we know the start sector, we can use this to verify the ISO image. For this I added the new `--offset` option to *isolyzer*. This lets you specify a start sector offset, which is subtracted from the expected size estimate that is calculated from the Primary Volume Descriptor[^1]. So we call *isolyzer*  like this:

```bash
isolyzer leesleeuw.iso --offset 27271
```

The *tests* elements now looks like this: 

```xml
<tests>
    <containsISO9660Signature>True</containsISO9660Signature>
    <containsApplePartitionMap>True</containsApplePartitionMap>
    <containsAppleHFSHeader>False</containsAppleHFSHeader>
    <containsAppleMasterDirectoryBlock>False</containsAppleMasterDirectoryBlock>
    <parsedAppleZeroBlock>True</parsedAppleZeroBlock>
    <parsedPrimaryVolumeDescriptor>True</parsedPrimaryVolumeDescriptor>
    <sizeExpected>554061824</sizeExpected>
    <sizeActual>554373120</sizeActual>
    <sizeDifference>311296</sizeDifference>
    <sizeAsExpected>False</sizeAsExpected>
    <smallerThanExpected>False</smallerThanExpected>
</tests>
```

The *isolyzer* output now shows that the size of the ISO image is about 311 KB (152 sectors) larger than the expected value; this is completely fine.

## Access the file system

Now that we're (reasonably) sure the ISO image is complete, the next step is to access its contents. If the image has a [hybrid file system](https://en.wikipedia.org/wiki/Hybrid_disc#Multiple_file_systems), it may be possible to mount it directly on some platforms. For instance, opening an image that contains an Apple partition with Linux Mint's Disk Image Mounter will mount the Apple partition (but not the ISO 9660 file system, which may not necessarily point to the same files!). Fortunately, [this message on the *libcdio* mailing list](https://lists.gnu.org/archive/html/libcdio-devel/2010-02/msg00050.html) by Thomas Schmitt (one of the authors of the [*libburnia*](https://en.wikipedia.org/wiki/Libburnia) library) explains how to mount these images. The trick here is to insert a block of data at the start of our ISO image that corresponds to the size of the sectors that are missing from the physical disc (i.e. the sectors that are part of the first session). The effect of this is that  all sector references will again match the actual sector locations in the image.

First we create a file that contains 27271 sectors that are filled with zero-bytes (note that the *seek* position equals *Session Start Sector - 1*): 

```bash
dd if=/dev/zero bs=2K count=1 seek=27270 of=leesleeuw_tmp.iso
```

Next we append our ISO image to this file:

```bash
cat leesleeuw.iso >>leesleeuw_tmp.iso
```

Finally we mount the image:

```bash
sudo mount -t iso9660 -o loop,sbsector=27270 leesleeuw_tmp.iso /media/johan
```

We can now navigate the file system with the file manager.

## Create access ISO

In order to use the ISO with an emulator or virtual machine, we have to do some additional work. In theory, modifying all sector addresses in the ISO would do the trick, but as far as I'm aware there is no software tool that is capable of this. Instead, we'll use the mounted file system from the previous step to create a completely new ISO image. We can do this with the following command:

```bash
xorrisofs -r -J -o leesleeuw_new.iso /media/johan
```

Again, credits go to Thomas Schmitt who suggested this on the [*libcdio* mailing list](https://lists.gnu.org/archive/html/libcdio-devel/2010-02/msg00053.html). 

Finally we check the image with *isolyzer*:

```xml
<tests>
    <containsISO9660Signature>True</containsISO9660Signature>
    <containsApplePartitionMap>False</containsApplePartitionMap>
    <containsAppleHFSHeader>False</containsAppleHFSHeader>
    <containsAppleMasterDirectoryBlock>False</containsAppleMasterDirectoryBlock>
    <parsedPrimaryVolumeDescriptor>True</parsedPrimaryVolumeDescriptor>
    <sizeExpected>508913664</sizeExpected>
    <sizeActual>508913664</sizeActual>
    <sizeDifference>0</sizeDifference>
    <sizeAsExpected>True</sizeAsExpected>
    <smallerThanExpected>False</smallerThanExpected>
</tests>
```

I tried this approach for a couple of ISO images with old Windows installables. I mounted the images to a Windows 2000 machine running in VirtualBox, and in all cases I was able to access the images and install the software without problems. 

## Recommendations and caveats

Based on some limited tests, the above approach looks useful for providing access to extracted data sessions from CD-Extra discs that would otherwise be inaccessible. There are some caveats though:

First, the derived ISO images should *only* be treated as access images, not as preservation masters! There are several reasons for this. An important one is that if the original image contained an Apple partition, the reformatting procedure will get rid of it! Since Apple partitions may point to different data than the ISO 9660 file system, this may result in loss of data (files).

Second, sometimes the data session may reference the audio session. A simple example would be an audio player application that lets a user play audio tracks. This functionality will be lost in a emulated environment. I'm not aware of any solutions for this, especially given the lack of (open) disc image formats that are able to capture all data on a multisession CD. However, it is not impossible that such a format will appear at some point in the future. If that happens, it may be possible to reformat the ripped audio and data  tracks into that format, but this would only work if a full record of the sector layout of the physical disc is available. 

Since we need some of this information already for accessing the ISO images, the most important recommendation that follows from this work is to always keep a record of the sector layout of CD-Extra / Blue Book discs. This can be done by simple running [*cd-info*](https://linux.die.net/man/1/cd-info) on each disc, and storing its output as metadata with the ripped/imaged files. 

Finally, I was surpprised at the complete lack of any software tool that is capable of manipulating sector offsets in an ISO image. Having such a tool would enable us to create readable access ISOs in a more straightforward way (i.e. without first having to add padding bytes to the source source image and thren having to mount the resulting image). It might also be less error-prone. I wonder if there's any interest from the wider community to invest in the development of such a tool?

## Links

* [Isolyzer](https://github.com/KBNLresearch/isolyzer)
* [Sample multisession ISO image](https://github.com/KBNLresearch/isolyzer/raw/master/testFiles/multisession.iso) (6 MB download; start sector of the data session is 21917)

[^1]: This is analogous to the `-N` option in *cdinfo*.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2017/04/25/imaging-cd-extra-blue-book-discs/)
