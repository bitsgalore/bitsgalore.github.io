---
layout: post
title: When (not) to migrate a PDF to PDF/A
tags: [PDF]
comment_id: 44
---

It is well-known that PDF documents can contain features that are preservation risks (e.g. see [here](https://web.archive.org/web/20130515073645/http://libraries.stackexchange.com/questions/964/what-preservation-risks-are-associated-with-the-pdf-file-format) and [here](http://wiki.opf-labs.org/display/TR/Portable+Document+Format)). Migration of existing *PDF*s to *PDF/A* is sometimes advocated as a strategy for mitigating these risks. However, the benefits of this approach are often questionable, and the migration process can also be quite risky in itself. As I often get questions on this subject, I thought it might be worthwhile to do a short write-up on this.

<!-- more -->

## *PDF/A* is a profile

First, it's important to stress that each of the *PDF/A* standards (*A-1*, *A-2* and *A-3*) are really just *profiles* within the *PDF* format. More specifically, *PDF/A-1* offers a subset of [*PDF 1.4*](http://acroeng.adobe.com/PDFReference/PDF_1.4/PDF%20Reference%201.4.pdf), whereas *PDF/A-2* and *PDF/A-3* are based on [the ISO 32000 version of *PDF 1.7*](http://acroeng.adobe.com/PDFReference/ISO32000/PDF32000-Adobe.pdf). What  these profiles have in common, is that they prohibit some features (e.g. multimedia, encryption, interactive content) that are allowed in 'regular' *PDF*. Also, they narrow down the way other features are implemented, for example by requiring that all fonts are embedded in the document. This can be illustrated with the following simple Venn diagram below, which shows the feature sets of the aforementioned *PDF* flavours:

![PDF Venn diagram]({{ BASE_PATH }}/images/2014/08/pdfVenn.png)

Here we see how *PDF/A-1* is a subset of *PDF 1.4*, which in turn is a subset of *PDF 1.7*. *PDF A/2* and *PDF A/3* (aggregated here as one entity for the sake of readability) are subsets of *PDF 1.7*, and include all the features of *PDF A/1*.  

Keeping this in mind, it's easy to see that migrating an arbitrary *PDF* to *PDF/A* can result in problems.

## Loss, alteration during migration

Suppose, as an example, that we have a *PDF* that contains a movie. This is prohibited in *PDF/A*, so migrating to *PDF/A* will simply result in the loss of the multimedia content. Another example are fonts: all fonts in a *PDF/A* document must be embedded. But what happens if the source *PDF* uses non-embedded fonts that are not available on the machine on which the migration is run? Will the  migration tool exit with a warning, or will it silently use some alternative, perhaps similar font? And how do you check for this?

## Complexity and effect of errors

Also, migrations like these typically involve a complete re-processing of the *PDF*'s internal structure. The format's complexity implies that there's a lot of potential for things to go wrong in this process. This is particularly true if the source *PDF* contains subtle errors, in which case the risk of losing information is very real (even though the original document may be perfectly readable in a viewer). Since we don't really have any tools for detecting such errors (i.e. a [sufficiently reliable *PDF* validator](http://duff-johnson.com/wp-content/uploads/2014/01/PDFValidationDreamOrYawn.pdf)), these cases can be difficult to deal with. Some further considerations can be found [here](http://web.archive.org/web/20130605142355/http://libraries.stackexchange.com/questions/1117/converting-invalid-pdfs-or-not-for-digital-preservation) (the context there is slightly different, but the risks are similar).

## Digitised vs born-digital

The origin of the source *PDF*s may be another thing to take into account. If *PDF*s were originally created as part of a digitisation project (e.g. scanned books), the *PDF* is usually little more than a wrapper around a bunch of images, perhaps augmented by an OCR layer. Migrating such *PDF*s to *PDF/A* is pretty straightforward, since the source files are unlikely to contain any features that are not allowed in *PDF/A*. At the same time, this also means that the benefits of migrating such files to *PDF/A* are pretty limited, since the source *PDF*s weren't problematic to begin with!

The potential benefits *PDF/A* may be more obvious for a lot of born-digital content; however, for the reasons listed in the previous section, the migration is more complex, and there's just a lot more that can go wrong (see also [here](http://qanda.digipres.org/19/what-are-the-benefits-and-risks-of-using-the-pdf-a-file-format?show=21#a21) for some additional considerations).

## Conclusions

Although migrating *PDF* documents to *PDF/A* may look superficially attractive, it is actually quite risky in practice, and it may easily result in unintentional data loss. Moreover, the risks increase with the number of preservation-unfriendly features, meaning that the migration is most likely to be successful for source *PDF*s that weren't problematic to begin with, which belies the very purpose of migrating to *PDF/A*. For specific cases, migration to *PDF/A* may still be a sensible approach, but the expected benefits should be weighed carefully against the risks. In the absence of stable, generally accepted tools for assessing the quality of *PDF*s (both source *and* destination!), it would also seem prudent to always keep the originals.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2014/08/27/when-not-migrate-pdf-pdfa/)
