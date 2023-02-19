---
layout: post
title: Update on jpylyzer
tags: [jpeg-2000,JP2,jpylyzer,Debian,packaging]
comment_id: 61
---

## Introduction

In this blog post I will give a brief update of the latest [*jpylyzer*] [jpylyzerOPF] developments. *Jpylyzer* is a validation and feature extraction tool for the [JP2 (JPEG 2000 Part 1)][jp2Spec] still image format.

<!-- more -->

## History of jpylyzer

Around mid-summer 2011, the [KB][KB] started initial preparations for migrating 146 TB of TIFF images from the Dutch [Metamorfoze][Metamorfoze] program to JP2. We realised that the possibility of hardware failure (e.g. short network interruptions) during the migration process would imply a major risk for the creation of malformed and damaged files. Around the same time, we received some rather worrying reports from the [British Library][BL], who were confronted with JP2 images that contained damage that couldn't be detected with existing tools such as [JHOVE][JHOVE]. 

This prompted me to have a go at  writing a rudimentary software tool that was able to detect some simple forms of file corruption in JP2. A [blog post][jp2StructureChecker] I wrote on this resulted in quite a bit of feedback, and several people asked about the possibility to extend the tool's functionality to a full-fledged JP2 validator and feature extractor. Since this fitted in nicely with some [SCAPE][SCAPE] work that was envisaged on quality assurance in imaging workflows, I started work on a [first prototype of *jpylyzer*][jpylyzerFirstBlog], which saw the light of day in December.

In the remainder of this blog post I will outline the main developments that have happened since then.

## Refactoring of existing code

Shortly after the release of the first prototype, my KB colleague René van der Ark spontaneously offered to do a refactoring job on my original code, which was clumsy and unnecessarily lengthy in places. This has resulted in a code that is more modular, and which adheres more closely to established programming practices. As a result, the refactored code is significantly more maintainable than the original one, which makes it easier for other programmers to contribute to *jpylyzer*. This should also contribute to the long-term sustainability of the software.

## New features

Since the first prototype the following functionality was added to *jpylyzer*:
  
- New validator functions were added for *XML* boxes, *UUID* boxes, *UUID Info* boxes, *Palette* boxes and *Component Mapping* boxes.
- A check was added that verifies whether the number of tiles in an image matches the image- and tile size information in the codestream header.
- Another check was added that verifies whether all tile-parts within each tile exist.
- The ICC profile feature extraction function has been given an overhaul, and it now extracts all ICC header items. In addition, the output is now reported in a more user-friendly format.
- For codestream comments, all characters from the Latin character set are now supported.
- The reporting of the validation results has been made more concise. By default, *jpylyzer* now only reports the results of validation tests that *failed* (previously *all* test results were reported). This behaviour can be overruled with a new *--verbose* switch.

In addition to the above, various bugs and minor issues have been addressed as well.

## Debian packages

During the SCAPE Braga meeting in February, work started on the creation of [Debian packages][DEB] for *jpylyzer*. The availability of Debian packages greatly simplifies *jpylyzer*'s installation on Linux-based systems. [This work][DebBraga] was done by Dave Tarrant [(University of Southampton)][SOTON], Miguel Ferreira, Rui Castro, Hélder Silva [(KEEP Solutions)][KEEPS] and Rainer Schmidt [(AIT)][AIT]. 

## Jpylyzer now hosted by OPF

In order to make a tool sustainable, it is important that its maintenance and development are not solely dependent on one single institution or person. Because of this, *jpylyzer* is now hosted by the Open Planets Foundation, which ensures the involvement of a wider community. *Jpylyzer* also has [its own home page on the OPF site] [jpylyzerOPF]. It contains links to the source code, Windows executables, Debian packages and the User Manual.

## Jpylyzer home page

<http://jpylyzer.openpreservation.org/>

[jpylyzerOPF]: http://jpylyzer.openpreservation.org/
[jpylyzerGit]:https://github.com/openpreserve/jpylyzer
[userManual]:http://jpylyzer.openpreservation.org//userManual.html
[jp2Spec]: http://www.jpeg.org/public/15444-1annexi.pdf
[jpylyzerFirstBlog]: {{ BASE_PATH }}/2011/12/14/prototype-jp2-validator-and-properties-extractor
[jp2StructureChecker]: {{ BASE_PATH }}/2011/09/01/simple-jp2-file-structure-checker
[BL]: http://www.bl.uk/
[KB]: http://www.kb.nl/index-en.html
[JHOVE]:http://hul.harvard.edu/jhove/
[Metamorfoze]:http://www.metamorfoze.nl/english/home
[SCAPE]: http://www.scape-project.eu/
[KEEPS]: http://www.keep.pt/en
[DEB]: http://en.wikipedia.org/wiki/Deb_%28file_format%29
[SOTON]:http://www.southampton.ac.uk/
[AIT]:http://www.ait.ac.at/
[DebBraga]: http://www.openplanetsfoundation.org/blogs/2012-02-15-sustainability-and-adoption-preservation-tools

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2012/04/23/update-jpylyzer/)
