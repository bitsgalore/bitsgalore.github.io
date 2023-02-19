---
layout: post
title: Identification of PDF preservation risks&#58; analysis of Govdocs selected corpus
tags: [PDF,Apache-Preflight]
comment_id: 47
---

This blog follows up on three earlier posts about detecting  preservation risks in *PDF* files. In  [part 1]({{ BASE_PATH }}/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression) I explored to what extent the [*Preflight*](http://pdfbox.apache.org/cookbook/pdfavalidation.html) component of the [*Apache PDFBox*](http://pdfbox.apache.org/) library can be used to detect specific preservation risks in *PDF* documents. This was followed up by some work during the *SPRUCE* Hackathon in Leeds, which is covered by [this blog post by Peter Cliff](http://www.openplanetsfoundation.org/blogs/2013-03-15-pdf-eh-another-hackathon-tale). Then last summer I did a series of [additional tests]({{ BASE_PATH }}/2013/07/25/identification-pdf-preservation-risks-sequel) using files from the [*Adobe Acrobat Engineering* website](http://acroeng.adobe.com/wp/). The main outcome of this more recent work was that, although showing great promise, *Preflight* was struggling with many more complex *PDF*s. Fast-forward another six months and, thanks to the excellent response of the *Preflight* developers to our bug reports, the most serious of these problems are now largely solved[^1]. So, time to move on to the next step!

<!-- more -->

## Govdocs Selected

Ultimately, the aim of this work is to be able to profile large *PDF* collections for specific preservation risks, or to verify that a *PDF* conforms to an institute-specific policy before ingest. To get a better idea of how that might work in practice, I decided to do some tests with the [*Govdocs Selected*](http://www.openplanetsfoundation.org/blogs/2012-07-26-1-million-21000-reducing-govdocs-significantly) dataset, which is a subset of the [Govdocs1](http://digitalcorpora.org/corpora/files) corpus. As a first step I ran the latest version of *Preflight* on every *PDF* in the corpus (about 15 thousand)[^2]. 

## Validation errors

As I was curious about the most common validation errors (or, more correctly, violations of the *PDF/A-1b* profile), I ran a little post-processing script on the output files to calculate error occurrences. The following table lists the results. For each *Preflight* error (which is represented as an error code), the table shows the number of *PDF*s for which the error was reported (expressed as a percentage)[^3].

|Error code|% PDFs reported|Description (from [Preflight source code](http://svn.apache.org/repos/asf/pdfbox/trunk/preflight/src/main/java/org/apache/pdfbox/preflight/PreflightConstants.java))|
|:---|:---|:---|
|2.4.3|79.5|color space used in the PDF file but the DestOutputProfile is missing|
|7.1|52.5|Invalid metadata found|
|2.4.1|39.1|RGB color space used in the PDF file but the DestOutputProfile isn't RGB|
|1.2.1|38.8|Error on the object delimiters (obj / endobj)|
|1.4.6|34.3|ID in 1st trailer and the last is different|
|1.2.5|32.1|The length of the stream dictionary and the stream length is inconsistent|
|7.11|31.9|PDF/A Identification Schema not found|
|3.1.2|31.6|Some mandatory fields are missing from the FONT Descriptor Dictionary|
|3.1.3|29.4|Error on the "Font File x" in the Font Descriptor *(ed.:font not embedded?)*|
|3.1.1|27.2|Some mandatory fields are missing from the FONT Dictionary|
|3.1.6|17.1|Width array and Font program Width are inconsistent|
|5.2.2|13|The annotation uses a flag which is forbidden|
|2.4.2|12.8|CMYK color space used in the PDF file but the DestOutputProfile isn't CMYK|
|1.2.2|12|Error on the stream delimiters (stream / endstream)|
|1.2.12|9.5|The stream uses a filter which isn't defined in the PDF Reference document|
|1.4.1|9.3|ID is missing from the trailer|
|3.1.11|8.4|The CIDSet entry i mandatory from a subset of composite font|
|1.1|8.3|Header syntax error|
|1.2.7|7.5|The stream uses an invalid filter (The LZW)|
|3.1.5|7.3|Encoding is inconsistent with the Font|
|2.3|6.7|A XObject has an unexpected key defined|
|Exception|6.6|Preflight raised an exception|
|3.1.9|6.1|The CIDToGID is invalid|
|3.1.4|5.7|Charset declaration is missing in a Type 1 Subset|
|7.2|5|Metadata mismatch between PDF Dictionnary and xmp|
|7.3|4.3|Description schema required not embedded|
|2.3.2|4.2|A XObject has an unexpected value for a defined key|
|7.1.1|3.3|Unknown metadata|
|3.3.1|3.1|a glyph is missing|
|1.4.8|2.6|Optional content is forbidden|
|2.2.2|2.4|A XObject SMask value isn't None|
|1.0.14|2.1|An object has an invalid offset|
|1.4.10|1.6|Last %%EOF sequence is followed by data|
|2.2.1|1.6|A Group entry with S = Transparency is used or the S = Null|
|1|1.6|Syntax error|
|5.2.3|1.5|Annotation uses a Color profile which isn't the same than the profile contained by the OutputIntent|
|1.0.6|1.2|The number is out of Range|
|5.3.1|1.1|The AP dictionary of the annotation contains forbidden/invalid entries (only the N entry is authorized)|
|6.2.5|1|An explicitly forbidden action is used in the PDF file|
|1.4.7|1|EmbeddedFile entry is present in the Names dictionary|

This table does look a bit intimidating (but see [this summary of *Preflight* errors](http://wiki.opf-labs.org/display/TR/Summary+of+Apache+Preflight+errors)); nevertheless it is useful to point out a couple of general observations:

* Some errors are *really* common; for instance, error *2.4.3* is reported for nearly 80% of all *PDF*s in the corpus!
* Errors related to color spaces, metadata and fonts are particularly common.
* File structure errors (1.x range) are reported quite a lot as well. Although I haven't looked at this in any detail, I expect that for some files these errors truly reflect a deviation from the *PDF/A-1* profile, whereas in other cases these files may simply not be valid *PDF* (which would be more serious). 
* About 6.5% of all analysed files raised an exception in *Preflight*, which could either mean that something is seriously wrong with them, or alternatively it may point to bugs in *Preflight*.

## Policy-based assessment

Although it's easy to get overwhelmed by the *Preflight* output above, we should keep in mind here that the ultimate aim of this work is *not* to validate against *PDF/A-1*, but to assess arbitrary *PDF*s against a pre-defined technical profile. This profile may reflect an institution's low-level preservation policies on the requirements a *PDF* must meet to be deemed suitable for long-term preservation. In *SCAPE* such low-level policies are called *control policies*, and you can find more information on them [here](http://www.openplanetsfoundation.org/blogs/2013-07-29-scape-creating-machine-understandable-policy-human-readable-policy) and [here](http://www.openplanetsfoundation.org/blogs/2013-09-04-control-policies-scape-project). 

To illustrate this, I'll be using a hypothetical control policy for *PDF* that is defined by the following objectives:

1. File must not be encrypted or password protected
1. Fonts must be embedded and complete
1. File must not contain *JavaScript*
1. File must not contain embedded files (i.e. file attachments)
1. File must not contain multimedia content (audio, video, 3-D objects)
1. File should be valid *PDF*

*Preflight*'s output contains all the information that is needed to establish whether each objective is met (except objective 6, which would need a [full-fledged *PDF* validator](http://duff-johnson.com/2014/01/24/are-your-documents-readable-how-would-you-know/)). By translating the above objectives into a set of [*Schematron*](http://en.wikipedia.org/wiki/Schematron) rules, it is pretty straightforward to assess each *PDF* in our dataset against the control policy. If that sounds familiar: this is the same approach that we used earlier for [assessing *JP2* images against a technical profile]({{ BASE_PATH }}/2012/09/04/automated-assessment-jp2-against-technical-profile). A schema that represents our control policy can be found [here](https://github.com/openplanets/pdfPolicyValidate/blob/master/schemas/pdf_policy_preflight_test.sch). Note that this is only a first attempt, and it may well need some further fine-tuning (more about that later).

## Results of assessment

As a first step I validated all *Preflight* output files against [this schema](https://github.com/openplanets/pdfPolicyValidate/blob/master/schemas/pdf_policy_preflight_test.sch). The result is rather disappointing:

|Outcome|Number of files|%|
|:---|:---|:---|
|Pass|3973|26|
|Fail|11120|74|

So, only 26% of all *PDF*s in *Govdocs Selected* meet the requirements of our control policy! The figure below gives us some further clues as to why this is happening:

![Failed assertions graph]({{ BASE_PATH }}/images/2014/01/failedAssertions_small.png)

Here each bar represents the occurrences of individual failed tests in our [schema](https://github.com/openplanets/pdfPolicyValidate/blob/master/schemas/pdf_policy_preflight_test.sch). 

## Font errors galore

What is clear here is that the majority of failed tests is font-related. The *Schematron* rules that I used for the assessment currently includes *all* font errors that are reported by *Preflight*. Perhaps this is too strict on objective 2 ("*Fonts must be embedded and complete*"). A particular difficulty here is that it is often hard to envisage the impact of particular font errors on the rendering process. On the other hand, the results are consistent with the outcome of a 2013 survey by the [PDF Association](http://www.pdfa.org/), which showed that its members see fonts as the most challenging aspect of *PDF*, both for processing and writing (source: [this presentation](http://duff-johnson.com/wp-content/uploads/2014/01/PDFValidationDreamOrYawn.pdf) by Duff Johnson). So, the assessment results may simply reflect that font problems are widespread[^4]. One should also keep in mind that *Govdocs selected* was created by selecting on unique combinations of file properties from files in [Govdocs1](http://digitalcorpora.org/corpora/files). As a result, one would expect this dataset to be more heterogeneous than most 'typical' *PDF* collections, and this would also influence the results. For instance, the *Creating Program* selection property could result in a relative over-representation of files that were produced by some crappy creation tool. Whether this is really the case could be easily tested by repeating this analysis for other collections.

## Other errors

Only a small small number of *PDF*s with encryption, *JavaScript*, embedded files and multimedia content were detected. I should add here that the occurrence of *JavaScript* is probably underestimated due to a [pending *Preflight* bug](https://issues.apache.org/jira/browse/PDFBOX-1754). A major limitation is that there are currently no reliable tools that are able to test overall conformity to *PDF*. This problem (and a hint at a solution) is also the subject of a recent [blog post by Duff Johnson](http://duff-johnson.com/2014/01/24/are-your-documents-readable-how-would-you-know/). In the current assessment I've taken the occurrence of *Preflight* exceptions (and general processing errors) as an indicator for non-validity. This is a pretty crude approximation, because some of these exceptions may simply indicate a bug in *Preflight* (rather than a faulty *PDF*). One of the next steps will therefore be a more in-depth look at some of the *PDF*s that caused an exception.

## Conclusions

These preliminary results show that policy-based assessment of *PDF* is possible using a combination of *Apache Preflight* and *Schematron*. However, dealing with font issues appears to be a particular challenge. Also, the lack of reliable tools to test for overall conformity to *PDF* (e.g. [ISO 32000](http://acroeng.adobe.com/PDFReference/ISO32000/PDF32000-Adobe.pdf)) is still a major limitation. Another limitation of this analysis is the lack of ground truth, which makes it difficult to assess the accuracy of the results.

## Demo script and data downloads

For those who want to have a go at the analyses that I've presented here, I've created a simple [demo script here](https://github.com/openplanets/pdfPolicyValidate). The raw output data of the *Govdocs selected* corpus can be found [here](https://github.com/openplanets/preflightGovdocsSelected). This includes all *Preflight* files, the *Schematron* output and the error counts. A download link for the *Govdocs selected* corpus can be found at the bottom of [this blog post](http://www.openplanetsfoundation.org/blogs/2012-07-26-1-million-21000-reducing-govdocs-significantly).

## Acknowledgements

*Apache Preflight* developers Eric Leleu, Andreas Lehmkühler and Guillaume Bailleul are thanked for their support and prompt response to my  questions and bug reports.

## Related blog posts

* [Identification of PDF preservation risks with Apache Preflight: a first impression]({{ BASE_PATH }}/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression)
* [Identification of PDF preservation risks: the sequel]({{ BASE_PATH }}/2013/07/25/identification-pdf-preservation-risks-sequel)
* [Are your documents readable? How would you know? (Duff Johnson)](http://duff-johnson.com/2014/01/24/are-your-documents-readable-how-would-you-know/)
* [From 1 Million to 21,000: Reducing Govdocs Significantly (Dave Tarrant)](http://www.openplanetsfoundation.org/blogs/2012-07-26-1-million-21000-reducing-govdocs-significantly)
* [Creating machine understandable policy from human readable policy (Catherine Jones)](http://www.openplanetsfoundation.org/blogs/2013-07-29-scape-creating-machine-understandable-policy-human-readable-policy)
* [Control Policies in the SCAPE Project (Sean Bechhofer)](http://www.openplanetsfoundation.org/blogs/2013-09-04-control-policies-scape-project)
* [PDF on the  OPF File Format Risk Registry](http://wiki.opf-labs.org/display/TR/Portable+Document+Format)

[^1]: This was already suggested by [this re-analysis of the *Acrobat Engineering* files ](http://wiki.opf-labs.org/display/TR/Analysis+of+Acrobat+Engineering+PDFs+with+Acrobat+Preflight+and+Apache+Preflight) that I did in November.
[^2]: This selection was only based on file extension, which introduces the possibility that some of these files aren't really *PDF*s.
[^3]: Errors that were reported for less than 1% of all analysed *PDF*s are not included in the table.
[^4]: In addition to this, it seems that *Preflight* [sometimes fails to detect fonts that are not embedded](https://issues.apache.org/jira/browse/PDFBOX-1864), so the number of *PDF*s with font issues may be even greater than this test suggests.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2014/01/27/identification-pdf-preservation-risks-analysis-govdocs-selected-corpus/)
