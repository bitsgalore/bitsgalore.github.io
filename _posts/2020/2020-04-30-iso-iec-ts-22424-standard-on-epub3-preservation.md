---
layout: post
title: ISO/IEC TS 22424 standard on EPUB3 preservation
tags: [EPUB, preservation-risks]
comment_id: 71
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2020/04/scream.jpg" alt="Scream">
  <figcaption><a href="https://commons.wikimedia.org/wiki/File:%27The_Scream%27,_undated_drawing_Edvard_Munch,_Bergen_Kunstmuseum.JPG">“The Scream”</a>, undated drawing by Edvard Munch, Bergen Kunstmuseum, Public domain.</figcaption>
</figure>

Earlier this week Library of Congress added [a new entry on the standard "Digital publishing — EPUB3 preservation" (ISO/IEC TS 22424)](https://www.loc.gov/preservation/digital/formats/fdd/fdd000519.shtml) to its excellent Digital Formats web site. This standard was developed by the [ISO Technical Committee on Document description and processing languages](https://www.iso.org/committee/45374.html), and was published in January this year (2020).

According to its authors, "the ISO/IEC TS 22424 series supports long-term preservation of EPUB publications via a dual strategy". The standard is made up of 2 parts, which are sold as separate documents on the ISO website:

1. [Part 1: Principles (ISO/IEC TS 22424-1:2020)](https://www.iso.org/standard/73163.html)

2. [Part 2: Metadata requirements ISO/IEC TS 22424-2:2020](https://www.iso.org/standard/73169.html)

In this blog post I will take a closer look at both parts of the standard. What do they purport, what is their scope, and to what degree do they live up to their stated promises? Readers who are only interested in the most important findings may want to jump to the "Summary and discussion" section at the end of this post.

<!-- more -->

As I didn't have access to the final as-published versions of the documents, this review is based on a combination of the (non-paywalled) preview documents on the ISO website, and a pre-publication "Final text" from September 2019. The latter may be marginally different from the published standard, but I don't expect these differences will affect the general conclusions of this review.

## Part 1 - Principles

Even though the full standard documents are behind a paywall, a [free preview of Part 1 is available here](https://www.iso.org/obp/ui/#iso:std:iso-iec:ts:22424:-1:ed-1:v1:en). It contains the introductory chapter, as well as the full table of contents.

### Purpose and scope

The introductory chapter clearly defines the purpose of this part of the standard:

> This document facilitates the long-term preservation of EPUB publications by specifying in general level EPUB features which are mandatory for long-term preservation (such as font embedding) and features which should be avoided if possible.
>
> This document can be seen as a stepping stone towards a detailed specification which would be related to EPUB in the same way as PDF/A, specified in ISO 19005-1 to ISO 19005-3, is related to the Portable Document Format (PDF).

This suggests that the purpose of Part 1 of the standard is make some first steps towards a "preservation-friendly" EPUB profile, analogous to the [PDF/A](https://en.wikipedia.org/wiki/PDF/A) profiles for the PDF format.

According to the authors, long-term preservation requires:

1. making the object such as \[sic\] EPUB publication fit for preservation, including features to be used and features to avoid;
2. packaging the object (and any metadata related to it) together with any additional data such as other versions of the object and other documentation into an Open Archival Information System (OAIS) submission information package (SIP).

They then write that "packaging is covered in ISO/IEC TS 22424-2" (Part 2 of the standard).

### Part 1 is actually about packaging (well, mostly)

Imagine my surprise then when I found the two main chapters of Part 1 to be titled "Packaging standards" (with a discussion of [OAIS Information Packages](https://wiki.dpconline.org/index.php?title=2.2.3_INFORMATION_PACKAGE_VARIANTS)), and "Construction of OAIS information packages". The latter chapter contains an exhaustive list of requirements and recommendations on how EPUB documents should be packaged in an OAIS Submission Information Package (SIP). So, on the surface it looks like the content that is supposed to be in Part 2 of the standard (packaging) somehow ended up in Part 1 instead!

However, a more in-depth look reveals that the generic packaging requirements (which are unrelated to EPUB) are mixed with requirements that actually *are* specific to EPUB. For example, section 6.2.1 ("EPUB publications SHALL be sent to a repository system as well‐formed and complete Submission Information Packages") lists requirements on validity against the EPUB specification, font embedding, multimedia content, remote resources and encryption. Confusingly, these are interspersed with other requirements on descriptive metadata and submission agreements (which are not EPUB-specific). Moreover, most of the packaging-related requirements and recommendations mentioned here are totally unrelated to EPUB, and would apply generically to any given format. This makes me wonder why these are part of this standard at all.

### Some requirements already covered by EPUB specification

In addition, some of the EPUB-specific requirements are already covered by the EPUB format specification. For example, section 6.4 ("Structure of information packages") lists requirements on the presence of a manifest file inside an EPUB, the use of the EPUB Open Container Format, the presence of a "container.xml" file, and metadata inside the EPUB package and navigation documents. Any EPUB file that is valid against the EPUB format specification (which is required as per section 6.2.1) already satisfies these requirements, so their inclusion here is unnecessary.

## Part 2 - Metadata requirements

For Part 2, there is again a [free preview](https://www.iso.org/obp/ui/#iso:std:73169:en) with an introductory chapter and the full table of contents.

### Purpose and scope

The introduction defines the purpose of this part of the standard as follows:

> This document facilitates the long-term preservation of EPUB publications by specifying metadata elements which are required or recommended for long-term preservation (such as identifiers) and the ways in which the EPUB publication and related metadata can be packaged. EPUB versions 3 and 3.0.1 are covered; if necessary, the EPUB version applicable is specified.

So, this suggests this is all about metadata and packaging. In the "Scope" section the authors also say this:

> This document makes EPUB compliant with current practices of Open Archival Information Systems (OAIS) archives and technical requirements of repository systems. The former tend to rely on OAIS in their operations; the latter prefer to ingest electronic documents only in containers conforming to standards such as METS (Metadata Encoding and Transmission Standard).

This statement is remarkable, since OAIS does not include any requirements about file formats, so the concept of "OAIS-compliant" file formats is meaningless!

### SIP-level metadata vs EPUB-level metadata

The main body of Part 2 contains requirements related to various types of metadata. For example, Chapter 6 covers "Packaging metadata", which the authors describe as follows:

> This chapter covers mainly metadata about the SIP (container) which is usually submitted using METS elements and attributes.

However, it turns out that some sections within this chapter are actually about metadata *within* EPUB files. Examples are section 6.6 ("Core media type resource identifiers"), which defines requirements on identifiers used in the EPUB manifest (which is part of the EPUB file), and section 6.7 ("Foreign resource identifiers").

Like Part 1, some of the EPUB-specific requirements that are listed here are already covered by the EPUB specification. For example, section 7.2 ("Technical metadata") states:

> EPUB version used SHALL be specified in the package element of the EPUB publication’s content.opf file.

Since 'version' is a mandatory attribute of an EPUB 3 Package Document, the inclusion of this requirement is unnecessary. Similarly, section 7.4 ("Structural metadata") states:

> EPUB Open Container Format (OCF) SHALL be used to describe the structure of \[sic\] EPUB publication.

Again, this is true for *any* valid EPUB file, so why is this included in this standard?

### OAIS packages (again!)

The final two chapters of Part 2 are about the structure and content of OAIS Submission Information Packages. But OAIS information packages are already discussed at length in Part 1 (even though that discussion probably shouldn't be there to begin with!), so why is this subject revisited here?

## Summary and discussion

Since this blog post turned out a lot lengthier than I originally intended, this section summarizes and then discusses the most important findings. Overall, I was really surprised to find so many serious issues in a standard that has been authored over a 3-year period by an ISO-coordinated committee, especially given that most of these issues are obvious from even a cursory inspection of the standards documents!

### Lack of structure

Based on the abstracts on the ISO web site and the introductory sections in the standards documents I expected the following:

- Part 1: requirements and recommendations that are a first step in the direction of a preservation-friendly EPUB profile, analogous to PDF/A for the PDF format.
- Part 2: requirements and recommendations on the packaging of publications in EPUB format into OAIS Submission Information Packages (SIPs), along with accompanying metadata.

Instead, Part 1 of the standard is mostly about packaging, but elements of a "preservation-friendly" EPUB profile are buried within sections with package requirements and recommendations. On the other hand, Part 2 (which is supposed to be about packaging and metadata) also contains additional EPUB-specific requirements that one would expect to be part of the EPUB profile (that is, Part 1). Moreover, some of the EPUB-specific requirements are already covered by the EPUB format specification, which makes their inclusion in ISO/IEC TS 22424 unnecessary.

The overall effect of this is that elements of the "dual strategy" that underlies ISO/IEC TS 22424 are now scattered across both parts of the standard in a seemingly arbitrary manner. This makes using and navigating the standard unnecessarily difficult. Since both parts of the standard are only available upon payment of CHF 118 and CHF 138, respectively, it also means that buyers of any of these documents will receive a product that is substantially different from its description on the ISO web site!

### Path towards a preservation-friendly EPUB profile

The introduction of Part 1 of the standard mentions that the EPUB-specific requirements here are only a "stepping stone" towards a more detailed specification (analogous to PDF/A), that could be developed at a later stage by the EPUB community. Even though the authors should be commended for their modesty in this regard, it does make me wonder if such an early-stages effort merits an ISO-published standard. I'm also not convinced of the effectiveness of a "stepping stone" document that is hidden behind a paywall. I do think a preservation-friendly EPUB profile would be useful, and I also think that various people from the digital libraries and archives sector would be willing to contribute to it. But only if this is done in an open way, and not by some arcane ISO-guided process that first keeps the progress hidden from its potential users for 3 years, and then buries the results behind a paywall (which appears to have been the case for the current standard).

### Should packaging even be part of this standard?

Although I've only given the packaging and package-level metadata requirements and recommendations a cursory look, on the surface many of them are generally applicable to any digital content, and are not specific to publications in EPUB format. At the same time, package-level requirements are typically strongly tied to institutional policies. For instance, Part 2 of the standard requires that the SIP-level METS and PREMIS metadata describe all resources that are inside an EPUB container, which seems a bit excessive to me. This all makes me wonder if OAIS-packaging should be part of this standard at all.

### No EPUB 3.2!

It is also worth noting that the current standard only covers EPUB 3 and EPUB 3.0.1 (which was published in 2014). It does not cover the current EPUB 3.2, but it does mention that this version will be covered in the next edition of the standard[^1]. This is surprising, given that EPUB 3.2 was already published in May 2019. It also means that the standard was already outdated at its date of publication.

## Additional resources

- [Digital publishing — EPUB3 preservation — Part 1: Principles](https://www.iso.org/standard/73163.html)

- [Digital publishing — EPUB3 preservation — Part 2: Metadata requirements](https://www.iso.org/standard/73169.html)

- [EPUB (Electronic publication) Version 3 Preservation. ISO/IEC TS 22424:2020 (Library of Congress Sustainability of Digital Formats)](https://www.loc.gov/preservation/digital/formats/fdd/fdd000519.shtml)

[^1]: EPUB 3.1 is not covered either, but this is understandable, since this version was never supported by any reader or validation software.