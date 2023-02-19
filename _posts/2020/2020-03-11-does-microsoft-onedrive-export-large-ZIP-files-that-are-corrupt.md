---
layout: post
title: Does Microsoft OneDrive export large ZIP files that are corrupt?
tags: [ZIP, preservation-risks, Microsoft, OneDrive]
comment_id: 70
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2020/03/broken-zip.jpg" alt="Broken zip £12.50">
  <figcaption><a href="https://www.flickr.com/photos/dichohecho/3810699621/">“Broken zip £12.50”</a> by <a href="https://www.flickr.com/photos/dichohecho/">dichoecho</a>, used under <a href="https://creativecommons.org/licenses/by/2.0/">CC BY</a> / Cropped from original.</figcaption>
</figure>

We recently started using [Microsoft OneDrive](https://en.wikipedia.org/wiki/OneDrive) at work. The other day a colleague used OneDrive to share a folder with a large number of ISO images with me. Since I wanted to work with these files on my Linux machine at home, and no official OneDrive client for Linux exists a this point, I used OneDrive's web client to download the contents of the folder. Doing so resulted in a 6 GB [ZIP](https://en.wikipedia.org/wiki/Zip_(file_format)) archive. When I tried to extract this ZIP file with my operating system's (Linux Mint 19.3 MATE) archive manager, this resulted in an error dialog, saying that "An error occurred while loading the archive":

![]({{ BASE_PATH }}/images/2020/03/archive-manager-onedrive.png)

The output from the underlying extraction tool ([*7-zip*](https://en.wikipedia.org/wiki/7-Zip)) reported a "Headers Error", with an "Unconfirmed start of archive". It also reported a warning that "There are data after the end of archive". No actual data were extracted whatsoever. This all looked a bit worrying, so I decided to have a more in-depth look at this problem. 

<!-- more -->

## Extracting with Unzip

As a first test I tried to extract the file from the terminal using *unzip* (v. 6.0) using the following command:

```bash
unzip kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip
```

This resulted in the following output:

```
Archive:  kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip
warning [kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip]:  1859568605 extra bytes at beginning or within zipfile
  (attempting to process anyway)
error [kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip]:  start of central directory not found;
  zipfile corrupt.
  (please check that you have transferred or created the zipfile in the
  appropriate BINARY mode and that you have compiled UnZip properly)
```

So, according to *unzip* the file is simply corrupt. *Unzip* wasn't able to extract any actual data.

## Extracting with 7-zip

Next I tried to  to extract the file with *7-zip* (v. 16.02) using this command:

```bash
7z x kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip
```

This resulted in the following (lengthy) output:

```
7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,4 CPUs Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz (506E3),ASM,AES-NI)

Scanning the drive for archives:
1 file, 6154566547 bytes (5870 MiB)

Extracting archive: kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip

ERRORS:
Headers Error
Unconfirmed start of archive


WARNINGS:
There are data after the end of archive

--
Path = kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip
Type = zip
ERRORS:
Headers Error
Unconfirmed start of archive
WARNINGS:
There are data after the end of archive
Physical Size = 4330182775
Tail Size = 1824383772

ERROR: CRC Failed : kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f/afd3f61a-5e0e-11ea-ab97-40b0341fbf5f/08.wav
                                                                              
Sub items Errors: 1

Archives with Errors: 1

Warnings: 1

Open Errors: 1

Sub items Errors: 1
```

Here we see the familiar "Headers Error" and "Unconfirmed start of archive" errors, as well as a warning about a [cyclic redundancy check](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) that failed on an extracted file. Unlike *unzip*, *7-zip* does succeed in extracting some of the data, but seeing that the size of the extracted folder is only 4.3 GB, the extraction is incomplete (the size of the ZIP file is 6 GB!).

## 4 GiB size limit and ZIP64

At this point I started wondering if these issues could be related to the *size* of this particular ZIP file, especially since I have been able to process zipped OneDrive folders before without any problems. The [Wikipedia entry on *ZIP*](https://en.wikipedia.org/wiki/Zip_(file_format)#ZIP64) states that originally the format had a 4 GiB limit on the total size of the archive (as well as both the uncompressed and compressed size of a file). To overcome these limitations, a "ZIP64" extension was added to the format in [version 4.5 of the ZIP specification](https://web.archive.org/web/20011203085830/http://www.pkware.com/support/appnote.txt) (which was published in 2001). To be sure, I verified that both *unzip* and *7-zip* on my machine support ZIP64[^1].

## Small OneDrive ZIP and home-rolled large ZIP

I did some additional tests to verify if my problem could be a ZIP64-related issue. First I downloaded a smaller (<4 GB) folder from OneDrive, and tried to extract the resulting ZIP file with *unzip* and *7-zip*. Both were able to extract the file without any issues. Next I created two 8 GB ZIP files from data on my local machine with both the *zip* and *7-zip* tools. I then tried to extract both files with both *unzip* and *7-zip* (i.e. I extracted each file with both tools). Again, both extracted these files without any problems. Since these tests demonstrate that both *unzip* and *7-zip* are able to handle both large ZIP files (which by definition use ZIP64) as well as smaller OneDrive ZIP files, this suggests that something odd is going on with OneDrive's implementation of ZIP64.

## Testing the ZIP file integrity

The *zip* tool has a switch that can be used to test the integrity of a ZIP file. I ran it on the problematic file like this:

```bash
zip -T kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip
```

Here's the result:

```
Could not find:
  kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.z01

Hit c      (change path to where this split file is)
    q      (abort archive - quit)
 or ENTER  (try reading this split again): 
```

So, apparently the *zip* utility thinks this is a multi-volume archive (which it isn't). Running this command on any of my other test files (the small OneDrive file, and the large files created by *zip* and *7-zip*) didn't result in any errors.

## Tests with Python's zipfile module

The Python programming language by default includes a [*zipfile*](https://docs.python.org/3/library/zipfile.html) module, which has tools for reading and writing ZIP files. So, I wrote the following script, which opens the ZIP file in read mode, and then reads its contents (I used Python 3.6.9 for this):

```python
import zipfile

# Open ZIP file
myZip = zipfile.ZipFile("kb-4d8a2f9a-5e0b-11ea-9376-40b0341fbf5f.zip",
                        mode='r')

# Read all files in archive and check their CRCs and file headers.
myZip.testzip()

# Close the ZIP file
myZip.close()
```

Running the script raised the following error:

```
zipfile.BadZipFile: zipfiles that span multiple disks are not supported
```

This looks somewhat related to the outcome of *zip*'s integrity test, which reported a multi-volume archive. Looking at the [source code of the zipfile module](https://github.com/python/cpython/blob/d3af92ecc2f41d920e9a66211e2ab631fc473163/Lib/zipfile.py#L232) shows that this particular error is raised if a check on 2 data fields from the "zip64 end of central dir locator" fails:

```python
if diskno != 0 or disks > 1:
  raise BadZipFile("zipfiles that span multiple disks are not supported")
```

Here's the description of this data structure in the [format specification](https://pkware.cachefly.net/webdocs/APPNOTE/APPNOTE-6.3.6.TXT):

```
zip64 end of central dir locator 
      signature                       4 bytes  (0x07064b50)
      number of the disk with the
      start of the zip64 end of 
      central directory               4 bytes
      relative offset of the zip64
      end of central directory record 8 bytes
      total number of disks           4 bytes
```

Python's *zipfile* raises the error if either the value of the "number of the disk with the start of the zip64 end of central directory" field (variable *diskno*) isn't equal to 0, or the "total number of disks" (variable *disks*) is larger than 1. So, I opened the file in a Hex editor, and zoomed in on the "zip64 end of central dir locator": 

![]({{ BASE_PATH }}/images/2020/03/onedrive-hex.png)

Here, the highlighted bytes (`0x504b0607`) make up the signature of the "zip64 end of central dir locator"[^2]. The 4 bytes inside the blue rectangle contain the "number of the disk" value. Here, its value is 0, which is the correct and expected value. The 4 bytes inside the red rectangle contain the "total number of disks" value, which is also 0. But this is really odd, since neither value should trigger the "zipfiles that span multiple disks are not supported" error! Also, a check on the 8 GB ZIP files that I had created myself with *zip* and *7-zip* showed both to have a value of 1 for this field. So what's going on here?

## Digging into zipfile's history

The most likely explanation I could think of, was some difference between my local version of the Python *zipfile* module and the latest published version on Github. Using Github's [blame view](https://help.github.com/en/github/managing-files-in-a-repository/tracking-changes-in-a-file), I inspected the revision history of the part of the check that raises the error. This revealed [a recent change to *zipfile*](https://github.com/python/cpython/commit/ab0716ed1ea2957396054730afbb80c1825f9786): prior to a patch that was submitted in May 2019, the offending check was done slightly differently:

```python
if diskno != 0 or disks != 1:
  raise BadZipFile("zipfiles that span multiple disks are not supported")
```

Note that in the old situation the test would fail if *disks* was any value other than 1, whereas in the new situation it only fails if *disks* is greater than 1. Given that for our OneDrive file the value is 0, this explains why the old version results in the error. The Git commit of the patch also includes the following note:

> Added support for ZIP files with disks set to 0. Such files are commonly created by builtin tools on Windows when use ZIP64 extension.

So could this be the vital clue we need to solve this little file format mystery? Re-running my Python test script with the latest version of the *zipfile* module did not result in any reported errors, so this looked hopeful for a start. But is the 0 value of "total number of disks" also the thing that makes unzip and 7-zip choke?

## Hacking into the OneDrive ZIP file

To put this to the test, I first made a copy of the OneDrive ZIP file. I opened this file in a Hex editor, and did a search on the hexadecimal string `0x504b0607`, which is the signature that indicates the start of the "zip64 end of central dir locator"[^3]. I then changed the first byte of the "total number of disks" value (this is the 13th byte after the signature, indicated by the red rectangle in the screenshot) from `0x00` to `0x01`:

![]({{ BASE_PATH }}/images/2020/03/onedrive-hex-fixed.png)

This effectively sets the "total number of disks" value to 1 (unsigned little-endian 32-bit value). After saving the file, I repeated all my previous tests with *unzip*, *7-zip*, as well as *zip*'s integrity check. The modified ZIP file passed all these tests without any problems! The contents of the file could be extracted normally, and the extraction is also complete. The file can also be opened normally in Linux Mint's archive manager, as this screenshot shows:

![]({{ BASE_PATH }}/images/2020/03/onedrive-fixed-archive-manager.png)

So, it turns out that the cause of the problem is the value of one field in the "zip64 end of central dir locator", which can be provisionally fixed by nothing more than changing one single bit!

## Other reports on this problem

If a widely-used platform by a major vendor such as Microsoft produces ZIP archives with major interoperability issues, I would expect others to have run into this before. [This question on *Ask Ubuntu*](https://askubuntu.com/questions/1115238/unable-to-extract-onedrive-zip-file-on-ubuntu-18-10) appears to describe the same problem, and [here's another report on corrupted large ZIP files](https://onedrive.uservoice.com/forums/913528-onedrive-on-the-web/suggestions/35321278-fix-the-corrupted-zip-download-or-be-honest-about) on a OneDrive-related forum. On Twitter Tyler Thorsted [confirmed](https://twitter.com/CHLThor/status/1237416651995283457) my results for a 5 GB ZIP file downloaded from OneDrive, adding that the Mac OS X archive utility didn't like the file either. Still, I'm surprised I couldn't find much else on this issue.

## Similarity to Apple Archive utility problem

The problem looks superficially similar to an older issue with Apple's Archive utility, which would write corrupt ZIP archives for cases where the ZIP64 extension is needed. From the [WikiPedia entry on *ZIP*](https://en.wikipedia.org/wiki/Zip_(file_format)#ZIP64):

> Mac OS Sierra's Archive Utility notably does not support ZIP64, and can create corrupt archives when ZIP64 would be required.

More details about this are available [here](https://web.archive.org/web/20140331005235/http://www.springyarchiver.com/blog/topic/topic/203), [here](https://apple.stackexchange.com/questions/221020/large-zip-files-created-in-os-x-cannot-be-opened-in-windows) and [here](https://github.com/thejoshwolfe/yauzl/issues/69). Interestingly a [Twitter thread by Kieran O'Leary](https://twitter.com/kieranjol/status/1200118816686137344) put me on track of this issue (which I hadn't heard of before). It's not clear to me if the OneDrive problem is identical or even related, but because of the similarities I thought it was at least worth a mention here.

## Conclusion

The tests presented here demonstrate how large ZIP files exported from the Microsoft OneDrive web client cannot be read by widely-used tools such as *unzip* and *7-zip*. The problem only occurs for large (> 4 GiB) files that use the ZIP64 extension. The cause of this interoperability problem is the value of the "total number of disks" field in the "zip64 end of central dir locator". In the OneDrive files, this value is set to 0 (zero), whereas most reader tools expect a value of 1. It is debatable whether the OneDrive files violate the [ZIP format specification](https://pkware.cachefly.net/webdocs/APPNOTE/APPNOTE-6.3.6.TXT), since the spec doesn't say anything about the permitted values of this field. Affected files can be provisionally "fixed" by changing the first byte of the "total number of disks" field in a hex editor. However, to ensure that existing files that are affected by this issue remain accessible in the long term, we need a more structural and sustainable solution. It is probably fairly trivial to modify existing ZIP reader tools and libraries such as *unzip* and *7-zip* to deal with these files. I'll try to get in touch with the developers of some of these tools about this issue. Ideally things should also be fixed on Microsoft's end. If any readers have contacts there, please bring this post to their attention!

## Test file

I've created an openly-licensed test file that demonstrates the problem. It is available here:

<https://zenodo.org/record/3715394>

## Update (17 March 2020)

For *unzip* I found [this ticket](https://sourceforge.net/p/infozip/bugs/42/) on the *Info-Zip* issue tracker, which looks identical to the problem discussed in this post. The ticket was already created in 2013, but its current status is not entirely clear.

For *7-zip*, things are slightly complicated by the fact that for Unix [a separate *p7zip* port](https://sourceforge.net/projects/p7zip/) exists, which currently is 3 major releases behind the [main 7-zip](https://sourceforge.net/projects/sevenzip/) project. In any case, I've just opened [this feature request](https://sourceforge.net/p/p7zip/feature-requests/46/) in the *p7zip* issue tracker.

Meanwhile Andy Jackson has been trying to [get this issue to the attention of Microsoft](https://twitter.com/anjacks0n/status/1238852027045883904), so let's see what happens from here.

## Fix-OneDrive-Zip script (update 8 June 2020)

In the comments section, Paul Marquess posted a link to a small Perl script he wrote that automatically updates the "total number of disks" field of a problematic OneDrive ZIP file. The script is available here:

<https://github.com/pmqs/Fix-OneDrive-Zip>

I ran a quick test with my openly-licensed test file, using the following command:

```bash
fix-onedrive-zip onedrive-zip-test-zeros.zip
```

After running the script, the file was indeed perfectly readable. Thanks Paul!

## Revision history

- 14 March 2020: added analysis with Python *zipfile*, and updated conclusions accordingly.
- 17 March 2020: added update with links to Info-Zip and p7zip issue trackers.
- 18 March 2020: added link to test file.
- 8 June 2020: added reference to *Fix-OneDrive-Zip* script by Paul Marquess.

[^1]: For unzip you can check this this by running it with the `--version` switch. If the output includes `ZIP64_SUPPORT` this means ZIP64 is supported.

[^2]: Note that this is the big-endian representation of the signature, whereas the ZIP formation specification uses little-endian representations. See more on endianness [here](https://en.wikipedia.org/wiki/Endianness).

[^3]: Since the "zip64 end of central dir locator" is located near the end of the file, the quickest way to find it is to scroll to the very end of the file in the Hex editor, and then do a reverse search ("Find Previous") from there.