---
layout: post
title: Magic editing and creation&#58; a primer
tags: [format-identification,magic,unix-file]
comment_id: 58
---

The purpose of this post is to give a brief introduction to creating, editing and submitting format signatures (or '*magic*' entries) for the well-known *File* tool. The occasion for this was some work I did last week on improving *File*'s identification of the *JPEG 2000* formats. I had some difficulty finding any easy-to-follow documentation that describes how to do this. The information is all out there, but it's pretty fragmented. So, I wrote this brief tutorial, which is intended as an accessible introduction to *magic* editing. It only covers the very basics, but hopefully this is enough to overcome some initial stumbling blocks.

<!-- more -->

## How to get File

*File* is part of most *Unix*/*Linux* distributions. If you're a *Windows* user I would recommend to get [Cygwin][cygwin], which includes the latest version of *File*. A stand-alone *Windows* port of *File* does exist, but apparently it is not maintained anymore.

## How it works

Just like tools such as *DROID*, *Fido* and *Apache Tika*, *File*'s identification is based on format signatures (which in the case of *File* are usually called *magic numbers*. These are stored in a *magic* file, which is located in the *magic* directory. Typical locations of this *magic* directory are:

```data
/usr/share/misc
```

Or:

```data
/usr/share/file
```

Or, on CygWin:

```data
/cygwin/usr/share/misc/
```

## Files in the magic directory: compiled versus source

Inside the *magic* directory you will see a file called  *magic.mgc*. This is a *compiled* *magic* file. This is the one that *File* uses by default (although you can override this behaviour, which we'll see in a moment).

Depending on your system, you may also see another file that is simply called *magic*. This is the uncompiled (i.e. human-readable) source of the *magic* file.

Note that in the source distribution of *File*, the source *magic* is organised in a directory structure. You can see this in the [Github mirror of File's source repository][magDir].

## Compiling a magic source file

So how do we get from a source file to a compiled *magic* file? Easy: just run *File* with the `-C` (compile) switch, and use the `-m` switch to specify the source file. For example, supposing we have a source file called *myMagic*, compile it using:

```bash
file -C -m myMagic
```

This produces the compiled *magic* file *myMagic.mgc*. You can then use the compiled file by specifying its name using (again) the `-m` switch, e,.g.:

```bash
file -i -m myMagic *
```

Note that in the above case *File* actually uses *myMagic.mgc* (apparently it expects this extension); the following command line:

```bash
file -i -m myMagic.mgc *
```

produces identical results.

## Format of the magic source file

So what does the *magic* source file look like? I will only stick to the basics here, as an exhaustive description of the *magic* source format is given in the  [man pages][magicManPage] of *magic*.

The most important thing to remember is that each line in the file specifies a test. Each test is made up of 4 items, which are separated by one or more whitespace characters (usually tabs, but spaces appear to work as well):

+ *offset* - specifies the offset, in bytes, into the file of the data which is to be tested. 
+ *type* - the type of the data to be tested (see [here][magicManPage] for a list of all possible values) . 
+ *test* - the value to be compared with the value from the file.
+ *message* - the message to be printed if the comparison succeeds

Here's an example (actually this is the *JPEG 2000* *magic* that is currently used in *File* 5.11):

```data
0	string	\x00\x00\x00\x0C\x6A\x50\x20\x20\x0D\x0A\x87\x0A	JPEG 2000
```

Here we have a test that compares the start of a file object (byte offset 0) against a 12-character string (here represented as hexadecimal codes). If the pattern is found, the text *JPEG 2000* will be printed to screen. Note that for most formats we will need further *sublevel tests*, which I will illustrate in the next section. Also, even though this introduction only decribes tests at fixed byte-positions, more sophisticated tests are possible as well. See the [man pages][magicManPage] for details on this.

## Improving the JPEG 2000 magic

The main problem of the above *JPEG 2000* entry is that it only checks for the first 12 bytes of a file. This is enough for establishing that a file is part of the *JPEG 2000* 'family' of file formats, but it doesn't tell you the exact sub-format, which can be any of the following:

+ *JP2* (basic still image format)
+ *JPX* (extended still image format)
+ *JPM* (compound format)
+ *MJ2* (Motion *JPEG 2000*)

In other words: although *File* may be telling us that a file object matches *JPEG 2000*, this gives us zero information on whether it contains simple image data (*JP2*) or video content (*MJ2*)! Fortunately, the headers of each of the above formats include a *Brand* field, which is a 4-byte string of characters that uniquely identifies each format. This string starts at byte 20, and for *JP2* it equals '\x6a\x70\x32\x20', so we can simply add this as a second test to the existing *magic* entry:

```data
>20	string	\x6a\x70\x32\x20	Part 1 (JP2)
```

Note the "`>`" character, which indicates that this is a higher-level test on top of the previous one.

## Adding mimetype information

Optionally, tests may also be associated with a *mimetype*. In that case the line that represents the test is followed by a second line, which contains the following items:

+ *!:mime* - indicates that this line is a *mimetype* declaration
+ *MIME Type* - the actual *mimetype*

For *JP2* this is:

```data
!:mime	image/jp2
```

## Adding the other formats

Repeating the above procedure for the full set of *JPEG 2000* formats we end up with this:

```data
0	string	\x00\x00\x00\x0C\x6A\x50\x20\x20\x0D\x0A\x87\x0A	JPEG 2000
>20	string	\x6a\x70\x32\x20	Part 1 (JP2)
!:mime	image/jp2
>20	string	\x6a\x70\x78\x20	Part 2 (JPX)
!:mime	image/jpx
>20	string	\x6a\x70\x6d\x20	Part 6 (JPM)
!:mime	image/jpm
>20	string	\x6d\x6a\x70\x32	Part 3 (MJ2)
!:mime video/mj2
```

## Compiling the file

To create the compiled file, follow the simple steps below::

1. Save your *magic* entry to a file (e.g. *jpeg2000Magic*). See also the [complete entry on my personal Github repo][magicJvdK].

2. If you're using *Windows*, you may need to convert *Windows*-style linebreaks to *Unix* linebreaks, for which you can use the *dos2unix* tool (which is included in *Cygwin*):

```bash
dos2unix jpeg2000Magic
```

3. Now compile the file:

```bash
file -C -m jpeg2000Magic
```

*Et voilà*, our compiled *magic* file is ready for use!

## Testing it

For testing, run *File* on any number of files, e.g.:

```bash
file -i -m jpeg2000Magic *
```

Here's an example of the output you may get:

```data
balloon.jp2:  image/jp2; charset=binary
balloon.jpf:  image/jpx; charset=binary
balloon.jpm:  image/jpm; charset=binary
Speedway.mj2: video/mj2; charset=binary
```

## Submitting magic

File's [source repository on Github][githubFile] gives a number of guidelines for submitting *magic*. *Don't* use the Github repo for submitting stuff or commenting, as it is just a read-only mirror! Instead, make your submits through the [*File* mailing list][fileList], or use the [bug tracker][bugtracker].

## Useful links

+ [Fine Free File Command][ffFile]  
+ [Documentation of the *magic* file][magicManPage]  
+ [Read-only mirror of *File* CVS repository, updated nightly][githubFile]  
+ [*File* mailing list][fileList]  
+ [Cygwin][cygwin]  
+ [MIME Media Types at the Internet Assigned Numbers Authority][mimeTypes]  
+ [Link to updated *JPEG 2000* *magic*][magicJvdK]


[magDir]: https://github.com/glensc/file/tree/master/magic/Magdir
[cygwin]:http://www.cygwin.com/

[ffFile]:http://www.darwinsys.com/file/

[magicManPage]:http://manpages.ubuntu.com/manpages/precise/en/man5/magic.5.html

[mimeTypes]:http://www.iana.org/assignments/media-types/index.html
[fileList]:http://mx.gw.com/mailman/listinfo/file

[bugtracker]:http://bugs.gw.com/my_view_page.php
[magicJvdK]:https://github.com/bitsgalore/jp2kMagic/blob/master/magic/jpeg2000Magic
[githubFile]:https://github.com/glensc/file

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2012/08/09/magic-editing-and-creation-primer/)
