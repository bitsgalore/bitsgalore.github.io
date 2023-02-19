---
layout: post
title: A simple JP2 file structure checker
tags: [jpeg-2000,JP2,JHOVE,jpylyzer]
comment_id: 65
---

Over the last few weeks I've been working on the design of a workflow
that the KB is planning to use for the migration of a collection of
(mostly old) TIFF images to JP2. One major risk of such a migration is
that hardware failures during the migration process may result in
corrupted images. For instance, one could imagine a brief network or
power interruption that occurs while an image is being written to disk.
In that case data may be missing from the written file. Ideally we would
be able to detect such errors using format validation tools such as
[JHOVE](http://hul.harvard.edu/jhove/). Some time ago Paul Wheatley
reported that the BL at some point were dealing with corrupted,
incomplete JP2 files that were nevertheless deemed "well-formed and
valid" by JHOVE. So I started doing some experiments in which I
deliberately butchered up some images, and subsequently checked to what
extent existing tools would detect this.

<!-- more -->

I started out with removing some trailing bytes from a lossily
compressed JP2 image. As it turned out, I could remove most of the image
code stream (reducing the original 2 MB image to a mere 4 kilobytes!),
but JHOVE would still say the file was "well-formed and valid". I was
also able to open and render these files with viewer applications such
as Adobe Photoshop, Kakadu's viewer and Irfanview. The behaviour of the
viewer apps isn't really a surprise, since the ability to render an
image without having to load the entire code stream is actually one of
the features that make JPEG 2000 so interesting for many access
applications. JHOVE's behaviour was a bit more surprising, and perhaps
slightly worrying.

## jp2StructCheck tool

This made me wonder about a way to detect incomplete code streams in JP2
files. A quick glance at the
[standard](http://www.jpeg.org/public/15444-1annexi.pdf) revealed that
image code streams should always be terminated by a two-byte 'end of
codestream marker'. As this is something that is straightforward to
check, I fired up [Python](http://www.python.org/) and ended up writing
a very simple [JP2 file structure
checker](https://github.com/bitsgalore/jp2StructCheck). Since the image
code stream in JP2 does not have to be located at the end of the file
(even though it usually is), it is necessary to do a superficial parsing
of JP2's 'box' structure (which is documented
[here](http://www.jpeg.org/public/15444-1annexi.pdf)). So I thought I
might as well include an additional check that verifies if the JP2
contains all required boxes.

In brief, when *jp2StructCheck* analyses a file, it first parses the
top-level box structure, and collects the unique identifiers (or marker
codes) of all boxes. If it encounters the box that contains the code
stream, it checks if the code stream is terminated by a valid
end-of-codestream marker. Finally, it checks if the file contains all
the compulsory/required top-level boxes. These are:

- JPEG 2000 signature box
- File Type box
- JP2 Header box
- Contiguous Codestream box

In order to test the box checking mechanism I did some additional image
butchering, where I deliberately changed the tags of existing boxes so
that they wouldn't be recognised. When I subsequently ran these images
through JHOVE, this revealed some additional surprises. For instance,
after changing the markers of the Contiguous Codestream box or even the
JP2 Header box (which effectively makes them unrecognisable), JHOVE
would still report these images as "well-formed and valid" (although in
the case of the missing JP2 Header box JHOVE did report an error).

## Limitations of jp2StructCheck

It is important to note here that *jp2StructCheck* only checks the
top-level boxes. In case of a superbox (which is a box that contains
child boxes), it does not recurse into its child boxes. For example, it
does not check if a JP2 Header box (which is a superbox) contains a Bits
Per Component Box (which is required by the standard). So the scope of
the tool is limited to a rather superficial check of the general file
structure. It is *not* a JP2 validator, and it is certainly not a
replacement for JHOVE (which performs a more in-depth analysis)! The
main scope is to be able to detect certain types of file corruption that
may occur as a result of hardware failure (e.g. network interruptions)
during the creation of an image.

In addition, the fact that a code stream is terminated by and
end-of-codestream marker is no guarantee that the code stream is
complete. For instance, if due to some hardware failure some part of the
middle of the codestream is not written, *jp2StructCheck* will not
detect this! It may be possible to improve the level of error detection
by including additional codestream markers. This is something I might
have a look at at some later point.

## Downloads

I created a [Github
repository](https://github.com/bitsgalore/jp2StructCheck) that contains
the source code of *jp2StructCheck*, some documentation, and a small
data set with some test images.

As some people may not want to install Python on their system, I also
created a [binary distribution that should work on most Windows
systems](https://github.com/downloads/bitsgalore/jp2StructCheck/jp2StructCheck31082011distWin32.zip).

The documentation (in PDF format) is
[here](https://github.com/downloads/bitsgalore/jp2StructCheck/jp2StructCheck.pdf).

Finally, use [this
link](https://github.com/downloads/bitsgalore/jp2StructCheck/testImages.zip)
to download the test images.

## Final notes

I'm curious to hear if anyone finds *jp2StructCheck* useful at all, so
please feel free to use the comment fields below for your feedback
(including reports on any bugs that may exist).

## Post script, February 2019

The *jp2StructCheck* tool is superseded by [*jpylyzer*](http://jpylyzer.openpreservation.org/)
(of which *jp2StructCheck* was an early precursor). Unlike *jp2StructCheck*, *jpylyzer* is a
full-fledged validator for the *JP2* format.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2011/09/01/simple-jp2-file-structure-checker/)
