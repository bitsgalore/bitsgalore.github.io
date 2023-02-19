---
layout: post
title: EPUB for archival preservation&#58; an update
tags: [EPUB,EPUBCheck]
comment_id: 53
---

## Introduction

Last year (2012) the KB released a [report on the suitability of the *EPUB* format for archival preservation](https://zenodo.org/record/839711). A substantial number of *EPUB*-related developments have happened since then, and as a result some of the report's findings and conclusions have become outdated. This applies in particular to the observations on *EPUB* 3, and the support of *EPUB* by characterisation tools. This blog post provides an update to those findings. It addresses the following topics in particular:

* Use of *EPUB* in scholarly publishing
* Adoption and use of *EPUB* 3
* *EPUB* 3 reader support
* Support of *EPUB* by characterisation tools

In the following sections I will briefly summarise the main developments in each of these areas, after which I will wrap up things in a concluding section.

<!-- more -->

## Use of *EPUB* in scholarly publishing

Although scholarly publishing is still dominated by *PDF*, the use of *EPUB* in this sector is on the rise. [This blog post by Todd Carpenter](http://scholarlykitchen.sspnet.org/2013/03/19/is-it-time-for-scholarly-journal-publishers-to-begin-distributing-articles-using-epub-3/) gives the following examples:

* BioMed Central is one of the first publishers that started offering their scholarly publications in *EPUB* format (as explained in this [December 2012 blog post](http://blogs.biomedcentral.com/bmcblog/2012/12/11/biomed-central-now-publishes-in-epub-format/)). One of the journals that are available in *EPUB* format is the  [Journal of Neuroinflammation](http://www.jneuroinflammation.com/content). 
* [Hindawi Publishing Corporation recently added *EPUB*](http://www.hindawi.com/epub/) as one of the available formats for all of their journal and book publications. 
* Lippincott Williams & Wilkins (which is part of Kluwer) also offers [some of their titles](http://journals.lww.com/pages/results.aspx?txtKeywords=epub) in *EPUB* format. 

At the time of writing, the above publishers are all using *EPUB* 2.

## Adoption and use of *EPUB* 3

Over the last year a number of organisations that are representing the publishing industry have expressed their support of *EPUB* 3. 

The [Book Industry Study Group (BISG)](http://www.bisg.org) is a trade association for [companies in  the publishing industry](http://www.bisg.org/directory/). Last year (August 2012) BISG released a [policy statement](http://www.bisg.org/what-we-do-4-155-pol-1201-endorsement-of-epub-3.php) in which it endorsed "*EPUB 3 as the accepted and preferred standard for representing, packaging, and encoding structured and semantically enhanced Web content — including XHTML, CSS, SVG, images, and other resources — for distribution in a single-file format*".

Early this year (March 2013) the [International Publishers Association (IPA)](http://www.internationalpublishers.org/) issued a [press release](http://www.internationalpublishers.org/images/stories/PR/2013/epub3pr_final.pdf) that also endorsed *EPUB* 3 as a "*preferred standard format for representing HTML and other web content for distribution as single-file publications*". IPA represents over 60 national publishing organisations from more than 50 countries.

Finally, the [European Booksellers Federation](http://www.europeanbooksellers.eu/) recently released a [report on the interoperability of eBook Formats](http://www.europeanbooksellers.eu/positionpaper/interoperability-e-books-formats). Its authors did a comparison of the features and functionality provided by *EPUB* 3, Amazon's [KF8 (Kindle)](https://en.wikipedia.org/wiki/Amazon_Kindle#Proprietary_formats_.28AZW.2C_KF8.29) and Apple's e-book formats. They concluded that *EPUB* 3 "*clearly covers the superset of the expressive abilities of all the formats*", and that there is "*no technical or functional reason not to use and establish EPUB 3 as an/the interoperable (open) ebook format standard*". This all suggests that *EPUB* 3 is widely supported by the publishing industry.

Having said that, the actual use of *EPUB* 3 is still limited at this stage, even though some publishers have already started using the format. Earlier this year technical publisher O’Reilly started releasing [all their new eBook bundles in *EPUB* 3 format](http://toc.oreilly.com/2013/02/oreillys-journey-to-epub-3.html). The announcement mentions that their backlist will be updated as well. Interestingly, they decided to create "hybrid" *EPUB*s that are backward-compatible with *EPUB* 2. In November 2012 publisher Hachette also [announced the launch of their *EPUB 3* program](http://www.digitalbookworld.com/2012/hachette-launches-epub3-program-committed-to-the-format/).

## *EPUB* 3 reader support

At this time reader support for *EPUB* 3 is still limited, but there have been a number of significant developments since the second half of 2012:

* [Apple iBooks](http://www.apple.com/apps/ibooks/) started support of *EPUB* 3 during the course of 2012, which means that the format can be read on Apple mobile devices (e.g. *iPad*). 
* [Azardi Desktop](http://azardi.infogridpacific.com/) is a viewer that supports most features of *EPUB* 3. It is available for Windows, Mac and Linux. 
* Helicon is an ebook production house that [claimed to have developed the first reading application for Android that fully supports *EPUB* 3](http://www.digitalbookworld.com/2012/helicon-books-claims-first-e-reader-for-android-with-full-epub3-support/) in December 2012.
* [Readium](http://readium.org/) is an open-source set of libraries for viewing and creating *EPUB* 2 and *EPUB* 3 content. The project has already produced a [Google Chrome extension](https://chrome.google.com/webstore/detail/empty-title/fepbnnnkkadjhjahcafoaglimekefifl?hl=en) that allows you to view *EPUB* content in the browser. Since March 2013, Readium is backed by the [Readium Foundation](http://readium.org/readium-foundation-announced), which is a consortium of (currently 27) [companies](http://readium.org/membership) that are active in digital publishing. Current work focuses on the development of [Readium SDK](http://readium.org/projects/readium-sdk), a high-performance Software Development Kit that is optimised for mobile devices. 
* E-Reader manufacturer Kobo [announced](http://www.digitalbookworld.com/2012/kobo-to-fully-support-epub-3-by-third-quarter-2013/) that it aims to offer full support of *EPUB* 3 by the third quarter of 2013.
* According to [this piece](http://goodereader.com/blog/electronic-readers/the-conundrum-of-digital-publishing-html5-or-epub-3/), Barnes and Noble also said they will support *EPUB* 3 sometime in  2013.
* In February Sony [added support for *EPUB* 3 to its Reader app for Google Android](http://goodereader.com/blog/electronic-readers/sony-reader-for-android-updated-to-support-epub-3/). Since Sony is a member of the [Readium Foundation](http://readium.org/readium-foundation-announced), it is to be expected that *EPUB* 3 support for their e-readers will follow as well (although the company hasn't made any official statement on this).

## Support of *EPUB* by characterisation tools

The 2012 report concluded that *EPUB* was not optimally supported by characterisation tools. This situation has improved quite a lot since that time.

### Identification

*EPUB* is now [included in *PRONOM*](http://www.nationalarchives.gov.uk/PRONOM/Format/proFormatSearch.aspx?status=detailReport&id=1270), and has a corresponding [DROID](http://www.nationalarchives.gov.uk/information-management/projects-and-work/droid.htm) signature. This means that [*Fido*](http://fido.openpreservation.org/) should now be able to identify the format as well. On a side note, PRONOM doesn't differentiate between *EPUB* 2 and 3, and it appears that the current record (which is only an outline record anyway) either combines both versions, or only refers to *EPUB* 2. PRONOM should probably be more specific on this.

### Validation and feature extraction

The 2012 report included tests of 2 *EPUB* validator tools: [*epubcheck*](http://code.google.com/p/epubcheck/) and [flightcrew](http://code.google.com/p/flightcrew/). While testing *epubcheck* in 2012, I was't entirely happy with the rather unstructured output that the tool produced. Also, I couldn't find *any* tool that was capable of extracting technical meta-information about an *EPUB*, like the presence of encryption or other digital rights management technology (feature extraction). Happily, starting with version 3.0 *epubcheck* is capable of extracting this kind of information. Moreover, it added an option to [report its output in structured *XML* format](http://code.google.com/p/epubcheck/wiki/Extraction) that follows the [*JHOVE*](http://sourceforge.net/projects/jhove/) schema. I haven't done any elaborate testing, but a quick run on some of [these *EPUB* 3 samples](http://code.google.com/p/epub-samples/) showed that *epubcheck* was able to identify font obfuscation, in which case a property *hasEncryption* (value *true*) is reported. I wasn't able to find any *EPUB* files with *DRM*, so I cannot confirm if *epubcheck* detects this as well.

### Flightcrew

As for *flightcrew*, no new versions of that tool have been released since August 2011, and it looks like it is not under any active development.

## Discussion and conclusions

Since the release of the [KB report on the suitability of *EPUB* for archival preservation](https://zenodo.org/record/839711) the *EPUB* landscape has changed rather a lot. First, a number of academic publishers have started to offer scholarly content in this format. Although *EPUB* 3 is still in its early stages, various organisations representing the publishing industry have explicitly expressed their support of *EPUB* 3. A number of software applications now exist that are able to read the format, and work on a high-performance open source *EPUB* 3 Software Development Kit is backed by major players in the digital publishing industry (including e-reader manufacturers such as Kobo and Sony). *EPUB* support by characterisation tools has improved as well, mostly thanks to a number of recent enhancements of *epubcheck*. So, overall, *EPUB*'s credentials as a preservation format appear to have improved quite a bit over the last year. In the case of *EPUB* 3 it's still too early to say anything about actual adoption, but the conditions for adoption to happen look pretty favourable. This is something I will get back to in my next update, perhaps in another year from now.   

## Useful links

* [International Digital Publishing Forum ](http://idpf.org/)

* [EPUB 3 Sample Documents](http://code.google.com/p/epub-samples/)

* [Fixed Layout (FLO) Demonstration ePub3 Documents by Azardi](http://azardi.infogridpacific.com/resources.html)

* [epubcheck](http://code.google.com/p/epubcheck/)

* [Readium Foundation](http://readium.org/)

* [EPUB for archival preservation (2012 report)](https://zenodo.org/record/839711)

* [On the Interoperability of eBook Formats - report by the European Booksellers Federation](http://www.europeanbooksellers.eu/positionpaper/interoperability-e-books-formats)

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/05/23/epub-archival-preservation-update/)
