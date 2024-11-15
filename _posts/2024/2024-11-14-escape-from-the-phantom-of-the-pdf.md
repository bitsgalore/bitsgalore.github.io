---
layout: post
title: Escape from the phantom of the PDF
headImage: "/images/2024/11/ghost.jpg"
headImageAltText: "Aquatint showing a graveyard scene in front of a church. At the center is a man in armour with a distressed expression on his face. To his left is a skeleton, and to his right a ghost."
description: "This post presents some tests on the support of octal escape sequences in PDF string objects. This is in response to a recent blogpost, whose authors argue that these are a preservation risk."
tags: [PDF, JHOVE, ExifTool, Apache-tika, VeraPDF, format-validation, preservation-risks]
comment_id: 93
---

<figure class="image">
  <img src="{{ BASE_PATH }}{{ page.headImage }}" alt="{{ page.headImageAltText }}">
  <figcaption><a href="https://wellcomecollection.org/works/sfqkzeu2">A man in armour is confronted by a ghost and a skeleton. </a>Aquatint. Wellcome Collection, Public Domain.</figcaption>
</figure>

In a [recent blog post](https://digitalpreservation.fi/en/2024-phantom-pdf-file), colleagues at the National Digital Preservation Services in Finland addressed an issue with PDF files that contain strings with octal escape sequences. These are not parsed correctly by [JHOVE](https://jhove.openpreservation.org/), and the resulting parse errors ultimately lead to (seemingly unrelated) validation errors. The authors argue that octal escape sequences present a preservation risk, as they may confuse other software besides JHOVE. Since this claim is not backed up by any evidence, here I put this to the test using 8 different PDF processing tools and libraries. 

<!-- more -->

## The Phantom of a PDF File

A few weeks ago I came across [The Phantom of a PDF File](https://digitalpreservation.fi/en/2024-phantom-pdf-file), a blog post by Juha Lehtonen & Johan Kylander. In this post, they address an issue with a [PDF file](https://digitalpreservation.fi/sites/default/files/2024/phantom_of_a_pdf_file_blog_post_2024.pdf) that gives an unexpected validation error in [JHOVE](https://jhove.openpreservation.org/). Digging into the PDF structure, they were able to track this down to the presence of [octal escape sequences](https://en.wikipedia.org/wiki/Escape_sequences_in_C) in the "Producer" field of the file's Document Information Dictionary:

```
9 0 obj
<<
/Title (Boo)
/CreationDate (D:20241029134330Z00'00') 
/Producer (\376\377\000P\000D\000F\000 \000P\000h\000a\000n\000t\000o\000m\000\000)
>>
endobj
```

JHOVE cannot handle octal escape sequences properly, which leads to a parse error that ultimately results in JHOVE reporting a validation error that is completely unrelated to the "Producer" field. In their original post, the authors claimed that the way the octal escape sequences are used in this file is not allowed by the PDF specification. They also advised against the use of octal escape sequences, and to restrict the values in PDF metadata fields to plain ASCII, or otherwise UTF-16BE. Finally they argue that the presence of octal escape sequences[^1] "raises the risk of causing problems in the future", and that "JHOVE probably is not the only software that will get confused" by this.

## Is JHOVE the only software that gets confused by octal escape sequences?

After reading this post, I posted the link to [this existing ticket on PDF parse issues](https://github.com/openpreserve/jhove/issues/927). This immediately resulted in [a response by Peter Wyatt](https://github.com/openpreserve/jhove/issues/927#issuecomment-2454914910) of the [PDF Association](https://pdfa.org/), who pointed out that, contrary to the claims made in the initial version of the post, the use of octal escape sequences in the "problematic" file is actually perfectly valid according to [ISO 32000-1](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf). In response to this comment, the authors corrected this in an update to their post. However, they seem to stick by their claim that "JHOVE probably is not the only software that will get confused" by octal escape sequences. Out of curiosity, I decided to put this to the test.

## Enter the OPF Phantom

The [original file](https://digitalpreservation.fi/en/2024-phantom-pdf-file) contains two "Producer" fields: one in the Document Information Dictionary (which is the one we're interested in here), and another one that is part of the XMP metadata. Since both have the same value ("PDF Phantom"), I changed the XMP value to "OPF Phantom" in a Hex editor. This way, we can easily distiguish between both fields. The modified file [is available here](https://github.com/user-attachments/files/17685339/phantom_modified_xmp.pdf).

Next, I ran the modified file through 8 different PDF tools and libraries, and inspected the result to see how the "Producer" field is handled. Below are the commands and results.

## [ExifTool](https://exiftool.org/)

Command:

```
exiftool -X phantom_modified_xmp.pdf
```
Result:

```xml
<?xml version='1.0' encoding='UTF-8'?>
<rdf:RDF xmlns:rdf='http://www.w3.org/1999/02/22-rdf-syntax-ns#'>

<rdf:Description rdf:about='phantom_modified_xmp.pdf'
  xmlns:et='http://ns.exiftool.org/1.0/' et:toolkit='Image::ExifTool 12.60'
  xmlns:ExifTool='http://ns.exiftool.org/ExifTool/1.0/'
  xmlns:System='http://ns.exiftool.org/File/System/1.0/'
  xmlns:File='http://ns.exiftool.org/File/1.0/'
  xmlns:PDF='http://ns.exiftool.org/PDF/PDF/1.0/'
  xmlns:XMP-x='http://ns.exiftool.org/XMP/XMP-x/1.0/'
  xmlns:XMP-pdf='http://ns.exiftool.org/XMP/XMP-pdf/1.0/'>
 <ExifTool:ExifToolVersion>12.60</ExifTool:ExifToolVersion>
 <System:FileName>phantom_modified_xmp.pdf</System:FileName>
 <System:Directory>.</System:Directory>
 <System:FileSize>5.9 kB</System:FileSize>
 <System:FileModifyDate>2024:11:09 00:22:05+00:00</System:FileModifyDate>
 <System:FileAccessDate>2024:11:09 00:22:46+00:00</System:FileAccessDate>
 <System:FileInodeChangeDate>2024:11:09 00:22:05+00:00</System:FileInodeChangeDate>
 <System:FilePermissions>-rw-rw-r--</System:FilePermissions>
 <File:FileType>PDF</File:FileType>
 <File:FileTypeExtension>pdf</File:FileTypeExtension>
 <File:MIMEType>application/pdf</File:MIMEType>
 <PDF:PDFVersion>1.4</PDF:PDFVersion>
 <PDF:Linearized>No</PDF:Linearized>
 <PDF:PageCount>1</PDF:PageCount>
 <PDF:Title>Boo</PDF:Title>
 <PDF:CreateDate>2024:10:29 13:43:30Z</PDF:CreateDate>
 <PDF:Producer>PDF Phantom</PDF:Producer>
 <XMP-x:XMPToolkit>Image::ExifTool 12.71</XMP-x:XMPToolkit>
 <XMP-pdf:Producer>OPF Phantom</XMP-pdf:Producer>
</rdf:Description>
</rdf:RDF>
```

ExifTool correctly decodes the octal escape sequences (PDF:Producer), and also extracts the XMP value (XMP-pdf:Producer).

## [Pdfcpu](https://pdfcpu.io/)

Command:

```
pdfcpu info phantom_modified_xmp.pdf
```

Result:

```
         PDF version: 1.4
          Page count: 1
           Page size: 595.28 x 841.89 points
............................................
               Title: Boo
              Author: 
             Subject: 
        PDF Producer: PDF Phantom
     Content creator: 
       Creation date: D:20241029134330Z00'00'
   Modification date: 
............................................
              Tagged: No
              Hybrid: No
          Linearized: No
  Using XRef streams: No
Using object streams: No
         Watermarked: No
............................................
           Encrypted: No
         Permissions: Full access
```

Pdfcpu correctly decodes the octal escape sequences (PDF Producer).

## pdfinfo ([Poppler](https://poppler.freedesktop.org/))

Command:

```
pdfinfo phantom_modified_xmp.pdf
```

Result:

```
Title:          Boo
Producer:       PDF Phantom
CreationDate:   Tue Oct 29 14:43:30 2024 CET
Tagged:         no
UserProperties: no
Suspects:       no
Form:           none
JavaScript:     no
Pages:          1
Encrypted:      no
Page size:      595.276 x 841.89 pts (A4)
Page rot:       0
File size:      5906 bytes
Optimized:      no
PDF version:    1.4
```

Poppler correctly decodes the octal escape sequences (Producer).

## [VeraPDF](https://verapdf.org/)

Command:

```
verapdf --off --extract phantom_modified_xmp.pdf
```

The result includes:

```xml
<informationDict>
  <entry key="Title">Boo</entry>
  <entry key="Producer">PDF Phantom#x000000</entry>
  <entry key="CreationDate">2024-10-29T13:43:30.000Z</entry>
</informationDict>
```

VeraPDF correctly decodes the octal escape sequences. Note the null character at the end (which is actually part of the object string).

## [Apache Tika](https://tika.apache.org/)

Command:

```
java -jar ~/tika/tika-app-2.9.2.jar phantom_modified_xmp.pdf
```

Result:

```html
<?xml version="1.0" encoding="UTF-8"?><html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta name="pdf:PDFVersion" content="1.4"/>
<meta name="pdf:docinfo:title" content="Boo"/>
<meta name="pdf:hasXFA" content="false"/>
<meta name="access_permission:modify_annotations" content="true"/>
<meta name="access_permission:can_print_degraded" content="true"/>
<meta name="dcterms:created" content="2024-10-29T13:43:30Z"/>
<meta name="dc:format" content="application/pdf; version=1.4"/>
<meta name="access_permission:fill_in_form" content="true"/>
<meta name="pdf:hasCollection" content="false"/>
<meta name="pdf:encrypted" content="false"/>
<meta name="dc:title" content="Boo"/>
<meta name="Content-Length" content="5906"/>
<meta name="pdf:hasMarkedContent" content="false"/>
<meta name="Content-Type" content="application/pdf"/>
<meta name="pdf:producer" content="OPF Phantom"/>
<meta name="access_permission:extract_for_accessibility" content="true"/>
<meta name="access_permission:assemble_document" content="true"/>
<meta name="xmpTPg:NPages" content="1"/>
<meta name="resourceName" content="phantom_modified_xmp.pdf"/>
<meta name="pdf:hasXMP" content="true"/>
<meta name="access_permission:extract_content" content="true"/>
<meta name="access_permission:can_print" content="true"/>
<meta name="X-TIKA:Parsed-By" content="org.apache.tika.parser.DefaultParser"/>
<meta name="X-TIKA:Parsed-By" content="org.apache.tika.parser.pdf.PDFParser"/>
<meta name="access_permission:can_modify" content="true"/>
<meta name="pdf:docinfo:producer" content="PDF Phantom�"/>
<meta name="pdf:docinfo:created" content="2024-10-29T13:43:30Z"/>
<title>Boo</title>
</head>
<body><div class="page"><p/>
</div>
</body></html>
```

Tika correctly decodes the octal escape sequences (field pdf:docinfo:producer); like VeraPDF it shows the null character at the end of the string. Note that Tika also reports the XMP Producer value (field pdf:producer).

## [Qpdf](https://qpdf.sourceforge.io/)

Command:

```
qpdf --json phantom_modified_xmp.pdf
```

Output contains:

```json
"9 0 R": {
  "/CreationDate": "D:20241029134330Z00'00'",
  "/Producer": "PDF Phantom\u0000",
  "/Title": "Boo"
}
```

Qpdf correctly decodes the octal escape sequences, including the trailing null character.

## [Pdftk](https://www.pdflabs.com/tools/pdftk-server/)

Command:

```
pdftk phantom_modified_xmp.pdf dump_data
```
Result:

```
InfoBegin
InfoKey: CreationDate
InfoValue: D:20241029134330Z00&apos;00&apos;
InfoBegin
InfoKey: Producer
InfoValue: PDF Phantom
InfoBegin
InfoKey: Title
InfoValue: Boo
PdfID0: 71a810587639eb130aefddee35e3c49d
PdfID1: 71a810587639eb130aefddee35e3c49d
NumberOfPages: 1
PageMediaBegin
PageMediaNumber: 1
PageMediaRotation: 0
PageMediaRect: 0 0 595.276 841.89
PageMediaDimensions: 595.276 841.89
```

Pdftk correctly decodes the octal escape sequences.

## [PyMuPDF](https://pymupdf.readthedocs.io/)

I tested PyMuPDF with this simple test script:

```python
import pprint
import pymupdf

myPDF = "phantom_modified_xmp.pdf"

doc = pymupdf.open(myPDF)
metadata = doc.metadata
pprint.pp(metadata)
```

Result:

```
{'format': 'PDF 1.4',
 'title': 'Boo',
 'author': '',
 'subject': '',
 'keywords': '',
 'creator': '',
 'producer': 'PDF Phantom',
 'creationDate': "D:20241029134330Z00'00'",
 'modDate': '',
 'trapped': '',
 'encryption': None}
```

PyMUPDF correctly decodes the octal escape sequences.

## Conclusion

All the above tools and libraries were able to decode the octal escape sequences without any problems. So JHOVE's behaviour really seems to be the exception here, rather than the rule. Octal escape sequences are both allowed by the standard, and widely supported by PDF processing tools. Concluding, I would argue that they don't pose a significant (if at all) preservation risk.

## Acknowledgments

Thanks are due to Peter Wyatt (PDF Association) for his helpful comments in [this Github thread](https://github.com/openpreserve/jhove/issues/927).

[^1]: Actually they refer to this as a "dual encoding", but this term is confusing because the octal escape sequences aren't really an "encoding" at all, but rather a part of the lexical definition of PDF literal string objects. See [this follow-up comment by Peter Wyatt](https://github.com/openpreserve/jhove/issues/927#issuecomment-2466229030).