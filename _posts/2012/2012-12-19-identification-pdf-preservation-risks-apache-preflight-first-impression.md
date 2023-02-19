---
layout: post
title: Identification of PDF preservation risks with Apache Preflight&#58; a first impression
tags: [PDF,Apache-Preflight]
comment_id: 56
---

The *PDF* format contains various features that may make it difficult to
access content that is stored in this format in the long term. Examples
include (but are not limited to):

- Encryption features, which may either restrict some functionality
  (copying, printing) or make files inaccessible altogether.
- Multimedia features (embedded multimedia objects may be subject to
  format obsolescence)
- Reliance on external features (e.g. non-embedded fonts, or
  references to external documents)

<!-- more -->

A more exhaustive overview is given here:

[Adobe Portable Document Format - Inventory of long-term preservation risks](https://zenodo.org/record/801661)

and also here:

<https://web.archive.org/web/20130515073645/http://libraries.stackexchange.com/questions/964/what-preservation-risks-are-associated-with-the-pdf-file-format>

When *creating* a *PDF*, it is possible to minimise these risks by using
one of the *PDF/A* standards, which delineate a number of *PDF* feature
profiles that are unlikely to result in any long-term accessibility
problems. However, the simple fact is that most *PDF*s that are out
there are not *PDF/A*.

## PDF Profiling

For assessing risks in existing collections, it would be helpful to be
able to screen or profile *PDF*s for specific 'risky' features, such as
encryption or font embedding. Since *PDF/A* was specifically designed to
eliminate these 'risky' features, one would expect that *PDF/A*
validators (i.e. software tools that check the conformance of a *PDF*
file against the *PDF/A* specification) would be able to provide some
useful information on this.

In a first attempt to test whether this approach is feasible at all, I
did some tests with *Apache Preflight*, an open-source *PDF/A-1*
validator that is part of the [*Apache
PDFBox*](http://pdfbox.apache.org/) library.The specific
objectives of this work were:

- To get a first impression of the *Apache Preflight* (part of
  *PDFBox*) *PDF/A-1b* validator.
- To investigate if *Apache Preflight* is able to detect unwanted
  (from a preservation point of view) features in *PDF* files (i.e.
  *PDF*s that are not necessarily of the *PDF/A* sub-type) such as
  password protection, encryption and non-embedded fonts.
- To provide a comparison with the *Preflight* module of *Adobe
  Acrobat* 9.5.
- To decide if doing more work on *Apache Preflight* (more elaborate
  testing, possible involvement in its development) are worthwhile.

The results can be found in the report [Identification of preservation
risks in PDF with Apache Preflight: a first
impression](https://zenodo.org/record/2556637)

## The Archivist's PDF Cabinet of Horrors

The report's findings are to a large extent based on a suite of small,
simple test files that were created especialy for this work. Each file
contains one 'risky' feature, with focus on the following feature
classes:

- Encryption
- Multimedia
- Scripts
- Fonts
- File attachments
- External references
- Byte corruption

The dataset can be found here:

<http://www.opf-labs.org/format-corpus/pdfCabinetOfHorrors/>

## Link to full report

[Identification of preservation risks in PDF with Apache Preflight: a
first impression](https://zenodo.org/record/2556637)

## Update (January 2013)

Since the report was published, a number of improvements have been made
to Apache Preflight which should fix some of the reported issues. I
haven't tested the latest version yet, but will try doing this some time
soon.

## Update (March 2013)

During the SPRUCE [hackathon on unified
characterisation](http://wiki.opf-labs.org/display/SPR/SPRUCE+Hackathon+Leeds%2C+Unified+Characterisation)
(Leeds, 11-12 March, 2013) additional work was done on this. See this
[blog post by Pete
Cliff](https://openpreservation.org/blogs/2013-03-15-pdf-eh-another-hackathon-tale)
for more details. Importantly, the tests done during the hackathon
showed that *Apache Preflight*'s ability to identify 'risky' features
has improved significantly since I published my report back in December,
and the issues that are mentioned in the report appear to have been
largely resolved!

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression/)
