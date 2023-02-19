---
layout: post
title: Update on Isolyzer&#58; UDF, HFS+ and more!
tags: [isolyzer,optical-media,ISO-9660,HFS,UDF]
comment_id: 21
---
Earlier this year I [blogged about *Isolyzer*]({{ BASE_PATH }}/2017/01/13/detecting-broken-iso-images-introducing-isolyzer), a tool designed to help the detection of broken ISO images. Today I released a shiny new [beta version](https://github.com/KBNLresearch/isolyzer) that adds a significant amount of new functionality. Below is an overview of the main changes, followed by some warnings and caveats.

<!-- more -->

## Support of more file systems

Where previous versions only supported disc images with an [*ISO 9660*](https://en.wikipedia.org/wiki/ISO_9660) file system (with limited support for hybrid *ISO 9660*/Apple *HFS* file systems), the new release can deal with a much broader range of file systems. In particular, it adds support for the [*Universal Disk Format*](https://en.wikipedia.org/wiki/Universal_Disk_Format) (*UDF*) and Apple's [*HFS+*](https://en.wikipedia.org/wiki/HFS_Plus) file system. Unlike previous versions, *Isolyzer* can now also deal with Apple disc layouts that don't contain a [partition map](https://en.wikipedia.org/wiki/Apple_Partition_Map) (see also [here](https://en.wikipedia.org/wiki/Hybrid_disc#Multiple_file_systems) for more details on Apple disc layouts). Crucially, all of the above are supported both as stand-alone file systems (e.g. a CD image with exclusively a *HFS+* file system) as well as in various hybrid configurations (e.g. [*UDF Bridge*](http://www.afterdawn.com/glossary/term.cfm/udf_bridge) format).

Details on how *Isolyzer* performs the calculations for estimating the expected image size in each of these situations can be found [here](https://github.com/KBNLresearch/isolyzer#calculation-of-the-expected-file-size). As it turns out, for UDF this calculation is not as straightforward as one would expect. The result is that *Isolyzer* typically under-estimates the true image size by several sectors. In most cases this is unlikely to be a problem; nevertheless, it may be possible to improve this in future versions.

## Changes to the output format

The addition of multiple file system support required some changes to *Isolyzer*'s output format. The main change is the addition of a *fileSystems* element, which in turn holds one or more *fileSystem* elements that each contain information about a detected file system. The updated output format is [documented here](https://github.com/KBNLresearch/isolyzer#isolyzer-output), and some examples are [available here](https://github.com/KBNLresearch/isolyzer#examples). Note that these changes may break existing workflows that use *Isolyzer*. This is also the main reason for making this a major (1.x) release. It is still possible that the format will change somewhat in the final (stable) release; this will largely depend on any feedback we may get from users of the tool.

## Documentation and test images

Finally *Isolyzer*'s documentation has been given a major overhaul, as you can see from [the main Github page](https://github.com/KBNLresearch/isolyzer/). The repo now includes a [new set of small test images](https://github.com/KBNLresearch/isolyzer/tree/master/testFiles) that cover all of the currently supported file systems (with the exception of *HFS*/*HFS+* file systems with a partition map, for which I've been unable to create or find a sufficiently small sample).

## Installation

You can install *Isolyzer* with [*pip*](https://en.wikipedia.org/wiki/Pip_(package_manager)), using the following command:

```bash
pip install isolyzer --user
```

For Windows users [64- and 32-bit binaries are available here](https://github.com/KBNLresearch/isolyzer/releases/tag/1.0.0). These binaries are completely stand-alone and don't require Python on your machine.

## Feedback appreciated

As always feedback on *Isolyzer* is highly appreciated, if only because any problem its users come across we'll probably run into ourselves at some point! (Please note though that because of upcoming holidays I may be a little slow in following up any queries.) 

## Links

* [*Isolyzer* on Github](https://github.com/KBNLresearch/isolyzer/)
* [Windows binaries (64- and 32-bit)](https://github.com/KBNLresearch/isolyzer/releases/tag/1.0.0))

## Postscript (2 February 2019)

Removed some outdated and possibly confusing information from the "Installation" section.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2017/07/12/update-on-isolyzer-udf-hfs-and-more/)
