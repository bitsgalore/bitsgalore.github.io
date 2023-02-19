---
layout: post
title: What do we mean by "embedded" files in PDF?
tags: [PDF]
comment_id: 55
---

The most important new feature of the recently released [PDF/A-3](http://www.iso.org/iso/catalogue_detail.htm?csnumber=57229) standard is that, unlike *PDF/A-2* and *PDF/A-1*, it allows you to embed *any* file you like. Whether this is a good thing or not is the subject of some [heated on-line discussions](http://blogs.loc.gov/digitalpreservation/2012/11/all-in-embedded-files-in-pdfa/). But what do we actually mean by *embedded files*? As it turns out, the answer to this question isn't as straightforward as you might think. One of the reasons for this is that in colloquial use we often talk about "embedded files" to describe the inclusion of *any* "non-text" element in a *PDF* (e.g. an image, a video or a file attachment). On the other hand, the word "embedded files" in the *PDF* standards (including *PDF/A*) refers to something much more specific, which is closely tied to *PDF*'s internal structure.

<!-- more -->

## Embedded files and embedded file streams

When the *PDF* standard mentions "embedded files", what it really refers to is a specific data structure. *PDF* has a *File Specification Dictionary* object, which in its simplest form is a table that contains a reference to some external file. *PDF 1.3* extended this, making it possible to embed the contents of referenced files directly within the body of the *PDF* using *Embedded File Streams*. They are described in detail in Section 7.11.4 of  the [*PDF* Specification (ISO 32000)](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf). A *File Specification Dictionary* that refers to an embedded file can be identified by the presence of an *EF* entry. 

Here's an example (source: [ISO 32000](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf)). First, here's a file specification dictionary:

```data
31 0 obj   
<</Type /Filespec /F (mysvg.svg) /EF <</F 32 0 R>> >>    
endobj
```

Note the  *EF* entry, which references another *PDF* object. This is the actual embedded file stream. Here it is:

```data
32 0 obj   
<</Type /EmbeddedFile /Subtype /image#2Fsvg+xml /Length 72>>   
stream  
…SVG Data…  
endstream  
endobj
```

Note that the part between the *stream* and *endstream* keywords holds the actual file data, here an *SVG* image, but this could really be anything!

So, in short, when the *PDF* standard mentions "embedded files", this really means *Embedded File Streams*. 

## So what about "embedded" images?

Here's the first source of confusion: if a *PDF* contains images, we often colloquially call these "embedded". However, internally they are not represented as *Embedded File Streams*, but as so-called *Image XObjects*. (In fact the *PDF* standard also includes yet another structure called *inline images*, but let's forget about those just to avoid making things even more complicated.) 

Here's an example of an *Image XObject* (again taken from  [ISO 32000](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf)):

```data
10 0 obj            % Image XObject
<< 
    /Type /XObject
    /Subtype /Image
    /Width 100
    /Height 200
    /ColorSpace /DeviceGray
    /BitsPerComponent 8
    /Length 2167
    /Filter /DCTDecode
>>
stream
…Image data…
endstream  
endobj
```

Similar to embedded filestreams, the part between the *stream* and *endstream* keywords holds the actual image data. The difference is that only a limited set of pre-defined formats are allowed. These are defined by the *Filter* entry (see Section 7.4 in [ISO 32000](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf)). In the example above, the value of *Filter* is *DCTDecode*, which means we are dealing with *JPEG* encoded image data.

## Embedded file streams and file attachments

Going back to embedded file streams, you may now start wondering what they are used for. According to Section 7.11.4.1 of [ISO 32000](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf), they are primarily intended as a mechanism to ensure that external references in a *PDF* (i.e. references to other files) remain valid. It also states:

> The embedded files are included purely for convenience and need not be directly processed by any conforming reader.

This suggests that the usage of embedded file streams is simply restricted to file attachments (through a *File Attachment Annotation* or an *EmbeddedFiles* entry in the document’s name dictionary).

Here's a sample file (created in *Adobe Acrobat* 9) that illustrates this:

<http://www.opf-labs.org/format-corpus/pdfCabinetOfHorrors/fileAttachment.pdf>

