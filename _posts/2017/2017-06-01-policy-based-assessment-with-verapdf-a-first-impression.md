---
layout: post
title: Policy-based assessment with VeraPDF - a first impression
tags: [PDF,VeraPDF,schematron]
comment_id: 23
---

Some four years ago I wrote [a blog post]({{ BASE_PATH }}/2013/07/25/identification-pdf-preservation-risks-sequel) that demonstrated how *Apache Preflight* (the PDF/A validator tool that is part of [*Apache PDFBox*](https://pdfbox.apache.org/)) can be used to detect features in a PDF that are potential preservation risks. A [follow-up blog]({{ BASE_PATH }}//2014/01/27/identification-pdf-preservation-risks-analysis-govdocs-selected-corpus) applied [*Schematron*](https://en.wikipedia.org/wiki/Schematron) rules to the *Preflight* output in an attempt at doing policy-based assessments. The results of that work were quite promising, but dealing with Preflight's multitude of (especially font-related) validation errors proved to be a challenge.

The idea of using a *PDF/A* validor for policy-based assessments of "regular" *PDF* files (i.e. *PDF*s that are not necessarily *PDF/A*) was explicitly addressed as a use case for [*veraPDF*](http://verapdf.org/). With *VeraPDF* now having entered its "final testing phase", I thought this was a good time for a small test-drive of *veraPDF*'s capabilities in this area. All test results are based on *VeraPDF* 1.4.7.

<!-- more -->

## Test data
  
For this test I used *PDF*s from the [*Adobe Acrobat Engineering* website](https://web.archive.org/web/20130503115947/http://acroeng.adobe.com/wp/) (sadly gone since 2015). As in my 2013 blog post, I limited the analysis to:

* all files in the *General* section of the [*Font Testing*](https://web.archive.org/web/20150228065249/http://acroeng.adobe.com:80/wp/?page_id=101) category;
* all files in the *Classic Multimedia* section of the [*Multimedia & 3D Tests*](https://web.archive.org/web/20150228104639/http://acroeng.adobe.com:80/wp/?page_id=61) category.

The dataset is quite small, but contains many complex and otherwise challenging *PDF*s, which make it an interesting dataset for testing. 
 
## Policy

The policy is similar to the one used in [my 2014 blog post]({{ BASE_PATH }}//2014/01/27/identification-pdf-preservation-risks-analysis-govdocs-selected-corpus), and it is defined by the following objectives:

1. No encryption / password protection
2. All fonts are embedded
3. No embedded files
4. No file attachments
5. No multimedia content (audio, video, 3-D objects)
6. No PDFs that raise an exception or result in a processing error in *VeraPDF* (*PDF* validity proxy) 

(Note that the 2014 blog post also mentioned the absence of *JavaScript* as an additional objective. However, it turned out that the necessary output for this is not currently reported by *VeraPDF*.) 

Subsequently I 'translated' each of these objectives into *Schematron* rules. For a basic *how-to* see the  [*veraPDF Policy Checking* documentation](http://docs.verapdf.org/policy/). 

The full *Schematron* file can be found [here](https://github.com/KBNLresearch/pdfPolicyVeraPDF/blob/master/schemas/demo-policy.sch).

## VeraPDF configuration

It is important to note that, unlike in my earlier *Apache Preflight* experiments, the *Schematron* rules do not rely on the *PDF/A* validation output! Instead, *VeraPDF* can be instructed to include a 'features report' in its output, which directly points to technical features such as font properties, annotation types, security features, and so on. Most of the features that are needed for a policy-based assessment are disabled by default. So, we first need to activate these in the configuration (file *features.xml* in *VeraPDF*'s *config* directory). I edited it as below:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<featuresConfig>
    <enabledFeatures>
        <feature>ANNOTATION</feature>
        <feature>DOCUMENT_SECURITY</feature>
        <feature>EMBEDDED_FILE</feature>
        <feature>FONT</feature>
        <feature>INFORMATION_DICTIONARY</feature>
    </enabledFeatures>
</featuresConfig>
```

## Basic operation

Supposing that the *PDF*s we want to analyze are in directory `~/myPdfs`, and that the *Schematron* rules that represent our policy are in the file `demo-policy.sch`, we can do a policy-based validation of all these files with one single command:

```bash
verapdf -x --policyfile demo-policy.sch ~/myPdfs/* > myPdfsOut.xml
```

Here the `-x` switch activates feature extraction. The output file `myPdfsOut.xml` contains, for each *PDF*, an element with *PDF/A* validation output, an element with the features report, and an element with the policy report.

## Analysis script

Typically the *VeraPDF* output is rather unwieldy. To facilitate things I wrote a [custom analysis script](https://github.com/KBNLresearch/pdfPolicyVeraPDF), which does the following things:

1. It runs *VeraPDF*
2. It creates a [trimmed-down version of the output file](https://github.com/KBNLresearch/pdfPolicyVeraPDF/blob/master/examples/fonts_san.xml) that only contains the policy report. Also, for each PDF, it removes duplicate instances of failed (policy) checks (e.g. if a check on font embedding fails for 10 different fonts, only one reference to the failed check is kept)
3. It creates a [comma-delimited summary file](https://github.com/KBNLresearch/pdfPolicyVeraPDF/blob/master/examples/fonts_summary.csv) which lists for each PDF its path/name, followed by the description of each unique failed validation rule (taken from the *message* element in *VeraPDF*'s output).

## Running the analysis

For this analysis I ran the above script for both the *fonts* and *multimedia* files, using the following command line (here for the *fonts* files):

```bash
~/pdfPolicyVeraPDF/policyValidate.sh /home/johan/pdfAcrobatEngineering/fonts /home/johan/pdfPolicyVeraPDF/schemas/demo-policy.sch fonts
```

## Results, fonts category

The following table lists, for each *PDF* in the *fonts* category, the corresponding (unique) validation errors (taken from the summary CSV file). Note that the text strings in the right column correspond to text values in the *assert* elements of the policy file.

|Test file|Failed assert(s)|
|:--|:--|
|EmbeddedCmap.pdf|Font is not embedded|
|embedded_fonts.pdf|Font is not embedded|
|embedded_pm65.pdf||
|notembedded_pm65.pdf|Font is not embedded|
|printtestfont_nonopt.pdf||
|printtestfont_opt.pdf||
|substitution_fonts.pdf|Font is not embedded|
|text_images_pdf1.2.pdf|Font is not embedded|
|TEXT.pdf|Font is not embedded|
|Type3_WWW-HTML.PDF|Font is not embedded|

These results show that most of the *PDF*s fail our policy on the font embedding objective.

## Results, multimedia category

Similarly, below are the results for the *multimedia* category:

|Test file|Failed assert(s)|
|:--|:--|
|20020402_CALOS.pdf|Font is not embedded;Movie annotation|
|3-D_PDF.pdf|3D annotation|
|AdobeChassisDemo-commented.pdf|3D annotation|
|AdobeChassisDemo-commented_Review.pdf|3D annotation|
|AVI+Transitions Demo.pdf|Document not parsable|
|Binder_6-3DPages.pdf|3D annotation|
|Disney-Flash.pdf|Font is not embedded;Screen annotation|
|drape_raster_contour_sample.pdf|Font is not embedded;3D annotation|
|gXsummer2004-stream.pdf|Document not parsable|
|Jpeg_linked.pdf|Encrypted document;Document not parsable|
|LabelExample.pdf|Encrypted document;Document not parsable|
|movie_down1.pdf|Movie annotation|
|movie.pdf|Movie annotation|
|MultiMedia_Acro6.pdf|Encrypted document;Document not parsable|
|MusicalScore.pdf|Font is not embedded;Screen annotation|
|phlmapbeta7.pdf|Font is not embedded;Screen annotation|
|remotemovieurl.pdf|Font is not embedded;Movie annotation|
|ScriptEvents.pdf|Font is not embedded;Screen annotation|
|Service Form_media.pdf|Font is not embedded;Screen annotation|
|SVG-AnnotAnim.pdf|Font is not embedded|
|SVG.pdf|Font is not embedded|
|Trophy.pdf|Font is not embedded;Screen annotation|
|us_population.pdf||

Here the reasons for failing the policy are more diverse. Many of these *PDF*s contain *Screen*, *Movie* or *3D* annotations. Non-embedded fonts are common as well. Three *PDF*s were not parsable because of encryption. This turns out to be a [a bug](https://github.com/veraPDF/veraPDF-apps/issues/202) that is fixed in newer versions of *VeraPDF*. Two files (*AVI+Transitions Demo.pdf* and *gXsummer2004-stream.pdf*) were not parsable at all. These files could not be opened in Adobe Acrobat either. Finally, one 49 MB file (which is not listed in the table) resulted in an out-of-memory error that crashed *VeraPDF* altogether. I [reported this as a bug](https://github.com/veraPDF/veraPDF-apps/issues/195).

## General observations

First of all I was impressed with the amount of detailed information that *VeraPDF* can provide of a *PDF* file. I was also pleasantly surprised at the relative ease of doing policy-based assessments. This is mainly thanks to *VeraPDF*'s features report, which allows one to address features such as specific annotation types directly. During my earlier attempts at policy-based assessment with *Apache Preflight*, the detection of non-embedded fonts was particularly difficult (have a look at the [Schematron file](https://github.com/openpreserve/pdfPolicyValidate/blob/master/schemas/pdf_policy_preflight_test.sch#L55) to see what I mean). With *VeraPDF* this only needs [one single line](https://github.com/KBNLresearch/pdfPolicyVeraPDF/blob/master/schemas/demo-policy.sch#L42) (though admittedly this probably means that errors related to damaged or malformed fonts won't be reported).
Thanks to *VeraPDF*'s built-in functionality to do the Schematron validation, it is no longer necessary to use an external Schematron validator (though this is still possible).

## Actions missing in action?

One thing I missed is the reporting of *Actions*. Without this, it is not possible to identify *PDF*s that contain *JavaScript* (and some other features as well). An option to include *Actions* in the 'Feature Report' would make a welcome addition. As the *PDF/A* validation profiles already include checks on *Actions*, this is probably pretty straightforward (see also [this issue](https://github.com/veraPDF/veraPDF-apps/issues/174)).

## Writing a policy file
   
Not having worked on *PDF*-related things for a while myself, it took me some time to figure out how to put together the (Schematron) policy file. The *VeraPDF* [documentation gives some guidance](http://docs.verapdf.org/policy/), but I couldn't find an exhaustive description of every possible feature  in the features report. This meant I first had to run *VeraPDF* (with feature extraction enabled) on a number of files that I *knew* to contain certain features I wanted to include in my policy (e.g. embedded fonts, multimedia), inspect the *XML* output, and then write my Schematron rules based on that output. As I have a pretty good knowledge of the specific *PDF* data structures involved I was able to do this, but it did make me wonder about users who don't have that technical knowledge. Possible solutions would be:

* Additional documentation of all possible output elements in the features report. This [seems to be in the works already](http://docs.verapdf.org/cli/feature-extraction/) (though not complete yet)
* Inclusion of some example policy files. Actually *veraPDF*'s Github repo [contains a number of these](https://github.com/veraPDF/veraPDF-policy-docs/tree/master/Schemas) already, but they are not (yet) referenced by the documentation, and I only found out about them after I ran my tests.

It would also help if users of *veraPDF* would publish and share their policy files.

Finally it just occurred to me this is a good occasion to give one more bump to [this 2009 report I wrote on long-term preservation risks of PDF](https://doi.org/10.5281/zenodo.801661). It explicitly lists the data structures (e.g. annotations, actions) that are associated with specific (risky) features, which might provide users some guidance as to what features are potentially interesting for inclusion in a policy. 

## Links

* [PDF policy-based validation demo, veraPDF](https://github.com/KBNLresearch/pdfPolicyVeraPDF) - Github repo with scripts, Schematron policy file and all output files
* [VeraPDF](http://verapdf.org/)
* [Adobe Portable Document Format - Inventory of long-term preservation risks](https://doi.org/10.5281/zenodo.801661)


<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2017/06/01/policy-based-assessment-with-verapdf-a-first-impression/)
