---
layout: post
title: Identification of PDF preservation risks with VeraPDF and JHOVE
headImage: "/images/2023/05/Rock_em_Sock_em_Robots_Game.jpg"
headImageAltText: "Photo of a red toy robot and a similar looking blue toy robot in a boxing ring. Both robots face each other in a threatening stance."
description: "The PDF format has several features that are potential preservation risks. This post reviews to what extent such features can be detected using VeraPDF and JHOVE."
tags: [PDF, preservation-risks, VeraPDF, JHOVE]
comment_id: 88
---

<figure class="image">
  <img src="{{ BASE_PATH }}{{ page.headImage }}" alt="{{ page.headImageAltText }}">
  <figcaption><a href="https://commons.wikimedia.org/wiki/File:Rock_%27em_Sock_%27em_Robots_Game.jpg">"Rock 'em Sock 'em Robots Game"</a> by  Lorie Shaull, used under <a href="https://creativecommons.org/licenses/by-sa/4.0/deed.en">CC BY-SA 4.0</a>, via Wikimedia Commons.</figcaption>
</figure>

The PDF format has a number of features that don’t sit well with the aims of long-term preservation and accessibility. This includes encryption and password protection, external dependencies (e.g. fonts that are not embedded in a document), and reliance on external software. In this post I'll review to what extent such features can be detected using [VeraPDF](https://verapdf.org/) and [JHOVE](https://jhove.openpreservation.org/). It further builds on earlier work I did on this subject between 2012 and 2017.

<!-- more -->

## Context of this work