Looking at the underlying code we can see the *File Specification Dictionary*:

```data
37 0 obj    
<<
    /Desc()
    /EF<</F 38 0 R>>
    /F(KSBASE.WQ2)
    /Type/Filespec/UF(KSBASE.WQ2)>>
endobj
```

Note the */EF* entry, which means the referenced file  is embedded (the actual file data are in a separate stream object).

Further digging also reveals an *EmbeddedFiles* entry:

```data
33 0 obj   
<<
    /EmbeddedFiles 34 0 R
    /JavaScript 35 0 R
>>   
endobj
```

However, careful inspection of [ISO 32000](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf) reveals that embedded file streams can also be used for  multimedia! We'll have a look at that in the next section...

## Embedded file streams and multimedia

Section 13.2.1 (Multimedia) of the
[PDF Specification (ISO 32000)](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf) describes how multimedia content is represented in *PDF* (emphases added by me):

> - **Rendition actions** (...) shall be used to begin the playing of multimedia content.
>
> - A rendition action associates a **screen annotation** (...) with a **rendition** (...)
> - **Renditions** are of two varieties: **media renditions** (...) that define the characteristics of the media to be played, and selector renditions (...) that enables choosing which of a set of media renditions should be played.
> - **Media renditions** contain entries that specify what should be played (...), how it should be played (...), and where it should be played (...) 

The actual data for a media object are defined by *Media Clip Objects*, and more specifically by the *media clip data dictionary*. Its description (Section 13.2.4.2) contains a note, saying that this dictionary "may reference a URL to a streaming video presentation or a movie *embedded in the PDF file*". The description of the media clip data dictionary (Table 274) also states that the actual media data are "either a full file specification or a form XObject".

In plain English, this means that multimedia content in *PDF* (e.g. movies that are meant to be rendered by the viewer) may be represented internally as an embedded file stream.

The following sample file illustrates this:

<http://www.opf-labs.org/format-corpus/pdfCabinetOfHorrors/embedded_video_quicktime.pdf>

This *PDF* 1.7 file was created in *Acrobat* 9, and if you open it you will see a short *Quicktime* movie that plays upon clicking on it.

Digging through the underlying *PDF* code reveals a *Screen Annotation*, a *Rendition Action* and a *Media clip data dictionary*. The latter looks like this:

```data
41 0 obj
<<
    /CT(video/quicktime)
    /D 42 0 R
    /N(Media clip from animation.mov)
    /P<</TF(TEMPACCESS)>>
    /S/MCD
>>
endobj
```

It contains a reference to another object (42 0), which turns out to be a *File Specification Dictionary*:

```data
42 0 obj
<<
    /EF<</F 43 0 R>>
    /F(<embedded file>)
    /Type/Filespec
    /UF(<embedded file>)
>>
endobj
```

What's particularly interesting here is the */EF* entry, which means we're dealing with an embedded file stream here. (The actual movie data are in a stream object (43 0) that is referenced by the file specification dictionary.)

So, the analysis of this sample file confirms that embedded filestreams are actually used by *Adobe Acrobat* for multimedia content.

## What does *PDF/A* say on embedded file streams?

