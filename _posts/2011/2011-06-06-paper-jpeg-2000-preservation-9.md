---
layout: post
title: Paper on JPEG 2000 for preservation
tags: [jpeg-2000,JP2]
comment_id: 67
---

The JPEG 2000 compression standard is steadily becoming more and more
popular in the archival community. Several large (national) libraries
are now using the [JP2
format](http://www.jpeg.org/public/15444-1annexi.pdf) (which
corresponds to Part 1 of the standard) as the master format in mass
digitisation projects. However, some aspects of the JP2 file format are
defined in ways that are open to multiple interpretations. This applies
to the embedding of [ICC
profiles](http://en.wikipedia.org/wiki/ICC_profile) (which
are used to define colour space information), and the definition of grid
resolution. This situation has lead to a number of interoperability
issues that are potential risks for long-term preservation.

<!-- more -->

## Paper

I recently addressed this in a
[paper](http://www.dlib.org/dlib/may11/vanderknijff/05vanderknijff.html)
that has just been published in [D-Lib
Magazine](http://www.dlib.org/). An earlier version of the
paper was used as a ‘defect report’ by the [JPEG
committee](http://www.jpeg.org/). The paper gives a detailed
description of the problems, and shows to what extent the most
widely-used JPEG 2000 encoders are affected by these issues.

## Solutions

The paper also suggests some possible solutions. Importantly, none of
the found problems require any changes to the actual file format;
rather, some features should simply be defined slightly differently. In
the case of the ICC profile issue this boils down to allowing a widely
used class of ICC profiles that are currently prohibited in JPEG 2000.
The resolution issue could be fixed by a more specific definition of the
existing resolution fields.

## Amendment to the standard

Both issues will be addressed in an amendment to the standard. Rob
Buckley provides more details on this (along with some interesting
background information on colour space support in JP2) in a recent [blog
entry on the Wellcome Library’s JPEG 2000
blog](http://jpeg2000wellcomelibrary.blogspot.com/2011/04/guest-post-color-in-jp2.html).
As Rob puts it:

>The final outcome of all this will be a JP2 file format standard that
>aligns with current practice; supports RGB spaces such as Adobe RGB
>1998, ProPhoto RGB and eci RGB v2; and provides a smooth migration path
>from TIFF masters as JP2 increasingly becomes used as an image
>preservation format.

So, some relatively small adjustments to the standard could result in a
significant improvement of the suitability of JP2 for preservation
purposes.

Since various institutions are using JPEG 2000 *now*, the paper also
provides some practical recommendations that may help in mitigating the
risks for existing collections.

## Link to paper
 
[JPEG 2000 for Long-term Preservation: JP2 as a Preservation
Format](http://www.dlib.org/dlib/may11/vanderknijff/05vanderknijff.html)


<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2011/06/06/paper-jpeg-2000-preservation-9/)
