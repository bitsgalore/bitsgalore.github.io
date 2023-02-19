---
layout: post
title: Improved identification of XML&#58; a Python experiment
tags: [format-identification,unix-file,DROID,Fido]
comment_id: 66
---

As a part of the [SCAPE](http://www.scape-project.eu) project, I'm
currently heavily involved in the evaluation of various file format
identification tools. The overall aim of this work is to determine which
tools are suitable candidates for inclusion in the SCAPE architecture.
In addition, we're also trying to get a better idea of each tool's
specific strengths and weaknesses, which will hopefully serve as useful
input to the developers community. We're actually planning to publish
the first results of this work on the OPF blog some time soon, so you
may want to keep your eyes peeled for that.

<!-- more -->

## Identification using byte signatures

In this blog entry I will focus on one particular area in which most
identification tools appear to be struggling: the identification of XML
files. Most identification tools try to establish a file's format by
looking for characteristic byte sequences, or 'signatures'. Examples of
tools that use this approach are
[DROID](http://sourceforge.net/projects/droid/),
[Fido](https://github.com/openplanets/fido) and the [Unix File
tool](http://darwinsys.com/file/). Signature-based identification works
well for most binary formats, but for text-based formats the results are
often less reliable. This also applies to XML. Signature-based tools
typically identify XML by the presence of an XML declaration, which, in
its simplest form, looks like this:

```xml
<?xml version="1.0"?>
```

The problem is that not all XML files actually contain an XML
declaration. Also, the use of an XML declaration is not mandatory.
The[XML specification](http://www.w3.org/TR/xml/) states that for a file
to qualify as "valid" XML it *should* contain the declaration. This
merely means that the use of the declaration is recommended (which
follows from the use of the word *"should"* and not *"must"*). XML files
that don't contain the declaration are by definition not "valid", but
they may still be "well-formed".

However, if (part of) the declaration is used as a signature, this means
that any files that don't have the declaration will not be identified as
XML by any of the above tools. This is exactly what happened in our
tests for DROID, Fido and the Unix File tool. DROID and Fido simply
leave such files unidentified, whereas the Unix File tool identifies
them as 'plain text' (which, of course, is correct at a lower level, but
not very helpful). Unfortunately, such files are pretty common in
practice.

## Using an XML parser to identify XML

A different approach to identify these files would be to run them
through an XML parser. If a parser can make sense of a file's contents
this means it is well-formed (but not necessarily valid!) XML. In all
other cases, it's something else.

I ended up writing some [Python](http://www.python.org/) code to see how
this would work in practice. I first created two re-usable Python
functions that check any given file for well-formedness using Python's
highly performant 'expat' parser (based on [original code by Farhad
Fouladi](http://code.activestate.com/recipes/52256-check-xml-well-formedness/)).
I then wrote a simple command-line application around it, which is
called "isXMLDemo.py". The demo can be used to analyse one file at a
time, or, alternatively, all files in a directory tree. The output is a
formatted text file that contains, for each analysed file, the
identification result (which is either "isXML" or "noXML").

I was surprised at how fast the XML parsing actually is. To give an
indication, I used "isXMLDemo.py" to analyse a 1.15 GB dataset that
contains 11,892 file objects. I ran this experiment under Microsoft
Windows XP Professional using a PC with a 3 Ghz GenuineIntel processor
and 1 GB RAM. The total time needed to analyse all files was about 90
seconds, which corresponds to an average throughput of about 131 files
per second.

## XML parsing in Fido?

Since the core functions that do the actual XML parsing are completely
reusable, it would probably be fairly easy to incorporate this kind of
identification into Fido. This would obviously have some impact on
Fido's performance, but not by very much. XML parsing could also be
offered as an option. In that case, the decision on whether to parse or
not to parse is up to the user.

An obvious limitation of this approach is that it will not identify XML
that is not well-formed. Also, it makes the line between identification
and validation somewhat blurry, but in practical terms that shouldn't be
a real problem. Finally, one could argue that knowing that a file
contains XML is not very informative at all, since it is merely a
container for something else.  This was the subject of an [earlier blog
post by Asger
Blekinge](http://www.openplanetsfoundation.org/blogs/2011-02-17-new-direction-file-characterisation).
However, even then, identifying the container is a necessary first step,
and one that the current tools don't seem to be too good at yet.

## Demo

For those who want to do some tests for themselves, I have <strike>attached the
demo script to this post</strike> [uploaded the demo script to GitHub](https://github.com/bitsgalore/isXMLDemo). The <strike>ZIP file</strike> repository contains the Python script with its  documentation in PDF format. If you end up with any interesting
results, or if you have any other thoughts on this: please report back
in the comments!

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2011/07/11/improved-identification-xml-python-experiment/)