In [**PDF/A-1**](http://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=38920), embedded file streams are not allowed at all:

> A file specification dictionary (...) shall not contain the **EF** key. A file's name dictionary shall not contain the **EmbeddedFiles** key

In [**PDF/A-2**](http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=50655), embedded file streams *are* allowed, but only if the embedded file itself is *PDF/A* (1 or 2) as well: 

> A file specification dictionary, as defined in ISO 32000-1:2008, 7.11.3, may contain the **EF** key, provided that the embedded file is compliant with either ISO 19005-1 or this part of ISO 19005.

Finally, in [**PDF/A-3**](http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=57229) this last limitation was dropped, which means that *any* file may be embedded[^1].

## Does this mean *PDF/A-3* supports multimedia?

**No, not at all!** Even though nothing stops you from embedding multimedia content (e.g. a *Quicktime* movie), you wouldn't be able to use it as a renderable object inside a *PDF/A-3* document. The reason is that the *annotations* and *actions* that are needed for this (e.g. *Screen* annotations and *Rendition* actions, to name but a few) are not allowed in *PDF/A-3*. So effectively you are only able to use embedded file streams as attachments.

## Adobe adding to the confusion

A few weeks ago the embedding issue came up again in a [blog post by Gary McGath](http://fileformats.wordpress.com/2012/12/22/not-pdf/). One of the comments there is from Adobe's Leonord Rosenthol (who is also the Project Leader for *PDF/A*). After correctly pointing out some mistakes in both the original blog post and in an earlier a comment by me, he nevertheless added to the confusion by stating that objects that are are rendered by the viewer (movies, etc.) all use *Annotations*, and that embedded files (which he apparently uses a a synonym to attachments) are handled in a completely different manner. This doesn't appear to be completely accurate: at least one class of renderable objects (screen annotations/rendition actions) may be using embedded filestreams. Also, embedded files that are used as attachments may be associated with a *File Attachment Annotation*, which means that "under the hood" both cases are actually more similar than first meets the eye (which is confirmed by the analysis of the 2 sample files in the preceding sections). Contributing to this confusion is also the fact that Section 7.11.4 of [ISO 32000](http://www.adobe.com/content/dam/Adobe/en/devnet/acrobat/pdfs/PDF32000_2008.pdf) erroneously states that embedded file streams are only used for non-renderable objects like file attachments, which is contradicted by their allowed use for multimedia content.

## Does any of this matter, really?

Some might argue that the above discussion is nothing but semantic nitpicking. However, details like these *do* matter if we want to do a proper assessment of preservation risks in *PDF* documents. As an example, [in this previous blog post]({{ BASE_PATH }}/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression) I demonstrated how a *PDF/A* validator tool can be used to profile *PDF*s for "risky" features. Such tools typically give you a list of features. It is then largely up to the user to further interpret this information.

Now suppose we have a pre-ingest workflow that is meant to accept *PDF*s with multimedia content, while at the same time rejecting file attachments. By only using the presence of an embedded file stream (reported by both *Apache*'s and *Acrobat*'s *Preflight* tools) as a rejection criterion, we could end up unjustly rejecting files with multimedia content as well. To avoid this, we also need to take into account what the embedded file stream is used for, and for this we need to look at what annotation types are used, and the presence of any *EmbeddedFiles* entry in the document’s name dictionary. However, if we don't know precisely *which* features we are looking for, we may well arrive at the wrong conclusions!

This is made all the worse by the fact that preservation issues are often formulated in vague and non-specific ways. An example is [this issue on the OPF Wiki on the detection of "embedded objects"](http://wiki.opf-labs.org/display/AQuA/Embedded+objects+in+PDFs). The issue's description suggests that images and tables are the main concern (both of which aren't strictly speaking embedded objects). The [corresponding solution page](http://wiki.opf-labs.org/display/AQuA/Detect%2C+extract+and+analyse+embedded+objects+in+PDFs) subsequently complicates things further by also throwing file attachments in the mix. In order to solve issues like these, it is helpful to know that images are (mostly) represented as *Image XObjects* in *PDF*. The solution should then be a method for detecting *Image XObjects*. However, without some background knowledge of *PDF*'s internal data structure, solving issues like these becomes a daunting, if not impossible task.

## Final note

In this blog post I have tried to shed some light on a number of common misconceptions about embedded content in *PDF*. I might have inadvertently created some new ones in the process, so feel free to contribute any corrections or additions using the comment fields below.

The *PDF* specification is vast and complex, and I have only addressed a limited number of its features here. For instance, one might argue that a discussion of embedding-related features should also include fonts, metadata, *ICC* profiles, and so on. The coverage of multimedia features here is also incomplete, as I didn't include *Movie Annotations* or *Sound Annotations* (which preceded the *Screen Annotations*, which are now more commonly used). These things were all left out here because of time and space constraints.  This also means that further surprises may well be lurking ahead!

[^1]: Source: [this unofficial newsletter item](http://www.pdfa.org/2012/10/pdf-association-newsletter-issue-26/), as at this moment I don't have access to the full specification of *PDF/A-3*.
<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/01/09/what-do-we-mean-embedded-files-pdf/)
