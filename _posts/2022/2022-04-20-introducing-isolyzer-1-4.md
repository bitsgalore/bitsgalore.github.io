---
layout: post
title: Introducing Isolyzer 1.4
tags: [isolyzer, optical-media, ISO-9660, HFS, High-Sierra, UDF]
comment_id: 79
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2022/04/cds.jpg" alt="Compact Discs still life">
</figure>

It's been a while since the last release of the [Isolyzer](https://github.com/KBNLresearch/isolyzer) tool, but after four years of near-inactivity I just published [Isolyzer 1.4](https://github.com/KBNLresearch/isolyzer/releases/tag/1.4.0). In this post I provide some background information on how this release came about, and I briefly explain the main changes.

<!-- more -->

## History of Isolyzer

For those unfamiliar with the Isolyzer tool, here's a brief recap. Isolyzer started its life in 2015. At that time I had just started working on optical media preservation and disc imaging. One of the problems I ran into, was that imaging optical media would occasionally result in incomplete (i.e. truncated) disc images. Worse, I found it near impossible to reliably identify these incomplete disc images with existing software tools. After a lot of digging into the specs, I figured out how to estimate expected file sizes using the block-level information in the ISO 9660 file system. I then applied this in a dedicated Python tool. Since many "ISO images" that exist in the wild are actually hybrids of different file systems (e.g. ISO 9660, Apple HFS or HFS+ and UDF), I gradually added support for those file systems as well. As a result, Isolyzer also became increasingly useful as a tool for extracting information about file systems inside ISO images. More details about Isolyzer's history can be found [here]({{ BASE_PATH }}/2017/01/13/detecting-broken-iso-images-introducing-isolyzer) and [here]({{ BASE_PATH }}/2017/07/12/update-on-isolyzer-udf-hfs-and-more).  

## Apple file system block size confusion

The initial trigger that started the work on this release was an email from Tyler Thorsted. He had run into a number of problematic ISO images of CD-ROMs that had been created with Roxio Toast, a widely used CD-burning software application for Apple Macintosh. Although these images contained an Apple HFS file system with Apple Partition maps, the underlying file systems were not properly identified and parsed by Isolyzer (and some other tools as well). After some poking around with a Hex editor, it turned out that the file system in the offending images was arranged into 2048-byte blocks, instead of the 512-byte blocks that were expected by Isolyzer. This seemed easy enough to fix: instead of assuming a fixed 512-byte block size, I changed the code to use the block size value from the "zero block" structure instead, and then use this to iterate over the partition maps.

## More block size confusion (and a better fix)

Sadly, Tyler noticed that the fixed code resulted in missing partition map output for some images that were handled correctly by Isolyzer prior to my fix. For this particular case, the culprit was that the block size value in the "zero block" (which read 2048 bytes) didn't correspond to the partition map data (which were arranged into 512 byte blocks). The "official" documentation that I could find provided some rather conflicting information on the correct implementation of partition maps, and their relation to the zero block. In the end I worked around this by using a simple iterative procedure that looks for partition maps at a number of pre-defined byte offsets into the image. The resulting match at the first (smallest) byte offset then gives the correct, actual block size. More details and links to sample files can be found in the (now closed) [issue on Github](https://github.com/KBNLresearch/isolyzer/issues/22).

## High Sierra file system

Tyler also made a [feature request](https://github.com/KBNLresearch/isolyzer/issues/26) for support of the ["High Sierra" file system](https://web.archive.org/web/20220111023846/https://www.os2museum.com/files/docs/cdrom/CDROM_Working_Paper-1986.pdf). This was the de facto standard CD-ROM file system for a few years during the late eighties, before it was made redundant by ISO 9660. The High Sierra format is very similar to ISO 9660, and shares the same data structures and fields (although the fields often appear in a different order). This made it relatively easy to add support for it in Isolyzer. As before, see the [Github issue](https://github.com/KBNLresearch/isolyzer/issues/26) for more details and a link to a sample file.

## XML schema

I also created an [XML schema](https://github.com/KBNLresearch/isolyzer/blob/main/xsd/isolyzer-v-1-0.xsd) that makes it possible to validate Isolyzer's output, and added a namespace definition. This should also make it easier to embed Isolyzer's output into other XML files, if needed. I should stress that the schema has only had limited testing so far, so please get in contact (or [report an issue](https://github.com/KBNLresearch/isolyzer/issues/new/choose)) if you come across unexpected behaviour.

## Native wildcard expansion on non-Windows platforms

One long-standing annoyance (at least for me) has been Isolyzer's handling of wildcard expressions at the command-line, which needed to be wrapped in quotation marks on Linux to work correctly. This is rooted in the slightly different ways wildcards are handled by Linux and Windows, respectively. This release fixes this by delegating wildcard expansion to the operating system for non-Windows operating systems. On Windows, Isolyzer still takes care of wildcard expansion (as it did before), since Windows does not do this natively.

## Python 2 no longer supported

This release also no longer provides support for Python 2 (which is now obsolete), so you'll have to use Python 3. Windows users can also use the stand-alone binaries, which don't require Python at all.

## Other changes

In addition to the above changes, this release also includes several minor bug fixes. I have also expanded the set of test files, and set up some unit tests that use these tests files. These changes are all invisible to the user, but they make the Isolyzer release process more straightforward.

## Installation

For a fresh single-user install with pip use: 

```bash
pip install isolyzer --user
```

To upgrade an existing version of Isolyzer, use:

```bash
pip install isolyzer --upgrade --user
```

Alternatively, Windows users can use the binaries that are available from [the release page](https://github.com/KBNLresearch/isolyzer/releases/tag/1.4.0). As always, these binaries are completely stand-alone, and don’t require Python on your machine.

## Feedback welcome

As always, feedback on this release is very welcome. Please feel free to [report an issue](https://github.com/KBNLresearch/isolyzer/issues/new/choose) if anything doesn't work as expected!

## Acknowledgements

Thanks are due to Tyler Thorsted for the useful online discussions on the Apple and High Sierra issues, and for providing the necessary test files.

## Revision history

- 7 June 2022: changed all release candidate references to the final 1.4.0 release.

## Further resources

- [Isolyzer 1.4.0 release page](https://github.com/KBNLresearch/isolyzer/releases/tag/1.4.0)
- [Isolyzer on Github (with documentation)](https://github.com/KBNLresearch/isolyzer)
