---
layout: post
title: The future of EPUB? A first look at the EPUB 3.1 Editor’s draft
tags: [EPUB]
comment_id: 29
---

About a month ago the [International Digital Publishing Forum](http://idpf.org/), the standards body behind the *EPUB* format, published an 
[Editor's Draft of *EPUB* 3.1](http://www.idpf.org/epub/31/spec/epub-spec.html). This is meant to be the successor of the [current 3.0.1 version](http://idpf.org/epub/301). 
IDPC has set up a [community review](http://idpf.org/news/first-editors-draft-of-epub-31-available-for-review), which allows interested parties to comment on the draft. The proposed changes relative to *EPUB* 3.0.1 are summarised [in this document](http://www.idpf.org/epub/31/spec/epub-changes.html). A note at the top states (emphasis added by me):

> The EPUB working group has opted for a *radical change approach* to the addition and deletion of features in the 3.1 revision to *move the standard aggressively forward* with the overarching goals of alignment with the Open Web Platform and simplification of the core specifications. 

As Gary McGath [pointed out earlier](https://fileformats.wordpress.com/2016/02/05/epub-3-1/), this is a pretty bold statement for what is essentially a minor version. The authors of the draft also mention that they expect it "will provoke strong reactions both for and against", and that changes that raise "strong negative reactions" from the community "will be reviewed for future drafts".

This blog post is an attempt to identify the main implications of the current draft for libraries and archives: to what degree would the proposed changes affect (long-term) accessibility? Since the current draft is particularly notable for its aggressive *removal* of various existing *EPUB* features, I will focus on these. These observations are all based on the 30 January 2016 draft of the changes document.

<!-- more -->

## [Removed support for EPUBCFI for linking](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-epub31-cfi)

The *EPUB* Canonical Fragment Identifier ([EPUBCFI](http://www.idpf.org/epub/linking/cfi/epub-cfi.html)) "defines a standardized method for referencing arbitrary content within an EPUB Publication". Until *EPUB* 3.0.1, Reading Systems were required to support EPUBCFI for hyperlinking within and between documents. This requirement is dropped in *EPUB* 3.1 (although it would still be possible to use EPUBCFI for annotations and bookmarks).

In principle this change could result in problems if an *EPUB* that uses CFI for hyperlinks is opened in a 3.1 reading system: in that case the hyperlinks would not work. However, according to *EPUB* editor Matt Garrish, [authors simply do not use CFI for hyperlinking](https://github.com/IDPF/epub-revision/issues/662#issuecomment-193793093). He also mentions a check by Google on their corpus of millions of books, which only turned up a few instances of CFI use. One of these was a link in an *EPUB* best practices book, while the remaining ones were all part of the *EPUB* test suite documents. If these results are representative of all *EPUB*s "in the wild", the implications of the change would be negligible.

## [Reduced set of metadata elements in Package Document](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-pkg-metadata)

*EPUB* 3.1 imposes restrictions on the metadata elements that can be embedded in the Package Document. Up to version 3.0.1, the full [Dublin Core Metadata Element Set](http://dublincore.org/documents/dces/) was supported, whereas in 3.1 only the `dc:identifier`, `dc:title`, `dc:language`, `dc:creator`, `dc:publisher` and `dc:type` elements are allowed. Additional metadata *can* be included, but they need to be defined in a separate resource (file), which is referenced from the `metadata` element using the [`link`](http://www.idpf.org/epub/31/spec/epub-packages.html#sec-link-elem) element. Below is an example that uses a [MARC](https://en.wikipedia.org/wiki/MARC_standards) file:

```xml
<link rel="record"
href="meta/9780000000001.xml" 
media-type="application/marc"/>
```

Complicating things further, the [EPUB 3.1 Packages draft](http://www.idpf.org/epub/31/spec/epub-packages.html#sec-link-elem) says:

>Linked resources that are not Publication Resources are not subject to Core Media Type requirements [EPUB31] and may be located inside or outside [EPUB31] the EPUB Container. Retrieval of Remote Resources is optional.

So, linked metadata resources can have *any* possible format, and they may not even be included in the *EPUB* container. Even though these changes would have no direct consequences for long-term accessibility, they would seriously complicate document processing (e.g. ingest) workflows that rely on the metadata in the Package Document. It would also affect end users who rely on these metadata fields to [sort and find their ebooks](https://github.com/IDPF/epub-revision/issues/642#issuecomment-181450515)[^5].

## [Removal of the NCX](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-pkg-ncx)

*EPUB* 2 documents contain the [*NCX*](http://www.idpf.org/epub/20/spec/OPF_2.0.1_draft.htm#Section2.4.1) file ("Navigation Control file for XML"), which provides a mechanism to navigate a publication. It is essentially a hierarchical table of contents. The *NCX* was [superseded](http://www.idpf.org/epub/301/spec/epub-publications.html#ncx-superseded) by the [*Navigation Document*](http://www.idpf.org/epub/301/spec/epub-contentdocs.html#sec-xhtml-nav) in *EPUB* 3.0.1. However, the *NCX* was allowed in *EPUB* 3.01 publications, which was useful for keeping *EPUB* 3 publications compatible with older (*EPUB* 2-based) reading systems[^2].  The 3.1 draft forbids the *NCX* altogether, which means that such "hybrid" *EPUB*s are not possible without breaking the specification.

The main consequence of this is that it would make *EPUB* 3.1 files incompatible with older reading systems. More specifically, basic navigation functionality such as direct access to a chapter from the table of contents would not work.

To get an approximate idea of the impact of this, I had a look at the [*EPUB* 3 support grid](http://epubtest.org/), which gives detailed information about the support of specific *EPUB* 3 features for commonly used devices, apps, and reading systems. [This link](http://epubtest.org/testsuite/epub3/feature/toc-nav/) shows support of the [`toc nav`](http://www.idpf.org/epub/301/spec/epub-contentdocs.html#sec-xhtml-nav-def-types-toc) element, which defines the primary navigational hierarchy in the Navigation Document. Only 55% (34 out of 62) of all tested reading systems fully support the `toc nav` element, with 37% (23  out of 62) not supporting it at all[^3]. This may not be a big deal for users of software-based reading systems (which make up the majority of the support grid), but users of (older) E-ink readers often don't have the option to upgrade their devices. A good example is this (now discontinued) [Sony e-Ink hardware reader](http://epubtest.org/evaluation/52/). Unfortunately, E-ink devices appear to be underrepresented in the support grid. For example, it contains no information whatsoever on any of the popular Kobo readers.

The proposal to remove the *NCX* provoked strong reactions in the [community review](https://github.com/IDPF/epub-revision/issues/633), with one respondent stating it would lead to ["dropping support for millions of eInk reading systems"](https://github.com/IDPF/epub-revision/issues/633#issuecomment-170398642). It would also contradict [this statement from the *EPUB* 3.0.1 specification](http://www.idpf.org/epub/301/spec/epub-publications.html#ncx-superseded) (emphasis added by me):

> The NCX feature defined in [OPF2] is superseded by the EPUB Navigation Document [ContentDocs301]. *EPUB 3 Publications* may include an NCX (as defined in OPF 2.0.1) for EPUB 2 Reading System forwards compatibility purposes, but EPUB 3 Reading Systems must ignore the NCX.

The explicit reference to *EPUB 3 Publications* (**not** *EPUB 3.0.1 Publications*!!) implies that the statement applies to *EPUB* 3 in general. Removing the *NCX* in another *EPUB* 3 release would be at odds with this.

## [Removal of the guide Element](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-pkg-guide)

The [*guide* element](http://www.idpf.org/epub/20/spec/OPF_2.0.1_draft.htm#Section2.6) was an optional data structure in *EPUB* 2 that provided "convenient access" to structural components of a publication. It was [deprecated](http://www.idpf.org/epub/301/spec/epub-publications.html#sec-guide-elem) in *EPUB* 3.0.1. Without any data on the actual usage of this feature, it is difficult to say much about the impact of its complete removal (this was also [pointed out by one respondent to the community review](https://github.com/IDPF/epub-revision/issues/644#issuecomment-191706305)).

## [Removal of the bindings Element](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-pkg-bindings)

In *EPUB* 3.0.1 the [*bindings*](http://www.idpf.org/epub/301/spec/epub-publications.html#sec-bindings-elem) element  could be used to define fallbacks for [foreign resources](http://www.idpf.org/epub/301/spec/epub-publications.html#gloss-publication-resource-foreign). [According to *EPUB* editor Matt Garrish](https://github.com/IDPF/epub-revision/issues/639) "this feature is not widely used or supported", and the impact on accessibility appears to be negligible.

## [Removal of the switch Element](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-cdoc-switch)

The [*switch*](http://www.idpf.org/epub/301/spec/epub-contentdocs.html#sec-xhtml-content-switch) element in *EPUB* 3.0.1 allows one to define alternative representations of XML fragments. Here's an example:

```xml
<epub:switch id="cmlSwitch">
   
   <epub:case required-namespace="http://www.xml-cml.org/schema">
      <cml xmlns="http://www.xml-cml.org/schema">
         <molecule id="sulfuric-acid">
            <formula id="f1" concise="H 2 S 1 O 4"/>
         </molecule>
      </cml>
   </epub:case>
   
   <epub:default>
      <p>H<sub>2</sub>SO<sub>4</sub></p>
   </epub:default>
   
</epub:switch>
```

Here, we have a chemical formula in [*ChemML*](https://en.wikipedia.org/wiki/Chemical_Markup_Language) format and in standard *HTML*. *ChemML* is not natively supported in *EPUB*, so by default a reader will display the *HTML* version. However, wrapping both in a *switch* element would allow a *ChemML*-capable reader to render that representation instead.

I asked *EPUB* editor Matt Garrish how an *EPUB* 3.1-compliant reader would render content that is wrapped in a *switch* element. He [replied](https://github.com/IDPF/epub-revision/issues/637#issuecomment-193894732) that by default *all* of the switch content would be rendered. So for the example above, a reader would try to render both the *HTML* and the *ChemML* versions (with the latter failing on most reading systems). Matt stressed the significance of the *switch* element, adding that people have been using it, "if not extensively". 

## [Removal of the trigger Element](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-cdoc-trigger)

The [*trigger*](http://www.idpf.org/epub/301/spec/epub-contentdocs.html#sec-xhtml-epub-trigger) element in *EPUB* 3.0.1 is used to define simple user interfaces for multimedia content. Since this can be done natively in *HTML* 5, it is dropped from *EPUB* 3.1. [Here](https://github.com/IDPF/epub-revision/issues/638#issue-125825057) editor Matt Garrish explains that the feature is both "sparsely used" (referring to a survey of publishers) and "poorly supported".

## Miscellaneous changes

Apart from the changes above (which all *remove* features from the existing specification), the *EPUB* 3.1 draft also adds a number of new features, and clarifies some existing ones. I won't go over them in detail, but here's a brief overview:

* [New Core Media Types](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-epub31-cmt) - adds support for [*WOFF* 2.0](https://www.w3.org/TR/WOFF2/) and [*SFNT*](https://en.wikipedia.org/wiki/SFNT) fonts.

* [Addition of Support for HTML Syntax of HTML5](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-cdoc-html) - adds support for the *HTML* Syntax of *HTML5* (currently only the *XHTML* syntax is supported[^4]).

* [Replacement of EPUB Style Sheets with CSS References](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-cdoc-css) - this replaces the current "*EPUB* Style Sheets profile" by the "official definition" of *CSS*.

* [Addition of WCAG Support](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-cdoc-wcag) - adds the recommendation that all *HTML* Content Documents conform to the WCAG Guidelines to ensure they are accessible for people with disabilities.

Finally, the draft contains clarifications on [Foreign Resource Fallbacks](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-epub31-fallbacks) and [Scripting Support](http://www.idpf.org/epub/31/spec/epub-changes.html#sec-cdoc-scripting).

## *EPUB* 3.1 or *EPUB* 4.0?

By now it should be clear that the aggressive removal of features in *EPUB* 3.1 would have some far-reaching consequences. This is particularly true for the removal of the *NCX*, which would make *EPUB* 3.1 files incompatible with many existing E-ink readers. It would do this by ruling out the option to make backward-compatible "hybrid" files. As Gary McGath [pointed out earlier](https://fileformats.wordpress.com/2016/02/05/epub-3-1/), introducing "radical changes" in what is essentially a minor version is pretty unusual practice for any standard. Nowadays, most software and file formats use some variation of [semantic versioning](http://semver.org/), with version numbers that follow the general form *MAJOR.MINOR.PATCH*. Here, each component of the version number has a well-defined meaning:

1. *MAJOR* version is increased in case of incompatible API changes,
2. *MINOR* version is increased when functionality is added in a backwards-compatible manner, and
3. *PATCH* version is increased in case of backwards-compatible bug fixes.

Since the current draft includes multiple backward-incompatible changes, this makes me wonder why the editors didn't name it *EPUB* 4.0 instead! Kovid Goyal, lead developer of the popular [Calibre](https://calibre-ebook.com/) software, made [the following comment on this](https://github.com/IDPF/epub-revision/issues/642#issuecomment-182916894):  

> [I]f you want to make backwards incompatible changes, please, dont do it in a point release. From glancing over your changes document, it seems to me that you want to make several breaking changes. That's great, EPUB 3 could do with some serious breaking. But name it EPUB 4. I really dont want to have tell my users that calibre supports EPUB 3.1 but not EPUB 3. 

I agree with Kovid here. Having multiple sub-versions of *EPUB* 3, with *some* of them being backward-compatible with *EPUB* 2, while this backward compatibility is explicitly ruled out in another sub-version, is bound to create a situation that will be incomprehensible for most e-book buyers. Worse, it could even undermine overall confidence in the format. For memory institutions it would also make the management of *EPUB* 3 publications unnecessarily complicated. Not only would some *EPUB* 3.1 files not render correctly in an *EPUB* 3.0.1 reader, the opposite would be true as well. 

## Flashback

In my 2012 [report on *EPUB* for archival preservation](https://zenodo.org/record/839711) I already mentioned the stability of the *EPUB* format as a concern:

> EPUB 3 shows quite major changes relative to version 2, which raises concerns about the 
> format's stability over time. These concerns are reinforced by the fact that EPUB 3 is heavily 
> dependent on (X)HTML5 and CSS3, both of which are unfinished "works in progress", which 
> may undergo various changes before being finalised. 

These concerns are once more confirmed by the current *EPUB* 3.1 draft. However, it remains to be seen how many of these changes will make it to the final version. The community review process is ongoing at this moment, so if you're getting a little uneasy after reading this blog post, there's still time to get involved and make your voice heard!

## Acknowledgement

Thanks to Matt Garrish for his prompt replies to my questions on Github.

[^1]:  Following [EPUB 3.1 Changes from EPUB 3.0.1, Editor's Draft 30 January 2016](http://www.idpf.org/epub/31/spec/epub-changes.html)

[^2]: See here how [O’Reilly’s keeps their *EPUB* 3 books compatible with *EPUB* 2 readers](http://toc.oreilly.com/2013/02/oreillys-journey-to-epub-3.html)

[^3]: This figure includes reading systems for which support is unknown

[^4]: See the [*HTML5* Reference](https://dev.w3.org/html5/html-author/#the-html-and-xhtml-syntax) for a discussion of the differences between both syntaxes

[^5]: The [discussion thread on this topic in the issue tracker](https://github.com/IDPF/epub-revision/issues/642) is worth checking out, as it contains some excellent additional observations.

<hr>
Originally published at the [KB Research blog](http://blog.kbresearch.nl/2016/03/10/the-future-of-epub-a-first-look-at-the-epub-3-1-editors-draft/)
