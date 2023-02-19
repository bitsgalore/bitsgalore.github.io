---
layout: post
title: Preserving optical media from the command-line
tags: [optical-media,ISO-9660]
comment_id: 31
---

The KB has quite a large collection of offline optical media, such as CD-ROMs, DVDs and audio CDs. We're currently investigating how to stabilise the contents of these materials using disk imaging. During the initial phase of this work I did a number of tests with various open-source tools. It's doubtful whether we'll end up using these same tools in our actual workflows. The main reason for this is the sheer size of the collection, which we estimated at some 15,000 physical carriers; possibly even more. At those volumes we will need a solution that involves the use of a disk robot, and these often require dedicated software (we still need to investigate this more in-depth).

Nevertheless, throughout the initial testing phase I was surprised at the number of useful tools that are available in the open source domain. Since this will probably be of interest to others as well, I decided to polish a selection from my [rough working notes](https://gist.github.com/bitsgalore/1bea8f015eca21a706e7#file-notescdimaging-md) into a somewhat more digestible form (or so I hope!). I edited my original notes down to the following topics:

* How to figure out the device path of the CD drive
* How to create an ISO image from a CD-ROM or DVD
* How to check the integrity of the created ISO image
* How to extract audio from an audio CD

In addition there's a final section that covers my attempts at imaging a multisession / mixed mode CD. The result of this particular exercise wasn't all that successful, but I included it anyway, as some may find it useful. All software mentioned here are open-source tools that are available for any modern Linux distribution (I'm using Linux Mint myself). Some can be used under Windows as well using [Cygwin](https://www.cygwin.com/).

<!-- more -->

## Find the device path of the CD drive (Linux)

The majority of the tools covered by this blog post need the device path of the CD drive as a command-line argument. Under Linux you can usually find this by inspecting the output of the following command (run this while a CD or DVD is inserted in your drive):

```bash
mount|grep ^'/dev'
```

If all goes well, the result will look similar to this:

```data
/dev/sda1 on / type ext4 (rw,errors=remount-ro)
/dev/sr0 on /media/johan/REBELS_0 type iso9660
(ro,nosuid,nodev,uid=1000,gid=1000,iocharset=utf8,mode=0400,dmode=0500,uhelper=udisks2)
```

So, in this case the path to the CD drive is */dev/sr0* (if you have multiple optical drives you may also see */dev/sr1*, and so on).

## Finding the device path on Windows (Cygwin)

For some reason the *mount* command doesn't result in the printing of any device paths in CygWin. Instead, try this:

```bash
ls /dev/
```

Which produces a list of all devices:

```data
clipboard  dsp   mqueue  random  sda2  sdc1    stdin   ttyS2
conin      fd    null    scd0    sdb   shm     stdout  urandom
conout     full  ptmx    sda     sdb1  sr0     tty     windows
console    kmsg  pty0    sda1    sdc   stderr  ttyS0   zero
```

In the above output both *sr0* and *scd0* point to the CD drive, and either the full paths */dev/sr0* or */dev/scd0* will work (again in case of multiple drives you may be looking for */dev/sr1* or */dev/scd1*).

In all examples below I assumed that the device path is  */dev/sr0*; substitute your own path if necessary. 

## Create ISO image of a CD-ROM or DVD

A number of tools allow you to create an (ISO[^2]) image from a CD-ROM or DVD. Although generic Unix data copying and recovery tools like [dd](http://linux.die.net/man/1/dd) and [ddrescue](http://linux.die.net/man/1/ddrescue) are often used for this, various people have pointed out that the result may be unreliable because they only perform limited error checking. See for example the comments [here](http://www.commandlinefu.com/commands/view/10957/rip-a-cddvd-to-iso-format.0) and [here](https://pthree.org/2011/09/26/how-to-properly-create-and-burn-cddvd-iso-images-from-the-command-line/); both recommend to use the [*readom*](http://linux.die.net/man/1/readom) tool, which is part of the [*cdrkit*](https://en.wikipedia.org/wiki/Cdrkit) library. My own experience with *readom* is that while it works great in most cases, it is less suitable for CD-ROMs that are damaged or otherwise degraded. In those cases *ddrescue* is often a better choice. So below I'll first show how to use *readom*, followed by a *ddrescue* example that specifically addresses the recovery of a CD-ROM gives read errors in *readom*.

### Running readom

The [documentation](http://linux.die.net/man/1/readom) recommends to always run *readom* as root. Also, before running *readom*, the CD or DVD must be unmounted[^4]. So, after inserting the CD or DVD, first enter this:

```bash
umount /dev/sr0
```

Then run *readom* as root:

```bash
sudo readom retries=4 dev=/dev/sr0 f=mydisk.iso
```

Here the value of the *retries* parameter defines the number of attempts that *readom* will make at trying to recover unreadable sectors. The default value is 128, which can result in huge processing times for CDs that are seriously damaged. The *f* parameter sets the name of the image file that is created. If all goes well the following output is printed to the screen at the end of the imaging process:

```data
Read  speed:  4234 kB/s (CD  24x, DVD  3x).
Write speed:     0 kB/s (CD   0x, DVD  0x).
Capacity: 309104 Blocks = 618208 kBytes = 603 MBytes = 633 prMB
Sectorsize: 2048 Bytes
Copy from SCSI (10,0,0) disk to file 'mydisk.iso'
end:    309104
addr:   309104 cnt: 44
Time total: 259.287sec
Read 618208.00 kB at 2384.3 kB/sec.
```

### When read errors happen: try ddrescue

If the source medium is in a bad condition or otherwise damaged, *readom* will most likely terminate prematurely with read errors. If this happens, you may get better results with [*ddrescue*](http://linux.die.net/man/1/ddrescue). There are two reasons for this:

1. Unlike *readom*, which usually gives up pretty soon after the first read error occurs, *ddrescue* was specifically designed to deal with source media that contain errors. Consequently, it is much more persistive in such cases.
2. If you try to read a defective source medium using two different CD drives (let's call them *A* and *B*), it is not uncommon to find that some sectors that result in read errors on drive *A* are read correctly by drive *B* (and vice versa). With *ddrescue* it is possible to take advantage of this.

The [*ddrescue* Manual](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html#Optical-media) manual gives a (very concise) example of how this works. Based on this I created the following, more detailed example. 

First we run *ddrescue* with the following command line:

```bash
ddrescue -b 2048 -r4 -v /dev/sr0 mydisk.iso mydisk.log
```

Here `-b` sets the block size (which is typically 2048 bytes for a CD-ROM); `-r4` sets the maximum number of retries in case of bad sectors to 4[^7], and `-v` activates verbose output mode. File *mydisk.log* is a so-called [*mapfile*](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html#Mapfile-structure) (known as *logfile* in *ddrescue* versions prior to 1.20). The *mapfile* contains (among a few other things) information on the recovery status of blocks of data. After running the above command on a faulty CD-ROM, we end up with output that looks like this:

```data
GNU ddrescue 1.17
About to copy 624918 kBytes from /dev/sr0 to mydisk.iso
    Starting positions: infile = 0 B,  outfile = 0 B
    Copy block size:  32 sectors       Initial skip size: 32 sectors
Sector size: 2048 Bytes

Press Ctrl-C to interrupt
rescued:   624871 kB,  errsize:   47104 B,  current rate:        0 B/s
    ipos:   508162 kB,   errors:       3,    average rate:     592 kB/s
    opos:   508162 kB,    time since last successful read:    12.3 m
Finished
```

From this we can see the following: 

* The CD-ROM contains 624918 kBytes of data (2nd line from top).
* Only 624871 kBytes were extracted ('rescued' field)
* A total of 47104 bytes were *not* rescued ('errorsize' field)
* 3 errors occurred while reading the CD ('errors' field)  

However, it is often possible to improve the result by additional runs of *ddrescue* using either different options, or other hardware. First we'll see if we can improve things by re-running in [*direct disc access*](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html#Direct-disc-access) mode (this does not work on some systems, in which case *ddrescue* will report a warning). So we use the following command[^6]:

```bash
ddrescue -d -b 2048 -r1 -v /dev/sr0 mydisk.iso mydisk.log
```

Here the `-d` switch activates direct disc access, which bypasses the kernel cache (note that the number of retries is set to 1 in the above example). Running the command causes *ddrescue* to update both the ISO and the mapfile. The screen output now looks like this:

```data
GNU ddrescue 1.17
About to copy 624918 kBytes from /dev/sr0 to mydisk.iso
    Starting positions: infile = 0 B,  outfile = 0 B
    Copy block size:  32 sectors       Initial skip size: 32 sectors
Sector size: 2048 Bytes

Press Ctrl-C to interrupt
Initial status (read from logfile)
rescued:   624871 kB,  errsize:   47104 B,  errors:       3
Current status
rescued:   624912 kB,  errsize:    6144 B,  current rate:        0 B/s
    ipos:   508162 kB,   errors:       3,    average rate:     1706 B/s
    opos:   508162 kB,    time since last successful read:       7 s
Finished
```

What we see here:

* 624871 kBytes were extracted (previously this was 624871)
* Consequently 'errsize' has gone down from to 47104 to 6144 bytes

So this is better, but still not perfect. So let's try if we can improve the results by using a different CD-reader. At this point I hooked up an external USB CD-drive, and moved my faulty CD-ROM from the internal reader to the external one. In this case my external drive is mapped under device path */dev/sr2* (re-run the aforementioned steps to find the device path if necessary). This gives the following command-line:

```bash
ddrescue -d -b 2048 -r4 -v /dev/sr2 mydisk.iso mydisk.log
```

Now the output looks like this:

```data
GNU ddrescue 1.17
About to copy 624918 kBytes from /dev/sr2 to mydisk.iso
    Starting positions: infile = 0 B,  outfile = 0 B
    Copy block size:  32 sectors       Initial skip size: 32 sectors
Sector size: 2048 Bytes

Press Ctrl-C to interrupt
Initial status (read from logfile)
rescued:   624916 kB,  errsize:    2048 B,  errors:       1
Current status
rescued:   624918 kB,  errsize:       0 B,  current rate:      682 B/s
    ipos:   106450 kB,   errors:       0,    average rate:      682 B/s
    opos:   106450 kB,    time since last successful read:       0 s
Finished
```

From the output we can see that after re-running *ddrescue* with the external drive, both 'errsize' and the number of errors have gone down to 0. In other words: all of the contents of the CD have been rescued without any errors. Yay!

In the above example I used two different CD readers that were connected to the same machine, but you could use as many readers as you like. It also possible to do the first run on one machine, transfer the ISO image and the mapfile to another machine, and then re-run *ddrescue* there (this even works across OS platforms).

## Check integrity of ISO image against physical CD-ROM or DVD

In theory you could use check the integrity of the created ISO image by computing a checksum on both the ISO file and the physical carrier, and then comparing both:

```bash
md5sum mydisk.iso
md5sum /dev/sr0
```

However, in practice this comparison is not all that useful. Using dedicated data recovery tools like *readom* and particularly *ddrescue* often results in a more accurate capture of the data on a disc than accessing the corresponding device directly. Because of this, computing a checksum on the device using something like `md5sum /dev/sr0` can give unreliable results, resulting in a checksum mismatch that does not indicate any fault of the ISO image. It is worth noting that the aforementioned [Aaron Toponce article](https://pthree.org/2011/09/26/how-to-properly-create-and-burn-cddvd-iso-images-from-the-command-line/) claims that *readom* already does a checksum check. If true, the additional check would be overkill (especially given that computing a checksum on a physical CD or DVD is time consuming). However, I couldn't find any confirmation of this in either *readom*'s documentation nor its source code (although I found the source hard to read, so I may have simply overlooked it).

## Verify ISO image

In theory, there shouldn't be any need for additional quality checks on an ISO image once its integrity against the physical carrier is confirmed by the checksum. However, since *cdrkit* includes an [*isovfy*](http://linux.die.net/man/8/isoinfo) tool that claims to " verify the integrity of an iso9660 image", I decided I might as well give it a try. It works by entering:

```bash
isovfy mydisk.iso
```

Here's some example output:

```data
Root at extent 13, 2048 bytes
[0 0]
No errors found
```

The documentation of the tool isn't very clear about *what* specific checks it performs. In one of my tests I fed it an ISO image that had its last 50 MB missing (truncated). This did not result in any error or warning message! Most of the reported *isovfy* errors that I came across in my tests simply reflected the file system on the physical CD not conforming to ISO 9660 (this seems to be pretty common). Based on this it looks like *isovfy* isn't  very useful after all.

### Isolyzer

In response to the problems I encountered with *isovfy*, I created the [*isolyzer*](https://github.com/KBNLresearch/isolyzer) tool. *Isolyzer* checks the file size of an ISO image against the size information in the file system headers. This can be used to identify damaged and incomplete ISO images. Currently supported file systems are ISO 9660, UDF, HFS, HFS+ and a number of hybrids of these file systems. More information on *Isolyzer* can be found [here]({{ BASE_PATH }}/2017/01/13/detecting-broken-iso-images-introducing-isolyzer) and [here]({{ BASE_PATH }}/2017/07/12/update-on-isolyzer-udf-hfs-and-more).

## Get information about an ISO image

### Isoinfo

The [*Primary Volume Descriptor*](http://wiki.osdev.org/ISO_9660#The_Primary_Volume_Descriptor) (PVD) of an ISO 9660 file system contains general information about the CD or DVD. The [*isoinfo*](http://linux.die.net/man/8/isoinfo) tool (which is also part of *cdrkit*) is able to  print the most important PVD fields to the screen:

```bash
isoinfo -d -i mydisk.iso
```

Result:

```data
CD-ROM is in ISO 9660 format
System id: 
Volume id: REBELS_0
Volume set id: 
Publisher id: 
Data preparer id: 
Application id: NERO - BURNING ROM
Copyright File id: 
Abstract File id: 
Bibliographic File id: 
Volume set size is: 1
Volume set sequence number is: 1
Logical block size is: 2048
Volume size is: 333151
Joliet with UCS level 3 found
NO Rock Ridge present
```

You can also run *isoinfo* directly on the physical carrier:

```bash
isoinfo -d -i /dev/sr0
```

To get a listing of all files and directories that are part of the filesystem, use this:

```bash
isoinfo -f -i mydisk.iso
```

Result:

```data
/AUTORUN.EXE;1
/AUTORUN.INF;1
/DISK0
/LICENSE2.TXT;1
/LICENSEF.TXT;1
/LICENSEU.TXT;1
/SETUP.EXE;1
/DISK0/CONTROLS.CFG;1
/DISK0/DISK0;1
::
::
etc
```

It looks like all items that are followed by *;1* are files, and those that aren't are directories. Also, the `-l` option can be used for a detailed list that includes additional file attributes (size, date, etc.).

### Disktype

The [*disktype*](http://disktype.sourceforge.net/) tool is particularly useful for identifying [hybrid disc](https://en.wikipedia.org/wiki/Hybrid_disc) images that combine multiple file systems. For example:   

```bash
disktype bewaarmachine.iso
```

This results in:

```data
--- bewaarmachine.iso
Regular file, size 342.1 MiB (358727680 bytes)
Apple partition map, 2 entries
Partition 1: 1 KiB (1024 bytes, 2 sectors from 1)
    Type "Apple_partition_map"
Partition 2: 172.4 MiB (180773376 bytes, 353073 sectors from 346957)
    Type "Apple_HFS"
    HFS file system
    Volume name "de bewaarmachine"
    Volume size 172.4 MiB (180764672 bytes, 44132 blocks of 4 KiB)
ISO9660 file system
    Volume name "BEWAARMACHINE_PC"
    Application "TOAST ISO 9660 BUILDER COPYRIGHT (C) 1993-1996 MILES SOFTWARE GMBH - HAVE A NICE DAY"
    Data size 169.4 MiB (177641472 bytes, 86739 blocks of 2 KiB)
```

In this example we have an image that contains both an ISO 9660 and an Apple HFS filesystem. *Disktype* can also be run directly on the physical carrier, using:

```bash
disktype /dev/sr0
```

### Isolyzer

The  [*isolyzer*](https://github.com/KBNLresearch/isolyzer) tool also gives detailed information about an ISO image, including the file systems it contains. As an example:

```bash
isolyzer bewaarmachine.iso > bewaarmachine.xml
```

This results [in this output file](https://gist.github.com/bitsgalore/bf5f9fb8e936efb9c4bc06a04443ef4a).

## Rip audio CD with cdparanoia

The data structure of an audio CD is fundamentally different from a CD-ROM or DVD, and because of this its content cannot be stored as an ISO image. The most widely-used approach is to extract (or "rip") the audio tracks on a CD to separate [*WAVE*](http://fileformats.archiveteam.org/wiki/WAV) files. A complicating factor here is that the way audio is encoded on a CD tends to obscure (small) read errors during playback. As a result, a single linear read will not result in a reliable transfer of the audio data. More details can be found in [this excellent article by Alexander Duryee](http://journal.code4lib.org/articles/9581). Duryee recommends a number of extraction tools that overcome this problem using sophisticated verification and correction functionality. One of these tools is the [*cdparanoia*](http://linux.die.net/man/1/cdparanoia) ripper. As an example, the following command can be used to rip a CD in batch mode, where each track is stored as a separate *WAVE* file:

```bash
cdparanoia -B -L
```

or:

```bash
cdparanoia -B -l
```

The `-L` switch results in the generation of a detailed log file; `-l` produces a summary log (name:  *cdparanoia.log*)[^5]. File names are generated automatically like this:

```data
track01.cdda.wav
track02.cdda.wav
track03.cdda.wav
```

[Here is a link to an example log file](https://gist.github.com/bitsgalore/1bea8f015eca21a706e7#file-cdparanoialogsummary-log). The output may look a little weird at first sight, which is because *cdparanoia* reports all status and progress information as symbols and smilies, respectively. Their meaning is explained in the [documentation](http://linux.die.net/man/1/cdparanoia).

## Extract data/audio from mixed mode and 'enhanced' CDs

Some CDs combine data and audio tracks. There are essentially two ways to do this:

- [Mixed Mode](https://en.wikipedia.org/wiki/Mixed_Mode_CD) CDs contain both audio and data, both of which are written into one single session. Mixed Mode was often used for '90s video games.
- The [Blue Book](https://en.wikipedia.org/wiki/Blue_Book_(CD_standard)) standard defines a way to combine audio and data tracks. Blue Book CDs contain two sessions, where the first one contains one or more audio tracks, and the second one a data track. Examples of such discs are "enhanced" audio CDs that include software or movies as bonus material. They are sometimes referred to a "CD-Extra" discs.

Even though the *data* part of such CDs is typically compatible with an ISO 9660 (or HFS, HFS+) file system, the audio tracks are not. Since [there is no good, open and mature file format to describe the contents of a CD precisely](http://anjackson.net/keeping-codes/practice/developing-a-robust-migration-workflow-for-preserving-and-curating-handheld-media.html), such CDs pose a particular challenge. In addition, tools such as *readom* and *ddrescue* typically only recognise the first session on a multisession CD, which means that they are not suitable for handling this type of disc.

Based on some (relatively limited) testing, the [*cdrdao*](http://linux.die.net/man/1/cdrdao) tool does a good job at imaging mixed mode CDs, and a reasonable (but less than ideal) job for Blue book discs. Below are some brief notes on how to recognise both types of disc, and how to image them.

### Identifying mixed-mode CDs

We can identify mixed-mode discs by running the *cd-info* command, which is part of the [*GNU libcdio*](https://www.gnu.org/software/libcdio/libcdio.html) package:

```bash
cd-info /dev/sr0
```

Now pay attention to the "CD Analysis Report" at the bottom of *cd-info*'s output:

```data
CD Analysis Report

CD-TEXT for Disc:
CD-TEXT for Track  1:
CD-TEXT for Track  2:
mixed mode CD   
CD-ROM with ISO 9660 filesystem
ISO 9660: 235494 blocks, label `TNT_ROM                         '
Application: TOAST ISO 9660 BUILDER COPYRIGHT (C) 1993 MILES SOFTWARE ENGINEERING - HAVE A NICE DAY
Preparer   : 
Publisher  : 
System     : APPLE COMPUTER, INC., TYPE: 0002
Volume     : TNT_ROM
Volume Set : 
mixed mode CD   XA sectors   
session #2 starts at track  2, LSN: 235719, ISO 9660 blocks: 235494
ISO 9660: 235494 blocks, label `TNT_ROM    
```

Both the 4th line from the top and the 3rd line from the bottom contain the text `mixed mode CD`.

### Imaging mixed-mode CDs

[This article on the Linux Reviews site](https://web.archive.org/web/20151202174511/http://linuxreviews.org/howtos/cdrecording/) (archived link) contains instructions on how to rip a mixed-mode CD using *cdrdao*. It involves a number of steps.

First we have to unmount the disc:

```bash
umount /dev/sr0
```

Then run *cdrdao* with the following arguments:

```bash
cdrdao read-cd --read-raw --datafile toolstales.bin --device /dev/sr0 --driver generic-mmc-raw toolstales.toc
```

The result of this is a disc image in *BIN/TOC* format. The *.toc* file looks like this:

```data
CD_ROM_XA

CATALOG "0000000000000"

// Track 1
TRACK MODE2_RAW
NO COPY
DATAFILE "toolstales.bin" 52:20:69 // length in bytes: 554058288


// Track 2
TRACK AUDIO
NO COPY
NO PRE_EMPHASIS
TWO_CHANNEL_AUDIO
SILENCE 00:02:00
FILE "toolstales.bin" #554058288 0 14:58:22
START 00:02:00
```

The *BIN/TOC* format is not easily accessible, so we need to do some additional post-processing to convert the image into a more accessible format.

First we convert the *.toc* file to the *.cue* format (as defined in Appendix A of the [*CDRWIN* User Guide](https://web.archive.org/web/20070614044112/http://www.goldenhawk.com/download/cdrwin.pdf)). For this we use the *toc2cue* tool (which is part of *cdrdao*):

```bash
toc2cue toolstales.toc toolstales.cue
```

We now have a *BIN/CUE* image. On Linux we can mount this image with a virtual drive controller such as [*cdemu*](https://cdemu.sourceforge.io/). This way both the audio and the data are accessible in the same way they would be from the physical carrier.

If needed it is possible to extract the data track of the *BIN/CUE* file to an ISO image, and any audio tracks to *WAVE* files. For this we need the [*bchunk*](http://linux.die.net/man/1/bchunk) tool. Now we invoke it with the following arguments:

```bash
bchunk -s -w toolstales.bin toolstales.cue toolstales
```

In the example above, the `-w` option tells *bchunk* to extract audio tracks to *WAVE* files, and the `-s` option does a byte swap on the audio samples[^8]. The last argument defines the base name for all created output files. In this case the command  results in 2 files: *toolstales01.iso*, which is a mountable ISO image, and *toolstales02.wav*, which is the audio track.

### Identifying enhanced CDs

To identify enhanced (Blue Book) CDs, we again use *cd-info*:

```bash
cd-info /dev/sr0
```

Here's the corresponding "CD Analysis Report" at the bottom of *cd-info*'s output:

```data
CD Analysis Report

CD-TEXT for Disc:
CD-TEXT for Track  1:
::  ::
CD-TEXT for Track 18:
CD-Plus/Extra   
session #2 starts at track 18, LSN: 163570, ISO 9660 blocks: 170006
ISO 9660: 170006 blocks, label `NO   
```

Note `CD-Plus/Extra`, which indicates this is a multi-session CD.

### Imaging enhanced CDs

The procedure for imaging enhanced CDs is largely identical to the one for mixed-mode CDs. However, a major limitation here is that *cdrdao* is not able to combine the data/audio from both sessions into one disc image: running *cdrdao* with the command-line arguments as shown in the previous section will only create an image of the first session! However, it is possible to image both sessions separately into two image files. As an example, below are the steps I followed in an attempt to make a copy of They Might Be Giants' [*"No"*](http://tmbw.net/wiki/No!) album (which contains some video content). Again I first unmounted the disk:

```bash
umount /dev/sr0
```

Then I used the below command to create an image of the first session (note the `--session` option):

```bash
cdrdao read-cd --read-raw --session 1 --datafile no1.bin --device /dev/sr0 --driver generic-mmc-raw no1.toc
```

And then again for the second session:

```bash
cdrdao read-cd --read-raw --session 2 --datafile no2.bin --device /dev/sr0 --driver generic-mmc-raw no2.toc
```

Running *cdrdao* twice like this, I was able to create two separate images with the audio and file system data, respectively.

As in the mixed-mode example, both sessions are extracted as *BIN/TOC* files, so again we use *bchunk* to convert to *BIN/CUE*:

```bash
toc2cue no1.toc no1.cue
toc2cue no2.toc no2.cue
```

And then use *bchunk* to extract ISO and *WAVE* files from the *BIN/CUE* images:

```bash
bchunk -s -w no1.bin no1.cue no1
bchunk -s -w no2.bin no2.cue no2
```

One thing to watch out for is that in most cases ISO images from an enhanced CD cannot directly be accessed or mounted. The reason for this is that the sector offsets that point to the files in the image are defined *relative to the beginning of the physical disc*, and not *relative to the start of the image*! More details on this can be found [in this blog post]({{ BASE_PATH }}/2017/04/25/imaging-cd-extra-blue-book-discs), which also describes a workaround that allows one to access such images under Linux.

## Additional material

* The rough, unedited notes on which this blog post is based can be found [here](https://gist.github.com/bitsgalore/1bea8f015eca21a706e7#file-notescdimaging-md) (they contain some additional material that I left out here for readability). 

* The [User Manual of *ddrescue*](https://www.gnu.org/software/ddrescue/manual/ddrescue_manual.html#Optical-media) gives some useful additional examples of how this tool can be used to recover data from a faulty CD-ROM.  

## Update June 2019

This blog post was updated June 2019. The main changes are:

- Revised section on checksum verification, and added explanation that in practice this is not very useful.
- Added references to *Isolyzer*
- The section on mixed-mode and multisession discs confusingly mixed up both types of carriers. I have clarified the distinction between both types of carriers, the instructions for imaging them are now in separate sections.


[^1]: If you don't do this you will get the error message *ERROR: Unable to open SCSI device /dev/sr0: Device or resource busy*.

[^2]: Whether the resulting image will conform to ISO 9660 depends on the source medium, as the image is simply a byte-exact copy of the data on the physical carrier's file system. So for a DVD that uses the [UDF](https://en.wikipedia.org/wiki/Universal_Disk_Format) format, the ISO image will be UDF as well.

[^4]: If you don't do this you will end up with this error: *Error trying to open /dev/sr0 exclusively (Device or resource busy)... retrying in 1 second.*

[^5]: Strangely, in my tests a parse error occurred when I specified user-defined file names here. Also, it appeared that the summary log file resulted in more detailed output than the detailed one. This needs a more in-depth look!

[^6]: It is important that the names of the ISO and mapping file are identical to those used in the previous *ddrescue* run. This allows the tool to process *only* the problematic sectors (and skip everything else). 

[^7]: This is a pretty arbitrary value, and you can use whatever value you like.

[^8]: I initially omitted this, and ended up with *WAVE* files that all played as static noise! This is a bit odd, since according to its [specification](http://www-mmsp.ece.mcgill.ca/Documents/AudioFormats/WAVE/WAVE.html) the *WAVE* format is little-Endian by definition.


<hr>
Originally published at the [KB Research blog](http://blog.kbresearch.nl/2015/11/13/preserving-optical-media-from-the-command-line/)
