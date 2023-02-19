---
layout: post
title: ICC profiles and resolution in JP2&#58; update on 2011 D-Lib paper
tags: [jpeg-2000,JP2]
comment_id: 52
---

It's been more than two years now since I wrote my D-Lib paper [*JPEG 2000 for Long-term Preservation: JP2 as a Preservation Format*](http://www.dlib.org/dlib/may11/vanderknijff/05vanderknijff.html). From time to time people ask me about the status of the issues that are mentioned in that paper, so here's a long overdue update.

<!-- more -->

## Issues addressed in the 2011 paper

The [D-Lib paper](http://www.dlib.org/dlib/may11/vanderknijff/05vanderknijff.html) mainly focused on two problems with the (then-current version of the) [JP2 format specification](http://www.jpeg.org/public/15444-1annexi.pdf):

1. The specification was overly restrictive on the embedding of ICC profiles. By only allowing *input* profiles, this ruled out the use of *display* profiles. In practice this meant that widely-used working colour spaces such as [*Adobe RGB*](http://www.adobe.com/digitalimag/pdfs/AdobeRGB1998.pdf) and [*eciRGB 2*](http://www.eci.org/doku.php?id=en:colourstandards:workingcolorspaces) could not be used in *JP2* without violating the standard.
2. *JP2* makes a distinction between *capture* resolution and *default display* resolution, which are stored in two designated sets of header fields. However, the specification was not clear in which case either set of fields should be used.

This lead to the situation that not all software products were interpreting the specification in the same way. For instance, some encoders would (silently) produce files in [*JPX*](http://fileformats.archiveteam.org/wiki/JPX) format whenever they encountered an input image with an embedded  *display* ICC profile. Other encoders would embed the profile, changing the profile class in the process, whereas others yet would ignore the limitation altogether and embed the profile without complaining. Similarly, some software products would only write (and read) the *capture* resolution fields (while ignoring any *default display* ones), whereas the opposite was true for other products. This in turn raised various interoperability issues, many of which are potential risks in a long-term preservation context. 

The 2011 paper concluded that:

>[t]hese issues could be remedied by some small adjustments of JP2's format specification, which would create minimal backward compatibility problems, if any at all.

## Amendment to the standard

So, enter the [amendment "*Updated ICC profile support and resolution clarification*"](http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=59863), which was published by ISO earlier this year. This amendment remedies the above issues by applying the following changes to the [existing JP2 format specification](http://www.jpeg.org/public/15444-1annexi.pdf):

1. The *Restricted ICC profile* method now permits the use of *display* profiles (previously only *input* profiles were allowed). The other restrictions (e.g. that ICC profiles should be of either the Monochrome or the Three-Component Matrix-Based type) remain unchanged.
2. It is more specific about the intended uses of the *capture* and the *default display* resolution boxes. Of particular interest here is that *capture* resolution now reflects the resolution at which the image samples were "captured or created". Previously the word "digitized" was used, which ruled out the case of born-digital materials. The use of the *default display* resolution box is also further clarified. In practice this means that the *capture* resolution is pretty much equivalent to the *XResolution* /*YResolution* fields in TIFF.  

Note that the full amendment text is only available after purchase at ISO (previously an earlier draft was available for free, but apparently it was taken down recently).

## Implementation of changes

Since a standard isn't worth much unless it is used, let's have a quick look at the three most popular JPEG 2000 implementations (a more elaborate overview is available [here](http://wiki.opf-labs.org/display/TR/Handling+of+ICC+profiles) and [here](http://wiki.opf-labs.org/display/TR/Resolution+not+in+expected+header+fields) at the [OPF File Format Risk Registry](http://wiki.opf-labs.org/display/TR/OPF+File+Format+Risk+Registry)).

Recent versions of **Kakadu**'s *kdu_compress* are now able to correctly handle *ICC* profiles. Compressing a *TIFF* that contains an *ICC* profile that meets the (updated) *restricted ICC profile* definition now produces a *JP2* with the profile correctly embedded. Moreover, *kdu_compress* now uses the *capture* resolution fields to store the image's resolution (as derived from the *TIFF*). Previously, the *Kakadu* demo applications were using the *default display* fields instead, which resulted in various interoperability issues, because most other decoders/encoders were using the *capture* fields. This is all solved in the latest version.

By default **Aware** and **Luratech** already used the *capture* resolution fields back in 2011, and this behaviour is now consistent with the updated standard. As for *ICC* profiles, *Aware* accepted *display* profiles without complaining in my 2011 tests, and with the amendment in effect, these images are now also valid *JP2*. *Luratech* used to handle *display* profiles by changing the profile class field tom *input*. That was in 2011, and I don't know if anything has changed since then, but then again this behaviour never caused by problems in the first place. 

## Round-up and conclusions

The amendment to JP2 fixes the previous shortcomings that were mentioned in my 2011 D-Lib paper. Moreover, the behaviour of the three most popular (commercial) JPEG 2000 implementations now closely follows the updated specification, which should minimise any interoperability problems related to ICC profiles and resolution.

## Links

[JPEG 2000 for Long-term Preservation: JP2 as a Preservation Format (D-Lib)](http://www.dlib.org/dlib/may11/vanderknijff/05vanderknijff.html)

[JP2 format specification (2004 version)](http://www.jpeg.org/public/15444-1annexi.pdf)

[ISO/IEC 15444-1:2004/Amd 6:2013. Updated ICC profile support and resolution clarification](http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=59863)

[Handling of ICC Profiles by JPEG 2000 encoders (OPF File Format Risk Registry)](http://wiki.opf-labs.org/display/TR/Handling+of+ICC+profiles)

[Handling of resolution fields by JPEG 2000 encoders (OPF File Format Risk Registry)](http://wiki.opf-labs.org/display/TR/Resolution+not+in+expected+header+fields)

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/07/01/icc-profiles-and-resolution-jp2-update-2011-d-lib-paper/)