In 2019 the KB published its current [preservation policy](https://www.kb.nl/en/file-download/download/public/845) for its digital collections. The policy expresses a number of preservation-related goals for the upcoming years. One of these is a (gradual) move from pure [bit preservation](https://blogs.loc.gov/thesignal/2011/09/b-is-for-bit-preservation/) towards full functional preservation. To this end, the KB has defined three "knowledge levels", which are linked to file formats[^3]:

1. For **stored** formats, only bit preservation is done, without any formal format identification (no PRONOM identifier).
2. For **identified** formats, formal format identification has resulted in a PRONOM identifier.
3. For **known** formats, format validation has been performed, and technical metadata have been extracted. This information can then be used to identify preservation-related risks at the level of individual files, and actions to mitigate these risks can be taken if necessary.

The move from bit preservation to functional preservation involves raising the "knowledge level" status of the formats in our digital collections from "stored" (which is the current situation) to "identified" or, ideally, "known". This development coincides with the ongoing migration of these collections to our new [Rosetta archiving system](https://www.kb.nl/en/actueel/nieuws/dutch-national-library-steps-new-future-digital-archiving).

## PDF preservation risks

Being one of the [most prevalent formats]({{ BASE_PATH }}/2015/04/29/top-50-file-formats-in-the-kb-e-depot) in our collections, it comes as no surprise that PDF is one of the first formats for which we want to raise the current knowledge level. This is why my colleagues at our digital preservation department asked me take a new dive into PDF features that are potential preservation risks, and, more importantly, how to detect such features with existing software tools.

As long-term readers may remember, risk detection in PDF isn't an unfamiliar subject to me. Between 2012 and 2014 I experimented with detecting "risky" PDF features using the open-source [Apache PDFBox](https://pdfbox.apache.org/) software[^4]. This work was done as part of the [SCAPE project](https://scape-project.eu/). In 2017 I [repeated some of the SCAPE-era experiments]({{ BASE_PATH }}/2017/06/01/policy-based-assessment-with-verapdf-a-first-impression), this time using [VeraPDF](https://verapdf.org/). For various reasons that work never saw a proper follow-up, but six years onwards it's finally on the agenda once again! In the meantime others have produced relevant related work as well. The [PDF Significant Properties Spreadsheet](https://docs.google.com/spreadsheets/d/1eW7R8yACBciNimr16Z2ptC7fs1FlmZMnzdtG_DHBuD4/edit?usp=sharing) by Tyler Thorsted (currently at Brigham Young University) deserves a special mention here. It gives an overview of metadata fields and technical properties that are reported by 12 different software tools.

## PDF features and risks

Before going any further, it's important to be clear about our definition of preservation risks, and how they are related to PDF features. For this analysis I largely followed the earlier work that I mentioned above as a starting point. This means the focus is mostly on the following features and their associated risks: 

- Encryption and security-related features. The associated risks are restricted functionality (e.g. text access, copying, printing), or files that are inaccessible altogether (because they can only be opened using a password).
- External dependencies, such as fonts that are not embedded, or dependencies on external documents. The associated risk is that that documents may not render as originally intended if the external resources are not available.
- Multimedia features, including 3-D content. The associated risk is that the multimedia content may not render without dedicated software.
- File attachments. The associated risk is that attached files may require external software to render.
- JavaScript. This presents various risks, which are mostly security-related.

This list is not exhaustive, but it provides a useful starting point. Deviations from the format specification (PDF validity) can also pose potential preservation risks. [Opinions are divided](https://blog.dshr.org/2009/01/postels-law.html) about the importance of format validity in actual practice. PDF format validation is a huge subject in its own right[^7], which is out of the scope of the current analysis.

## VeraPDF vs JHOVE

The [2017 analysis]({{ BASE_PATH }}/2017/06/01/policy-based-assessment-with-verapdf-a-first-impression) already showed that VeraPDF was able to detect most risk-associated PDF features, which made the inclusion of VeraPDF an obvious choice for this follow-up. VeraPDF is currently not included in Rosetta, and my colleagues wondered to what extent [JHOVE](https://jhove.openpreservation.org/) (which is part of Rosetta's default setup) would be up to the job. So, in the remainder of this blog post I will present a comparison of VeraPDF and JHOVE.

It's important to mention that VeraPDF and JHOVE have different (but partially overlapping) scopes and functionalities. VeraPDF was primarily written to test for conformance against the various [PDF/A](https://en.wikipedia.org/wiki/PDF/A) profiles, with recent versions also supporting (partial) conformance testing for [PDF/UA](https://en.wikipedia.org/wiki/PDF/UA). It does not (or at best only to a limited degree) validate the lower-level data structures that make up a PDF file. The latter is the primary domain of JHOVE's PDF module, which was designed for "full on" PDF validation[^5]. The main overlap between VeraPDF and JHOVE lies in their ability to extract metadata and technical features, and for the current analysis I have only looked at this particular functionality of both tools.

The following table shows the evaluated VeraPDF and JHOVE versions: 

|Software|Version|
|:--|:--|
|VeraPDF|1.22.3|
|JHOVE|1.28.0|

By default, VeraPDF's feature extraction mode only extracts metadata from a PDF's document information dictionary. This is not sufficient for the purpose of this analysis, so I updated VeraPDF's configuration and enabled *all* feature extraction types[^8].

## Methods and test data

I used 3 different data sets for this analysis. First, used the [PDF Cabinet of Horrors corpus](https://github.com/openpreserve/format-corpus/tree/master/pdfCabinetOfHorrors). This is a small, hand-curated corpus of PDFs that is part of the [Open Preservation Foundation's file format corpus](https://github.com/openpreserve/format-corpus/tree/master). Since this is an annotated dataset with files that have known features (such as security-related features, non-embedded fonts, and multimedia), it provides excellent ground truth. I ran VeraPDF and JHOVE on all files in this data set, and scrutinised the resulting output files in detail. The main goal of this part of the analysis (which is presented in detail in the next section) was twofold:

1. Establish to what extent VeraPDF and JHOVE are able to detect the known features in these files.
2. Identify broad patterns in VeraPDF's and JHOVE's output that are associated with potential preservation risks.

The results then served as input for a second test, which is based on a subset of files from the (now defunct) [Adobe Acrobat Engineering website](https://web.archive.org/web/20130503115947/http://acroeng.adobe.com/wp/).

Finally, I tested how well VeraPDF and JHOVE are able to handle PDF 2.0 documents by running them on the PDF Association's [PDF 2.0 examples](https://github.com/pdf-association/pdf20examples) dataset.

## Advance warning

The level of detail in the following sections may be too much for some (most?) readers to digest, except perhaps for the most hardcore PDF freaks (you know who you are!). This applies in particular to the "Horror Corpus" section. If this is the case, you may want to skip right to the "Discussion" section from here. Otherwise, now is the time to fasten your seatbelts!

## Analysis of Horror Corpus

I will start with a rather detailed analysis of the [Horror Corpus](https://github.com/openpreserve/format-corpus/tree/master/pdfCabinetOfHorrors), as this is an annotated corpus of files with known features. Each of the following sub-sections covers one "risky" feature, which is demonstrated by one or more files in the Horror Corpus. For each feature, I show how it is represented in VeraPDF's and JHOVE's output (if it is represented at all). Even though this only covers a limited selection of "risky" features, this analysis is useful to get a first impression of the differences between VerapDF and JHOVE. It also enables us to identify which parts of the output of both tools are of potential interest for further perusal.

### Open password

#### Test files

|File|Description|
|:--|:--|
|[encryption_openpassword.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/encryption_openpassword.pdf?raw=true)|Opening the document requires password.|

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2015/07/openpassword.png" alt="Screenshot of dialog window that asks to enter a Document Open password.">
  <figcaption>Adobe Acrobat dialog on opening a PDF with an open password.</figcaption> 
</figure>

#### VeraPDF

For this file, VeraPDF reports an exception (full output [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_openpassword-vera.xml)):

```xml
<taskResult type="PARSE" isExecuted="true" isSuccess="false">
    <duration start="1683818755295" finish="1683818755343">00:00:00.048</duration>
    <exceptionMessage>Exception: The PDF stream appears to be encrypted. caused by exception: Reader::init(...)encrypted pdf is not supported</exceptionMessage>
</taskResult>
```

#### JHOVE

Unlike VeraPDF, JHOVE appears to be able to parse the file in spite of the open password. Its [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_openpassword-jhove.xml) contains an "Encryption" element with several child elements:

```xml
<property>
 <name>Encryption</name>
 <values arity="List" type="Property">
 <property>
  <name>SecurityHandler</name>
  <values arity="Scalar" type="String">
   <value>Standard</value>
  </values>
 </property>
 <property>
  <name>EFF</name>
  <values arity="Scalar" type="String">
   <value>Standard</value>
  </values>
 </property>
 <property>
  <name>Algorithm</name>
  <values arity="Scalar" type="String">
   <value>Document-defined</value>
  </values>
 </property>
 <property>
  <name>KeyLength</name>
  <values arity="Scalar" type="Integer">
   <value>128</value>
  </values>
 </property>
 <property>
  <name>StandardSecurityHandler</name>
  <values arity="List" type="Property">
  <property>
   <name>UserAccess</name>
   <values arity="List" type="String">
    <value>Print</value>
    <value>Modify</value>
    <value>Extract</value>
    <value>Add/modify annotations/forms</value>
    <value>Fill interactive form fields</value>
    <value>Extract for accessibility</value>
    <value>Print high quality</value>
   </values>
  </property>
  <property>
   <name>Revision</name>
   <values arity="Scalar" type="Integer">
    <value>4</value>
   </values>
  </property>
  <property>
   <name>OwnerString</name>
   <values arity="Scalar" type="String">
    <value>0x03a59d10aae3b50f1a30c34fbb1be09ababfc1fb19f0d3491ee84c1752671918</value>
   </values>
  </property>
  <property>
   <name>UserString</name>
   <values arity="Scalar" type="String">
    <value>0xf240da93aa83bb9f336cf249812c4db700000000000000000000000000000000</value>
   </values>
  </property>
  </values>
 </property>
 </values>
</property>
```

Even though it's helpful that JHOVE shows that this file contains security-related features, the above output doesn't provide any direct clue of the more specific open password feature[^9]. This is unfortunate, given that open passwords are one of the most serious PDF preservation risks.

### Copy, printing and text access passwords

#### Test files

|File|Description|
|:--|:--|
|[encryption_nocopy.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/encryption_nocopy.pdf?raw=true)|Copying document contents requires password.|
|[encryption_noprinting.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/encryption_noprinting.pdf?raw=true)|Printing requires password.|
|[encryption_notextaccess.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/encryption_notextaccess.pdf?raw=true)|Text access (e.g. by a screen reader) requires password.|

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2015/07/printpassword.png" alt="Screenshot of Adobe Acrobat showing a 'you cannot print this document' message.">
  <figcaption>Adobe Acrobat's rendering of a PDF with that requires a password for printing.</figcaption> 
</figure>

#### VeraPDF

Information on usage restrictions can be found in the *documentSecurity* element of VeraPDF's output. Below is an example (taken from the [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_nocopy-vera.xml) of the copy-protected file):

```xml
<documentSecurity>
    <filter>Standard</filter>
    <version>4</version>
    <length>128</length>
    <ownerKey>5F7FA03AD5A40C66...</ownerKey>
    <userKey>A572ACDFF8B3DC74...</userKey>
    <encryptMetadata>true</encryptMetadata>
    <printAllowed>true</printAllowed>
    <printDegradedAllowed>true</printDegradedAllowed>
    <changesAllowed>true</changesAllowed>
    <modifyAnnotationsAllowed>true</modifyAnnotationsAllowed>
    <fillingSigningAllowed>true</fillingSigningAllowed>
    <documentAssemblyAllowed>false</documentAssemblyAllowed>
    <extractContentAllowed>false</extractContentAllowed>
    <extractAccessibilityAllowed>true</extractAccessibilityAllowed>
</documentSecurity>
```

Here, the "false" value of the *extractContentAllowed* element indicates this file is copy-protected. The *printAllowed* and *printDegradedAllowed* elements indicate whether printing is allowed (example [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_noprinting-vera.xml)), and "false" values of *extractContentAllowed* and *extractAccessibilityAllowed* indicate a text access password (example [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_notextaccess-vera.xml)).

#### JHOVE

For JHOVE, the *UserAccess* property (which is a child element of the *Encryption* property) provides information about usage restrictions. Below for the copy-restricted document (full output [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_nocopy-jhove.xml)): 

```xml
<property>
    <name>UserAccess</name>
    <values arity="List" type="String">
        <value>Print</value>
        <value>Modify</value>
        <value>Add/modify annotations/forms</value>
        <value>Fill interactive form fields</value>
        <value>Extract for accessibility</value>
        <value>Print high quality</value>
    </values>
</property>
```

And here for the print-restricted document (full output [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/encryption_noprinting-jhove.xml)):

```xml
<property>
    <name>UserAccess</name>
    <values arity="List" type="String">
        <value>Modify</value>
        <value>Extract</value>
        <value>Add/modify annotations/forms</value>
        <value>Fill interactive form fields</value>
        <value>Extract for accessibility</value>
    </values>
</property>
```

The above examples show that JHOVE only reports user access types that are *permitted*. This means that if we want to verify if a document is print-restricted, we need to check for the *absence* of the values *Print* and *Print high quality*. This is impractical, because it assumes the user has *a priori* knowledge of all possible values that JHOVE is capable of reporting (which are also undocumented). VeraPDF's makes this considerably easier by always reporting *all* user access types and their respective values. 

### Multimedia

#### Test files

|File|Description|
|:--|:--|
|[embedded_video_quicktime.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/embedded_video_quicktime.pdf?raw=true)|Contains embedded Quicktime movie.|
|[embedded_video_avi.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/embedded_video_avi.pdf?raw=true)|Contains embedded AVI movie.|

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2015/07/embeddquicktime.png" alt="Screenshot of Adobe Acrobat's rendering of a PDF with an embedded Quicktime movie. As Acrobat cannot play multimedia content natively, it shows a dialog saying 'The media requires an additional player. Please click Get Media Player to download the correct media player'.">
  <figcaption>Adobe Acrobat's rendering of a PDF with an embedded Quicktime movie.</figcaption> 
</figure>

#### VeraPDF

For both files the presence of the multimedia content is indicated by two elements in [VeraPDF's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/embedded_video_quicktime-vera.xml). First, the *actions* element contains a reference to a Rendition action:

```xml
<action type="Rendition">
    <location>Annotation</location>
</action>
```

 Furthermore, the *annotations* element contains a reference to a Screen annotation:

 ```xml
<annotation id="annotIndir35">
    <subType>Screen</subType>
    <rectangle lly="360.647" llx="73.180" urx="393.180" ury="600.647"></rectangle>
    <width>320.000</width>
    <height>240.000</height>
    <resources>
        <xobject id="xobjIndir39"></xobject>
    </resources>
    <invisible>false</invisible>
    <hidden>false</hidden>
    <print>true</print>
    <noZoom>false</noZoom>
    <noRotate>false</noRotate>
    <noView>false</noView>
    <readOnly>false</readOnly>
    <locked>false</locked>
    <toggleNoView>false</toggleNoView>
    <lockedContents>false</lockedContents>
</annotation>
 ```

#### JHOVE

The [JHOVE output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/embedded_video_quicktime-jhove.xml) only refers to the Screen annotation for these files (JHOVE doesn't report on actions):

```xml
<property>
    <name>Annotation</name>
    <values arity="List" type="Property">
    <property>
        <name>Subtype</name>
        <values arity="Scalar" type="String">
        <value>Screen</value>
        </values>
    </property>
        ::
</property>
```

Besides Rendition actions and Screen annotations, there are several other action and annotation types that indicate multimedia content, and we'll see some examples later on in the analysis of the Adobe Acrobat Engineering files.

### JavaScript

#### Test files

|File|Description|
|:--|:--|
|[javascript.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/javascript.pdf?raw=true)|Contains JavaScript.|

#### VeraPDF

The *actions* element of [VeraPDF's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/javascript-vera.xml) contains the following JavaScript action reference[^1]:

Result:

```xml
<action type="JavaScript">
<location>Document</location>
</action>
```

#### JHOVE

I couldn't find any JavaScript reference in the [JHOVE output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/javascript-jhove.xml) for the same file.

### Font embedding

#### Test files

|File|Description|
|:--|:--|
|[text_only_fontsNotEmbedded.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/text_only_fontsNotEmbedded.pdf?raw=true)|Only uses fonts that are not embedded.|
|[text_only_fontsEmbeddedAll.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/text_only_fontsEmbeddedAll.pdf?raw=true)|Only uses fonts that are embedded.|

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2015/07/fontslinux.png" alt="Screenshot of Adobe Acrobat rendering a PDF with a font whose glyphs are placed at odd positions.">
  <figcaption>Adobe Acrobat's rendering of a PDF that uses non-embedded font that is not available on the target machine.</figcaption> 
</figure>

#### VeraPDF

The *fonts* element in VeraPDF's output contains child elements for each font that is used in a document. For the PDF that only uses fonts that are not embedded, this results in the following output (full output file [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/text_only_fontsNotEmbedded-vera.xml)):

```xml
<font id="fntIndir30">
<type>TrueType</type>
<baseFont>TimesNewRomanPSMT</baseFont>
<firstChar>32</firstChar>
<lastChar>255</lastChar>
<encoding>WinAnsiEncoding</encoding>
<fontDescriptor>
     <subset>false</subset>
     <fontName>TimesNewRomanPSMT</fontName>
     <fontFamily>Times New Roman</fontFamily>
     <fontStretch>Normal</fontStretch>
     <fontWeight>400.000</fontWeight>
     <fixedPitch>false</fixedPitch>
     <serif>true</serif>
     <symbolic>false</symbolic>
     <script>false</script>
     <nonsymbolic>true</nonsymbolic>
     <italic>false</italic>
     <allCap>false</allCap>
     <smallCap>false</smallCap>
     <forceBold>false</forceBold>
     <fontBBox lly="-307.000" llx="-568.000" urx="2000.000" ury="1007.000"></fontBBox>
     <italicAngle>0.000</italicAngle>
     <ascent>891.000</ascent>
     <descent>-216.000</descent>
     <leading>0.000</leading>
     <capHeight>1000.000</capHeight>
     <xHeight>1000.000</xHeight>
     <stemV>82.000</stemV>
     <stemH>0.000</stemH>
     <averageWidth>0.000</averageWidth>
     <maxWidth>0.000</maxWidth>
     <missingWidth>0.000</missingWidth>
     <embedded>false</embedded>
</fontDescriptor>
</font>
```

Here, the value of *embedded* in the *fontDescriptor* sub-element indicates whether a font is embededded or not (in this example it isn't). So, to check if a PDF uses fonts that are not embedded, one can simply iterate over all *font* elements and check the value of */fontDescriptor/embedded* for each of these.

#### JHOVE

The [JHOVE output for the same PDF](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/text_only_fontsNotEmbedded-jhove.xml) does contain several font-related properties, but none of them are related to embedding. So, I initially assumed that JHOVE simply didn't report this. However, when I looked at [JHOVE's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/text_only_fontsEmbeddedAll-jhove.xml) for the PDF with only embedded fonts, I noticed this property (part of a *FontDescriptor* element): 

```xml
<property>
    <name>FontFile2</name>
    <values arity="Scalar" type="Boolean">
    <value>true</value>
    </values>
</property>
```

Since JHOVE doesn't report the *FontFile2* property for the PDF without embedded fonts, I wondered if this was a coincidence. As the meaning of this property (or any of JHOVE's properties for that matter) is not documented, I took a peek at [JHOVE's source code](https://github.com/openpreserve/jhove/blob/94da570caa55759354fa6fcd50e4ea7edbba1e7d/jhove-modules/pdf-hul/src/main/java/edu/harvard/hul/ois/jhove/module/PdfModule.java#L3830). This revealed that the *FontFile2* property originates from the "font descriptor" dictionary, which is documented in section 9.8 (Font Descriptors) of the [ISO 32000-1 PDF specification](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf). Here we can see (Table 122) that the keys *FontFile*, *FontFile2* and *FontFile3* (which also exist as separate JHOVE properties) all indicate embedded fonts.

This means we can actually use JHOVE's output to check for font embedding, but to identify a PDF with one or more fonts that are not embedded, one needs to iterate over all font property groups, and then check for the *absence* of either of three separate properties (*FontFile*, *FontFile2* and *FontFile3*). None of these properties are documented. This is not ideal to begin with, and made worse by JHOVE's rather labyrinthine output format.

### File attachments

#### Test files

|File|Description|
|:--|:--|
|[fileAttachment.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/fileAttachment.pdf?raw=true)|Contains file attachment that uses EmbeddedFiles entry.|
|[fileAttachment_fileAttachmentAnnotation.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/fileAttachment_fileAttachmentAnnotation.pdf?raw=true)|Contains file attachment that uses File Attachment Annotation.|

File attachments in PDF can be implemented in two different ways, using either an *EmbeddedFiles* entry in the document’s name dictionary, or a File Attachment Annotation.

#### VeraPDF

For the sample file with the *EmbeddedFiles* entry, [VeraPDF's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/fileAttachment-vera.xml) contains an *embeddedFiles* element with one or more child elements:

```xml
<embeddedFiles>
    <embeddedFile id="file1">
    <fileName>KSBASE.WQ2</fileName>
    <description></description>
    <filter>FlateDecode</filter>
    <creationDate>2012-11-23T15:40:38.000+01:00</creationDate>
    <modDate>2012-11-19T12:35:10.000Z</modDate>
    <checkSum>)#x000002Aﬁ˛½�ﬂô#x000004‘SèŠ-¡</checkSum>
    <size>20668</size>
    </embeddedFile>
</embeddedFiles>
```

For the sample file with the File Attachment Annotation, the *annotations* element in VeraPDF's [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/fileAttachment_fileAttachmentAnnotation-vera.xml) contains an *annotation* child element that has "FileAttachment" as its subtype:

```xml
<annotation id="annotIndir26">
    <subType>FileAttachment</subType>
    <rectangle lly="562.840" llx="185.281" urx="199.281" ury="582.840"></rectangle>
    <width>14.000</width>
    <height>20.000</height>
    <contents>PF.WK1</contents>
    <annotationName>9a716f25-da75-4309-b918-483cfa9f6473</annotationName>
    <modifiedDate>D:20131024143706+02'00'</modifiedDate>
    <resources>
        <xobject id="xobjIndir29"></xobject>
    </resources>
    <color red="0.250000" green="0.333328" blue="1.000000"></color>
    <invisible>false</invisible>
    <hidden>false</hidden>
    <print>true</print>
    <noZoom>true</noZoom>
    <noRotate>true</noRotate>
    <noView>false</noView>
    <readOnly>false</readOnly>
    <locked>false</locked>
    <toggleNoView>false</toggleNoView>
    <lockedContents>false</lockedContents>
</annotation>
```

#### JHOVE

For the sample file with the *EmbeddedFiles* entry, [JHOVE's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/fileAttachment-jhove.xml) contains a *PageMode* element with *UseAttachments* as one of its values:

```xml
<property>
    <name>PageMode</name>
    <values arity="Scalar" type="String">
        <value>UseAttachments</value>
    </values>
</property>
```

For the sample file with the File Attachment Annotation, JHOVE's [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/fileAttachment_fileAttachmentAnnotation-jhove.xml) reports a "FileAttachment" annotation:

```xml
<property>
<name>Annotation</name>
<values arity="List" type="Property">
<property>
     <name>Subtype</name>
     <values arity="Scalar" type="String">
     <value>FileAttachment</value>
     </values>
</property>
<property>
```

### Link to external file

#### Test files

|File|Description|
|:--|:--|
|[externalLink.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/externalLink.pdf?raw=true)|Contains link to an external document.|

#### VeraPDF

For the test file, the *actions* element of [VeraPDF's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/externalLink-vera.xml) contains a "Launch" action:

```xml
<action type="Launch">
     <location>Annotation</location>
</action>
```

Further to that, the *annotations* element contains a child element for a "Link" annotation:

```xml
<annotation id="annotIndir35">
     <subType>Link</subType>
     <rectangle lly="678.816" llx="70.860" urx="200.820" ury="694.584"></rectangle>
     <width>129.960</width>
     <height>15.768</height>
     <invisible>false</invisible>
     <hidden>false</hidden>
     <print>true</print>
     <noZoom>false</noZoom>
     <noRotate>false</noRotate>
     <noView>false</noView>
     <readOnly>false</readOnly>
     <locked>false</locked>
     <toggleNoView>false</toggleNoView>
     <lockedContents>false</lockedContents>
</annotation>
```

As explained in section 12.6.4.5 (Launch actions) of [ISO 32000-1](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf):

> A launch action launches an application or opens or prints a document.

So, it is not exclusively associated with external *documents*, but it might also indicate a dependency on external *software*, which makes it all the more relevant for preservation. Link annotations can also serve several purposes. Section 12.5.6.5 (Link annotations) of ISO 32000-1:

> A link annotation represents either a hypertext link to a destination elsewhere in the document (...) or an action to be performed.

If I'm reading this correctly, the presence of a link annotation alone not always imply an external dependency. 

#### JHOVE

[JHOVE's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/externalLink-jhove.xml) only contains this reference to the Link annotation:

```xml
<property>
    <name>Annotation</name>
    <values arity="List" type="Property">
    <property>
        <name>Subtype</name>
        <values arity="Scalar" type="String">
            <value>Link</value>
        </values>
    </property>
        ::
<property>
```

JHOVE doesn't report any actions (including the Launch action in this file).

### Web Capture content

#### Test files

|File|Description|
|:--|:--|
|[webCapture.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/webCapture.pdf?raw=true)|Contains Web Capture content.|

When I created this test file back in 2012, I was under the impression that all Web Capture content by its nature had a dependency on the live web. Reading section 14.10 (Web Capture) of [ISO 32000-1](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf) once more, it turns out that this is not the case:

> The information in the Web Capture data structures enables conforming products to perform the following
operations:
> - Save locally and preserve the visual appearance of material from the Web
> - Retrieve additional material from the Web and add it to an existing PDF file
> - Update or modify existing material previously captured from the Web

So, Web Capture content may simply be a local copy of material that was captured from the web at the time of the PDF's creation. This seems to be the case for this test file. When I open it on my machine (using [Xreader](https://github.com/linuxmint/xreader/)), it shows a snapshot of the Open Planets Foundation website from the time of the document's creation (2012): 

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/05/webcapture-xreader.png" alt="Screenshot that shows how the test file with web capture content is rendered in Xreader.">
</figure>

It's not entirely clear to me how and under what circumstances additional material is retrieved from the web, or existing material is updated. This makes it hard to judge how much of a preservation risk Web Capture content poses in actual practice.

#### VeraPDF

The [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/webCapture-vera.xml) of VeraPDF contains several interesting bits. First, there are these references to a GoTo action, a URI action and a SubmitForm action: 

```xml
 <action type="GoTo">
<location>Annotation</location>
</action>
<action type="URI">
<location>Annotation</location>
</action>
<action type="SubmitForm">
<location>Annotation</location>
</action>
```

Moreover, the *annotations* element contains a large number of Link annotations:

```xml
<annotation id="annotIndir163">
<subType>Link</subType>
<rectangle lly="720.000" llx="11.000" urx="88.000" ury="760.000"></rectangle>
<width>77.000</width>
<height>40.000</height>
<invisible>false</invisible>
<hidden>false</hidden>
<print>false</print>
<noZoom>false</noZoom>
<noRotate>false</noRotate>
<noView>false</noView>
<readOnly>false</readOnly>
<locked>false</locked>
<toggleNoView>false</toggleNoView>
<lockedContents>false</lockedContents>
</annotation>
```

#### JHOVE

Looking at [JHOVE's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/webCapture-jhove.xml), I couldn't find any reference to the Web Capture content at all. The missing references to the GoTo action, URI action and SubmitForm action are understandable, since JHOVE doesn't report any actions. The absence of any reference to the numerous Link annotations is nevertheless surprising.

### Byte corruption

#### Test files

|File|Description|
|:--|:--|
|[corruptionOneByteMissing.pdf](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/corruptionOneByteMissing.pdf?raw=true)|Has one byte of missing data, immediately following the file header.|

#### VeraPDF

The byte corruption in this file causes an exception in VeraPDF. Details can be found in the *taskResult* element of the [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/corruptionOneByteMissing-vera.xml) file:

Result:

```xml
<taskResult type="PARSE" isExecuted="true" isSuccess="false">
    <duration start="1683818745078" finish="1683818745121">00:00:00.043</duration>
    <exceptionMessage>Exception: Couldn't parse stream caused by exception: Pages not found</exceptionMessage>
</taskResult>
```

It looks like the *taskResult* element is currently *only* included if an exception occurs. It would be more consistent to include it for *every* processsed file. This would make it more straightforward to use the output to identify PDFS that caused any exceptions or parse errors in VeraPDF. These are usually an indication that something is seriously wrong with a file, and the value of the *isSuccess* attribute could be used as a rough validation proxy. I've created [an issue](https://github.com/veraPDF/veraPDF-library/issues/1336) for this.

#### JHOVE

[JHOVE's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/horror/corruptionOneByteMissing-jhove.xml) contains the following error:

```xml
<message offset="0" severity="error" id="PDF-HUL-140">Document catalog dictionary object number and trailer root ref number are inconsistent.</message>
```

Since this test file only represents one very specific case of byte corruption, I expect that other types might result in quite different JHOVE errors (but this is outside the scope of this investigation).

## Analysis Adobe Acrobat Engineering dataset

The results of the Horror Corpus analysis highlight the significance of actions and annotations, which are often indicative of features that are associated with preservation risks. The Horror Corpus analysis already showed that for one test file, annotations that were correctly reported by VeraPDF, were subsequently not picked up by JHOVE. I did some further tests to determine whether this was an isolated case, or perhaps an indication of a more structural problem. For these tests I used the files in the ["Classic Multimedia"](https://web.archive.org/web/20130726144923/http://acroeng.adobe.com/wp/?page_id=61) category of the Adobe Acrobat Engineering dataset. These files make heavy use of actions and annotations, which makes them well suited for this purpose.

### Results

The full VeraPDF and JHOVE output for these files can be found [here](https://github.com/KBNLresearch/pdf-characterisation/tree/main/output/ae-multimedia). The following table summarizes the actions and annotations that are reported by VeraPDF and JHOVE:

|File|Actions (VeraPDF)|Annotations (VeraPDF)|Annotations (JHOVE)|
|:--|:--|:--|:--|
|AdobeChassisDemo-commented_Review.pdf|JavaScript|3D<br>Ink<br>Text<br>Popup<br>Polygon|Popup<br>Ink<br>Text<br>3D<br>Polygon|
|movie.pdf||Movie||
|MusicalScore.pdf|GoTo<br>JavaScript<br>URI<br>Rendition|Screen<br>Widget<br>Link|Link<br>Widget<br>Screen|
|LabelExample.pdf|GoTo3DView<br>JavaScript|3D<br>Widget|3D<br>Widget|
|MultiMedia_Acro6.pdf|Rendition|Screen|Screen|
|AVI+Transitions Demo.pdf||||
|20020402_CALOS.pdf|Movie<br>Hide<br>Named<br>GoTo|Link<br>Movie<br>Widget||
|Disney-Flash.pdf|URI<br>Rendition<br>SubmitForm|Widget<br>Link<br>Screen|Widget<br>Link<br>Screen|
|gXsummer2004-stream.pdf||||
|SVG-AnnotAnim.pdf||SVG||
|Service Form_media.pdf|SubmitForm<br>Named<br>ResetForm<br>JavaScript<br>Rendition|Widget<br>Link<br>Screen|Screen<br>Widget<br>Link|
|Binder_6-3DPages.pdf|GoTo|3D|3D|
|us_population.pdf|GoTo|SVG|SVG|
|SVG.pdf|JavaScript<br>URI|Widget|Widget|
|phlmapbeta7.pdf|Rendition|Screen|Screen|
|ScriptEvents.pdf|JavaScript<br>GoTo<br>Rendition|Screen<br>Widget|Widget<br>Screen|
|Trophy.pdf|JavaScript<br>URI<br>GoTo<br>Rendition|Screen<br>Widget<br>Link|Widget<br>Link<br>Screen|
|VolvoS40V50-Full.pdf|Named<br>JavaScript<br>GoTo<br>URI<br>Rendition|Widget<br>Link<br>Screen|Screen<br>Widget<br>Link|
|Jpeg_linked.pdf|Named<br>GoTo<br>Rendition|Link<br>Screen|Link<br>Screen|
|AdobeChassisDemo-commented.pdf||3D<br>Popup<br>Polygon<br>Text|Popup<br>Polygon<br>3D<br>Text|
|drape_raster_contour_sample.pdf|URI|Link<br>3D|3D<br>Link|
|remotemovieurl.pdf||Movie<br>FreeText||
|3-D_PDF.pdf||3D|3D|
|movie_down1.pdf||Movie||

Note that the table doesn't have an "Actions" column for JHOVE. This is because, unlike VeraPDF, JHOVE's output doesn't include any information on actions at all.

Both VeraPDF and JHOVE provide information about annotations, and even though the results are broadly similar for both tools, there are some noteworthy differences.

For the files ["movie.pdf"](https://web.archive.org/web/20100714002808/http://acroeng.adobe.com:80/Test_Files/movie/movie.pdf), ["20020402_CALOS.pdf"](https://web.archive.org/web/20140519130059/http://acroeng.adobe.com/Test_Files/classic_multimedia//20020402_CALOS.pdf), ["remotemovieurl.pdf"](https://web.archive.org/web/20100714002816/http://acroeng.adobe.com:80/Test_Files/movie/remotemovieurl.pdf) and ["movie_down1.pdf"](https://web.archive.org/web/20100714002811/http://acroeng.adobe.com:80/Test_Files/movie/movie_down1.pdf), JHOVE doesn't report any annotations at all. In all four cases the [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/ae-multimedia/20020402_CALOS-jhove.xml) contains the following annotation-related validation error;

```xml
<message offset="17342" severity="error" id="PDF-HUL-120">Annotation dictionary missing required type (S) entry</message>
```

VeraPDF's output shows that all of these files contain a Movie annotation. I'm not sure if this is a coincidence.

For the file "SVG-AnnotAnim.pdf", JHOVE doesn't report the SVG annotation. A look at the [output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/ae-multimedia/SVG-AnnotAnim-jhove.xml) shows that JHOVE wasn't able to parse this file at all, with the following validation error:

```xml
<message offset="0" severity="error" id="PDF-HUL-144">Pages dictionary has no Type key or it has a null value.</message>
```

### Behaviour with corrupted files

Two PDFs in this dataset are corrupted: "gXsummer2004-stream.pdf" and "AVI+Transitions Demo.pdf". For both files, [VeraPDF's output](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/ae-multimedia/gXsummer2004-stream-vera.xml) reports a parse exception:

```xml
<taskResult type="PARSE" isExecuted="true" isSuccess="false">
```

Meanwhile [JHOVE](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/ae-multimedia/gXsummer2004-stream-jhove.xml) throws a "No PDF header" validation error:

```xml
<message offset="0" severity="error" id="PDF-HUL-137">No PDF header</message>
```

## Support for PDF 2.0

Since it's been almost six years now since the [launch](https://www.iso.org/news/ref2199.html) of PDF 2.0, I was curious to see to what extent it is supported by VeraPDF and JHOVE. To find out, I ran both on [this set of PDF 2.0 example files](https://github.com/pdf-association/pdf20examples) by the PDF Association.

Out of the 7 files in this dataset, 6 resulted in the following JHOVE error (full output [here](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/pdf20examples/Simple%20PDF%202.0%20file-jhove.xml)):

```xml
<message offset="0" severity="error" id="PDF-HUL-137">No PDF header</message>
```
These files were subsequently not parsed at all. The only exception here is this ["PDF 2.0 via incremental save.pdf"](https://github.com/pdf-association/pdf20examples/blob/master/PDF%202.0%20via%20incremental%20save.pdf) file, which has "1.7" as its designated version number in the PDF header.

By contrast, VeraPDF was able to read all PDF 2.0 files in this dataset without any problems.

## Discussion

The combined results of the above analyses of the Horror Corpus, the Adobe Acrobat Engineering dataset and the PDF 2.0 example files allow us to draw a number of conclusions.

### JHOVE

First of all, even though both VeraPDF and JHOVE are able to detect many PDF features that are associated with known preservation risks, JHOVE has several limitations that make it less appealing for this purpose:

- JHOVE's output doesn't include any information about **actions**, which means it cannot be used for detecting e.g. JavaScript or Launch actions (which are often indicative of external dependencies).
- Many PDF features that are preservation risks are associated with **annotations**. Even though JHOVE is able to report most of these, the comparison against VeraPDF's output shows that JHOVE's reporting of annotations is often incomplete.
- JHOVE's reporting on **encryption and security-related restrictions** could be more informative. Most importantly, it doesn't allow one to single out files that require an **open password** (which present one of the most serious PDF preservation risks).
- Moreover, JHOVE makes the detection of **user access restrictions** (print, copy and text access) unnecessarily difficult by only reporting these features if they are *not* restricted. This is impractical, because it means we have to check for the *absence* of these features in JHOVE's output. It also assumes prior knowledge on the user's behalf of properties that are completely undocumented.
- Even though JHOVE does report whether **fonts are embedded**, this information is encoded in a way that is needlessly complicated, because it involves checking the output for the *absence* (again!) of three properties that are undocumented.
- **No documentation exists of any of JHOVE's reported properties**. This can make their interpretation difficult. The aforementioned font embedding issue is a good example of this: if we want to know if a particular font is embedded or not, we need to check three separate properties (*FontFile*, *FontFile2* and *FontFile3*), which are all undocumented. I was only able to figure this out by digging into JHOVE's source code, and consulting the ISO 32000-1 PDF specification.
- Another (but related) problem is JHOVE's rather **clumsy and convoluted XML output format**, which is made up by a labyrinthine assortment of nested "property" elements. This makes the format difficult to read, either by a human or a machine[^2]. It's not possible to address the reported properties directly using, for example, standard [xpath](https://en.wikipedia.org/wiki/XPath) expressions. This is because all properties are encoded as identically-named *property* elements, where the name of each individual property is the text value of its *name* child element. Of course this doesn't preclude parsing the format altogether, but it does make working with JHOVE's output considerably harder than most modern, well-designed XML formats.
- Finally, the tests showed that JHOVE **lacks support for PDF 2.0**. On encountering a document with a PDF 2.0 header, JHOVE simply reports an error and stops parsing the file. As a result, JHOVE's output on PDF 2.0 documents does not contain any meaningful information.

### VeraPDF

VeraPDF was able to detect all features that are associated with known preservation risks in the "Horror Corpus" files. Unlike JHOVE, VeraPDF also reports on actions. VeraPDF's coverage of annotations is more comprehensive, and it also provides detailed information about security-related restrictions. I was unable to find any detailed documentation of VeraPDF's reported properties, but the documentation does provide a [general overview of the reported feature categories](https://docs.verapdf.org/cli/config/#features.xml). Also, VeraPDF's XML output format is much simpler, better structured and easier to navigate than the one used by JHOVE. Each reported property can be addressed directly by the name of its corresponding XML element, and the property names are mostly self-explanatory.

For convenience, the following table summarises the elements in VeraPDF's output that turned out to be the most interesting ones for the current analysis (asterisks indicates repeatable elements):

|Element (relative to *job* element)|Significance|
|:--|:--|
|taskResult|Parse status, rough validity proxy, may be used to identify malformed files and files that require open password|
 featuresReport/documentSecurity|Child elements indicate security-related restrictions (e.g. copy, printing and text access passwords)|
|featuresReport/actions/action\*|Some actions imply a preservation risk|
|featuresReport/annotations/annotation\*|Some annotations imply a preservation risk|
|featuresReport/documentResources/fonts/font/fontDescriptor/embedded\*|Value indicates whether font is embedded or not|
|featuresReport/embeddedFiles/embeddedFile\*|Indicates presence of file attachment|

In order to report these, at minimum the following feature types must be enabled in VeraPDF's configuration file:

``` xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<featuresConfig>
    <enabledFeatures>
        <feature>ACTION</feature>
        <feature>ANNOTATION</feature>
        <feature>DOCUMENT_SECURITY</feature>
        <feature>EMBEDDED_FILE</feature>
        <feature>FONT</feature>
    </enabledFeatures>
</featuresConfig>
```

## Conclusion

The tests with the Horror Corpus, the Acrobat Engineering files and the PDF 2.0 example files show that between VeraPDF and JHOVE, VeraPDF is the better choice for detecting specific PDF features that are associated with preservation risks. VeraPDF is able to report on a wider range of features, its output is far easier to interpret and process, and it is less likely to "give up" on files. A limitation of VeraPDF is that, unlike JHOVE, it is not capable of validating PDF's lower-level data structures (full PDF validation). Such deviations from the published specifications can result in a whole separate class of preservation risks, but these are outside the scope of this work. VeraPDF's parse status may be used as a rough validity proxy to identify malformed files. Depending on what preservation risks are considered important (which in turn depends on institutional contexts and preservation policies), both VeraPDF and JHOVE should be seen as complementary to each other. 

## Acknowledgements

Thanks are due to Sam Alloing (KB) and Tyler Thorsted (Brigham Young University) for their feedback to an earlier draft of this post, and for various (online) discussions related to this work.

## Further resources

- [Github repo with analysis scripts and raw tool output](https://github.com/KBNLresearch/pdf-characterisation)

- [Open Preservation Foundation format corpus](https://github.com/openpreserve/format-corpus)

- [PDF 2.0 examples](https://github.com/pdf-association/pdf20examples) 

- [PDF Significant Properties Spreadsheet by Tyler Thorsted](https://docs.google.com/spreadsheets/d/1eW7R8yACBciNimr16Z2ptC7fs1FlmZMnzdtG_DHBuD4/edit?usp=sharing)

[^1]: JavaScript detection wasn't possible in [an earlier analysis I did in 2017]({{ BASE_PATH }}/2017/06/01/policy-based-assessment-with-verapdf-a-first-impression), because VeraPDF didn't support actions at that time.

[^2]: For my analyses I had to re-run JHOVE with output set to TEXT on various occasions to make sense of the output.

[^3]: See the KB [File Format Guidelines](https://www.kb.nl/en/file-download/download/public/842) for more details.

[^4]: See ["Identification of PDF preservation risks with Apache Preflight: a first impression"]({{ BASE_PATH }}/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression), ["Identification of PDF preservation risks with Apache Preflight: the sequel"]({{ BASE_PATH }}/2013/07/25/identification-pdf-preservation-risks-sequel) and ["Identification of PDF preservation risks: analysis of Govdocs selected corpus"]({{ BASE_PATH }}/2014/01/27/identification-pdf-preservation-risks-analysis-govdocs-selected-corpus).

[^5]: Several other tools PDF validators exist, but none of these, including JHOVE, have [gained widespread acceptance by industry and other stakeholders](https://www.pdfa.org/wp-content/until2016_uploads/2015/12/iPres2014-CanonicalPDF-submission_20140827.pdf).

[^7]: See e.g. ["A PDF Test-Set for Well-Formedness Validation in JHOVE - The Good, the Bad and the Ugly"](https://zenodo.org/record/1228650) by Lindlar, Tunnat and Wilson.

[^8]: This is explained in VeraPDF's [documentation](https://docs.verapdf.org/cli/config/#features.xml).

[^9]: I initially assumed the *OwnerString* and *UserString* properties might be related to the presence of an owner password, but these are also reported for PDFs with other security-related restrictions, like print and copy passwords.