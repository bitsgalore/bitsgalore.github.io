---
layout: post
title: Identification of PDF preservation risks with Apache Preflight&#58; the sequel
tags: [PDF,Apache-Preflight]
comment_id: 51
---

Last winter I started a first attempt at [identifying preservation risks]({{ BASE_PATH }}/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression) in PDF files using the *Apache Preflight* PDF/A validator. This work was later followed up by others in two SPRUCE hackathons in Leeds (see [this blog post](http://www.openplanetsfoundation.org/blogs/2013-03-15-pdf-eh-another-hackathon-tale) by Peter Cliff) and London (described [here](http://wiki.opf-labs.org/display/SPR/PDFA+Validation+tools+give+different+results)). Much of this later work tacitly assumes that *Apache Preflight* is able to successfully identify features in PDF that are a potential risk for long-term access. This [Wiki page on uses and abuses of Preflight](http://wiki.opf-labs.org/display/SPR/PDFBox+Preflight+2++-+Uses+and+Abuses) (created as part of the final SPRUCE hackathon) even goes as far as stating that "*Preflight is thorough and unforgiving (as it should be)*". But what evidence do we have to support such claims? The only evidence that I'm aware of, are the results obtained from a [small test corpus of custom-created PDFs](https://github.com/openplanets/format-corpus/tree/master/pdfCabinetOfHorrors). Each PDF in this corpus was created in such a way that it includes only one specific feature that is a potential preservation risk (e.g. encryption, non-embedded fonts, and so on). However, PDFs that exist 'in the wild' are usually more complex. Also, the PDF specification often allows you to implement similar features in subtly different ways. For these reasons, it is essential to obtain additional evidence of *Preflight*'s ability to detect 'risky' features before relying on this tool in any operational setting.

<!-- more -->

## Adobe Acrobat Engineering test files

Shortly after I completed my initial tests, Adobe released the [Acrobat Engineering website](http://acroeng.adobe.com/wp/), which contains a large volume of test documents that are used by Adobe for testing their products. Although the test documents are not fully annotated, they are subdivided into categories such as *Multimedia & 3D Tests* and *Font tests*. This makes these files particularly useful for additional tests on *Preflight*.

## Methodology

The general methodology I used to analyse these files is identical to what I did in my [2012 report](https://zenodo.org/record/2556637): first, each PDF was validated using *Apache Preflight*. As a control I also validated the PDFs with the *Preflight* component of Adobe Acrobat, using the *PDF/A-1b* profile. The table below lists the software versions used:

|Software|Version|
|:---|:---|
|Apache Preflight|2.0.0|
|Adobe Acrobat|10.14|
|Acrobat Preflight|10.1.3 (090)|

## Re-analysis of PDF Cabinet of Horrors corpus

Because the current analysis is based on a more recent version of *Apache Preflight* than the one used in the 2012 report (which was 1.8.0), I first re-ran the analysis of the PDFs in the [PDF Cabinet of Horrors corpus](https://github.com/openplanets/format-corpus/tree/master/pdfCabinetOfHorrors). The main results are reproduced [here](http://wiki.opf-labs.org/display/TR/Portable+Document+Format). The main differences with respect to that earlier version are:

1. *Apache Preflight* now has an option to produce output in *XML* format (as [suggested by William Palmer](https://issues.apache.org/jira/browse/PDFBOX-1540) following the Leeds SPRUCE hackathon)
2. Better reporting of non-embedded fonts (see also [this issue](https://issues.apache.org/jira/browse/PDFBOX-1449))
3. Unlike the earlier version, *Preflight* 2.0.0 does not give any meaningful output in case of encrypted and password-protected PDFs! This is probably a bug, for which I submitted a report [here](https://issues.apache.org/jira/browse/PDFBOX-1659).

## Analysis Acrobat Engineering PDFs

Since the Acrobat Engineering site hosts a *lot* of PDFs, I only focused on a limited subset for the current analysis:

1. all files in the *General* section of the [Font Testing](http://acroeng.adobe.com/wp/?page_id=101) category;
2. all files in the *Classic Multimedia* section of the [Multimedia & 3D Tests](http://acroeng.adobe.com/wp/?page_id=61) category. 

The results are summarized in two tables (see next sections). For each analysed PDF, the table lists:

* the error(s) reported by Adobe Acrobat Preflight;
* the error code(s) reported by Apache Preflight (see Preflight's [source code](http://svn.apache.org/repos/asf/pdfbox/trunk/preflight/src/main/java/org/apache/pdfbox/preflight/PreflightConstants.java) for a listing of all possible error codes);
* the error description(s) reported by Apache Preflight in the *details* output element.   

For the sake of readability, the tables only list those error messages/codes that are directly related to font problems, multimedia, encryption and JavaScript. The full output for all tested files can be found [here](https://github.com/bitsgalore/apachePreflightAcroEng).

## Fonts

The table below summarizes the results of the PDFs in the [Font Testing](http://acroeng.adobe.com/wp/?page_id=101) category:

|Test file|Acrobat Preflight error(s)|Apache Preflight Error Code(s)|Apache Preflight Details
|:---|:---|:---|:---
|[EmbeddedCmap.pdf](http://acroeng.adobe.com/Test_Files/fonts//EmbeddedCmap.pdf)|Font not embedded (and text rendering mode not 3) ; Glyphs missing in embedded font |3.1.3|Invalid Font definition, FontFile entry is missing from FontDescriptor for HeiseiKakuGo-W5|
|[TEXT.pdf](http://acroeng.adobe.com/Test_Files/fonts//TEXT.pdf)|Font not embedded (and text rendering mode not 3); Glyphs missing in embedded font ; TrueType font has differences to standard encodings but is not a symbolic font; Wrong encoding for non-symbolic TrueType font|3.1.5; 3.1.1; 3.1.2; 3.1.3; 3.2.4|Invalid Font definition, The Encoding is invalid for the NonSymbolic TTF; Invalid Font definition, Some required fields are missing from the Font dictionary; Invalid Font definition, FontDescriptor is null or is a AFM Descriptor; Invalid Font definition, FontFile entry is missing from FontDescriptor for Arial,Italic *(repeated for other fonts)*; Font damaged, The CharProcs references an element which can't be read|
|[Type3_WWW-HTML.PDF](http://acroeng.adobe.com/Test_Files/fonts//Type3_WWW-HTML.PDF)|-|3.1.6|Invalid Font definition, The character with CID"58" should have a width equals to 15.56599 *(repeated for other fonts)*|
|[embedded_fonts.pdf](http://acroeng.adobe.com/Test_Files/fonts//embedded_fonts.pdf)|Font not embedded (and text rendering mode not 3); Type 2 CID font: CIDToGIDMap invalid or missing|3.1.9; 3.1.11|Invalid Font definition; Invalid Font definition, The CIDSet entry is missing for the Composite Subset|
|[embedded_pm65.pdf](http://acroeng.adobe.com/Test_Files/fonts//embedded_pm65.pdf)|-|3.1.6|Invalid Font definition, Width of the character "110" in the font program "HKPLIB+AdobeCorpID-MyriadRg"is inconsistent with the width in the PDF dictionary *(repeated for other font)*|
|[notembedded_pm65.pdf](http://acroeng.adobe.com/Test_Files/fonts//notembedded_pm65.pdf)|Font not embedded (and text rendering mode not 3); Glyphs missing in embedded font|3.1.3|Invalid Font definition, FontFile entry is missing from FontDescriptor for TimesNewRoman *(repeated for other fonts)*|
|[printtestfont_nonopt.pdf](http://acroeng.adobe.com/Test_Files/fonts//printtestfont_nonopt.pdf)*|ICC profile is not valid; ICC profile is version 4.0 or newer; ICC profile uses invalid color space;ICC profile uses invalid type|-|*Preflight throws exception (exceptionThrown), exits with message 'Invalid ICC Profile Data'*|
|[printtestfont_opt.pdf](http://acroeng.adobe.com/Test_Files/fonts//printtestfont_opt.pdf)*|ICC profile is not valid; ICC profile is version 4.0 or newer; ICC profile uses invalid color space; ICC profile uses invalid type|-|*Preflight throws exception (exceptionThrown), exits with message 'Invalid ICC Profile Data'*|
|[substitution_fonts.pdf](http://acroeng.adobe.com/Test_Files/fonts//substitution_fonts.pdf)|Font not embedded (and text rendering mode not 3)|3.1.1; 3.1.2; 3.1.3 |Invalid Font definition, Some required fields are missing from the Font dictionary; Invalid Font definition, FontDescriptor is null or is a AFM Descriptor; Invalid Font definition, FontFile entry is missing from FontDescriptor for Souvenir-Light *(repeated for other fonts)*|
|[text_images_pdf1.2.pdf](http://acroeng.adobe.com/Test_Files/fonts//text_images_pdf1.2.pdf)|Font not embedded (and text rendering mode not 3); Glyphs missing in embedded font; Width information for rendered glyphs is inconsistent|3.1.1; 3.1.2|Invalid Font definition, Some required fields are missing from the Font dictionary; Invalid Font definition, FontDescriptor is null or is a AFM Descriptor|

\* *As this document doesn't appear to have any font-related issues it's unclear why it is in the Font Testing category. Errors related to ICC profiles reproduced here because of relevance to Apache Preflight exception.*

## General observations

An intercomparison between the results of Acrobat Preflight and Apache Preflight shows that Apache Preflight's output may vary in case of non-embedded fonts. In most cases it produces error code 3.1.3 (as was the case with the *PDF Cabinet of Horrors* dataset), but other errors in the 3.1.x range may occur as well. The 3.1.6 "character width" error is something that was also encountered during the [London SPRUCE Hackathon](http://wiki.opf-labs.org/display/SPR/PDFA+Validation+tools+give+different+results), and according to the information [here](https://groups.google.com/forum/#!topic/pdfnet-sdk/L2osfwaap98) this is most likely the result of the PDF/A specification not being particularly clear. So, this looks like a non-serious error that can be safely ignored in most cases.

## Multimedia

The next table shows the results for [Multimedia & 3D Tests](http://acroeng.adobe.com/wp/?page_id=61) category:

|Test file|Acrobat Preflight error(s)|Apache Preflight Error Code(s)|Apache Preflight Details
|:---|:---|:---|:---
|[20020402_CALOS.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//20020402_CALOS.pdf)|-|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[Disney-Flash.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//Disney-Flash.pdf)|Contains action of type JavaScript; Document contains JavaScripts; Document contains additional actions (AA); Font not embedded (and text rendering mode not 3); Form field does not have appearance dict; Form field has actions; Incorrect annotation type used (not allowed in PDF/A); PDF contains EF (embedded file) entry|1.0; 1.2.1|*No multimedia-related errors; Preflight did report syntax and body syntax error*|
|[Jpeg_linked.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//Jpeg_linked.pdf)|Document is encrypted; Encrypt key present in file trailer; Named action with a value other than standard page navigation used; Incorrect annotation type used (not allowed in PDF/A); Font not embedded (and text rendering mode not 3)|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[MultiMedia_Acro6.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//MultiMedia_Acro6.pdf)|Document is encrypted; EmbeddedFiles entry in Names dictionary; Encrypt key present in file trailer; PDF contains EF (embedded file) entry; Incorrect annotation type used (not allowed in PDF/A)|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[MusicalScore.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//MusicalScore.pdf)|CIDset in subset font is incomplete; CIDset in subset font missing; Contains action of type JavaScript; Document contains JavaScripts; Document contains additional actions (AA); Font not embedded (and text rendering mode not 3); Form field has actions; Incorrect annotation type used (not allowed in PDF/A); PDF contains EF (embedded file) entry; Type 2 CID font: CIDToGIDMap invalid or missing|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[SVG-AnnotAnim.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//SVG-AnnotAnim.pdf)|Incorrect annotation type used (not allowed in PDF/A); PDF contains EF (embedded file) entry|5.2.1; 1.2.9|Forbidden field in an annotation definition, The subtype isn't authorized : SVG; Body Syntax error, EmbeddedFile entry is present in a FileSpecification dictionary|
|[SVG.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//SVG.pdf)|Contains action of type JavaScript; Document contains JavaScripts; Font not embedded (and text rendering mode not 3); Form field has actions; PDF contains EF (embedded file) entry|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[ScriptEvents.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//ScriptEvents.pdf)|Contains action of type JavaScript; Document contains JavaScripts; Font not embedded (and text rendering mode not 3); Form field has actions; Incorrect annotation type used (not allowed in PDF/A); PDF contains EF (embedded file) entry|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[Service Form_media.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//Service%20Form_media.pdf)|Contains action of type JavaScript; Contains action of type ResetForm; Document contains JavaScripts; Document contains additional actions (AA); Font not embedded (and text rendering mode not 3); Glyphs missing in embedded font; Incorrect annotation type used (not allowed in PDF/A); Named action with a value other than standard page navigation used; PDF contains EF (embedded file) entry|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[Trophy.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//Trophy.pdf)|Contains action of type JavaScript; Document contains JavaScripts; Font not embedded (and text rendering mode not 3); Form field has actions; Incorrect annotation type used (not allowed in PDF/A); PDF contains EF (embedded file) entry|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[VolvoS40V50-Full.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//VolvoS40V50-Full.pdf)|Preflight exits with: "An error occurred  while parsing a contents stream. Unable to analyze the PDF file"|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[gXsummer2004-stream.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//gXsummer2004-stream.pdf)|File cannot be loaded in Acrobat (*damaged file)*|1.0; 1.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[phlmapbeta7.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//phlmapbeta7.pdf)|Document contains additional actions (AA); Font not embedded (and text rendering mode not 3); Incorrect annotation type used (not allowed in PDF/A); PDF contains EF (embedded file) entry|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[us_population.pdf](http://acroeng.adobe.com/Test_Files/classic_multimedia//us_population.pdf)|Preflight exits with: "An error occurred  while parsing a contents stream. Unable to analyze the PDF file"|1.0; 1.2.1|*No multimedia, font or encryption-related errors; Preflight did report syntax and body syntax error*|
|[movie.pdf](http://acroeng.adobe.com/Test_Files/movie//movie.pdf)|Incorrect annotation type used (not allowed in PDF/A)|5.2.1|Forbidden field in an annotation definition, The subtype isn't authorized : Movie|
|[movie_down1.pdf](http://acroeng.adobe.com/Test_Files/movie//movie_down1.pdf)|Incorrect annotation type used (not allowed in PDF/A)|5.2.1|Forbidden field in an annotation definition, The subtype isn't authorized : Movie|
|[remotemovieurl.pdf](http://acroeng.adobe.com/Test_Files/movie//remotemovieurl.pdf)|Font not embedded (and text rendering mode not 3); Incorrect annotation type used (not allowed in PDF/A)|5.2.1; 3.1.1; 3.1.2; 3.1.3|Forbidden field in an annotation definition, The subtype isn't authorized : Movie; Invalid Font definition, Some required fields are missing from the Font dictionary; Invalid Font definition, FontDescriptor is null or is a AFM Descriptor; Invalid Font definition, FontFile entry is missing from FontDescriptor for Arial|

## General observations

The results from the *Multimedia* PDFs are interesting for several reasons. First of all, these files include a wide variety of 'risky' features, such as multimedia content, embedded files, JavaScript, non-embedded fonts and encryption. These were successfully identified by *Acrobat Preflight* in most cases. *Apache Preflight*, on the other hand, only reported non-specific and fairly uninformative errors (1.0 + 1.2.1) for 12 out of 17 files. Even though *Preflight* was correct in establishing that these files were not valid PDF/A-1b, it wasn't able to drill down to the level of specific features for the majority of these files.

Looking more into detail at those 1.0 and 1.2.1 errors, the detailed description of most of them is:

```data
 Syntax error, Expected pattern 'obj but missed at character 'o'
```

To me it looks like *Preflight* doesn't correctly parse the binary structure of the PDF. Opening a few of the problematic PDFs revealed that the object identifiers in these files were followed *immediately* by the object contents, e.g: 

```data
32 0 obj<</Kids[33 0 R]>>
endobj
```

whereas more commonly they are separated by a line terminator, like this:

```data
32 0 obj
<</Kids[33 0 R]>>
endobj
```

As far as I'm aware neither the PDF specification nor PDF/A have anything to say about line endings in this case, so my best guess is that this is simply a bug that results in the file not being fully parsed. I submitted a bug report for this issue [here](https://issues.apache.org/jira/browse/PDFBOX-1674).

## Summary and conclusions

The re-analysis of the PDF Cabinet of Horrors corpus, and the subsequent analysis of a sub-set of the Adobe Acrobat Engineering PDFs shows a number of things. First, *Apache Preflight* 2.0.0 does not properly identify encryption and password-protection. This looks like a bug that is probably easily fixed. Second, the analysis of the  *Font Testing* PDFs from the Acrobat Engineering site revealed that non-embedded fonts may result in a variety of error codes in *Apache Preflight* (assuming here that the *Acrobat Preflight* results are accurate). So, when using *Apache Preflight* to check font embedding, it's probably a good idea to treat all font-related errors (perhaps with the exception of character width errors) as a potential risk.  The more complex PDFs in the *Multimedia* category proved to be quite challenging to *Apache Preflight*: for most files here, it was not able to identify *specific features* such as multimedia content, embedded files, JavaScript and non-embedded fonts. A cursory analysis of some of the failed files suggests that this is probably a bug that results in *Apache Preflight* not being able to parse the file structure correctly. Keeping in mind that he specificity of *Preflight*'s validation output already improved considerably since version 1.8.0, a fix of both this issue and the encryption problem would probably result in another significant improvement. In the meantime, it's important to keep the expectations about the tool's capabilities realistic, in order to avoid some potential unintended misuses.

## Links

* [Full Acrobat Preflight  and Apache Preflight output for all tested files (Github)](https://github.com/bitsgalore/apachePreflightAcroEng)

* [Portable Document Format on OPF File Format Risk Registry](http://wiki.opf-labs.org/display/TR/Portable+Document+Format)

* [Adobe Acrobat Engineering website](http://acroeng.adobe.com/wp/)

* [Identification of PDF preservation risks with Apache Preflight: a first impression](https://zenodo.org/record/2556637)

* [PDFBox Preflight 2 - Uses and Abuses (OPF Wiki)](http://wiki.opf-labs.org/display/SPR/PDFBox+Preflight+2++-+Uses+and+Abuses)

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/07/25/identification-pdf-preservation-risks-sequel/)
