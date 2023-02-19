---
layout: post
title: Policy-based assessment of EPUB with Epubcheck
tags: [EPUB,EPUBCheck,schematron]
comment_id: 36
---

Back in 2012 the KB conducted a first [investigation](https://zenodo.org/record/839711) of the suitability of the *EPUB* format for long-term preservation. The KB will soon start receiving publications in this format, and in anticipation of this, our Collection Care department has formulated a policy on the minimum requirements an *EPUB* must meet to ensure long-term accessibility. The policy largely follows the recommendations from the 2012 report. This blog explores to what extent it is possible to automatically assess the *EPUB*s that we receive against our policy using a combination of the [*Epubcheck*](https://github.com/idpf/epubcheck) tool and [*Schematron*](http://www.schematron.com/) rules.

<!-- more -->

## KB *EPUB* policy

The KB's policy on *EPUB*  is made up of the following objectives:

1. File must be valid *EPUB* (either version 2 or 3)

    *Rationale*: this minimises the risk of interoperability problems.

2. File may not contain DRM or encryption

    *Rationale*: this minimises the risk that files become inaccessible. An edge case here is [font obfuscation](http://www.idpf.org/epub/30/spec/epub30-ocf.html#font-obfuscation), which mangles some leading bytes in embedded fonts. This technology is merely meant as a stumbling block to discourage third parties from re-using embedded fonts, and it doesn pose a serious threat to long-term accessibility.

3. File may not contain foreign resources

    *Rationale*: the [Core Media Types](http://www.idpf.org/epub/301/spec/epub-publications.html#sec-core-media-types) define a set of file formats that must be supported by all conforming *EPUB*  readers. [Foreign resources](http://www.idpf.org/epub/301/spec/epub-publications.html#gloss-publication-resource-foreign) are resources that are not part of this set, and the KB's policy is to not accept them. This requirement minimises the risk of accepting files that contain content that may not be rendered correctly by some readers.  

4. File may not contain *DTBook* content

    *Rationale*: *EPUB*  2 offered the option to use the [*DTBook*](http://www.niso.org/workrooms/daisy/Z39-86-2005.html) (*DAISY* Digital Talking Book) format as an alternative to *XHTML* 1.1. Support for *DTBook* was [dropped in *EPUB* 3](http://www.idpf.org/epub/30/spec/epub30-changes.html#sec-removals-dtbook). Support is already limited with current *EPUB* reading software: both the popular [*Calibre*](http://calibre-ebook.com/) and [*Readium*](http://readium.org/) viewers are unable to process *EPUBS* with *DTBook* content (although my [Sony Reader device](https://en.wikipedia.org/wiki/Sony_Reader#2012_Model_.28Discontinued_late_2013.29) handles them without problems). This does not bode well for the future.

## Automated conformance checking

To check if an *EPUB* conforms to the above policy, we need to:

1. test for validity against the format's standard;
2. extract technical information that tells us something about DRM and file resources inside the *EPUB*;
3. assess the results of steps 1 and 2 against our policy.

The [*Epubcheck*](https://github.com/idpf/epubcheck) validator is the obvious candidate for steps 1 and 2. Since *Epubcheck* is capable of reporting its results in XML format, we can use [*Schematron*](http://www.schematron.com/) rules for the final assessment step. The general approach is similar to earlier work on the [JP2]({{ BASE_PATH }}/2012/09/04/automated-assessment-jp2-against-technical-profile/) and [PDF]({{ BASE_PATH }}/2014/01/27/identification-pdf-preservation-risks-analysis-govdocs-selected-corpus/) formats, as well as the British Library's [Flint](https://github.com/openpreserve/flint) tool.  

## Test data

For testing, we first need a corpus of files that are known violate one or more objectives of our policy. As this turned out to be more difficult than expected, I created a small [set of test files](https://github.com/KBNLresearch/epubPolicyTests). Some of the files in this dataset were created from scratch; others were taken directly or adapted from existing openly licensed datasets. The following table lists the main characteristics of the files in the dataset[^1]:

|Test|Epub version|Description|
|:--|:--|:--|
|[Minimal](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_minimal.epub?raw=true)|2|Basic file with one text resource and one image|
|[Encryption](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_minimal_encryption.epub?raw=true)|2|Fake encrypted file that includes *encryption.xml* resource in `META-INF`, indicating that main text resource is encrypted[^2]|
|[Font obfuscation](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub30_font_obfuscation.epub?raw=true)|3|Includes fonts that are obfuscated (which results in *hasEncryption* in epubcheck). Taken from [EPUB 3 Sample Documents](https://code.google.com/p/epub-samples/) ([*wasteland with OTF fonts, obfuscated*](https://code.google.com/p/epub-samples/downloads/detail?name=wasteland-otf-obf-20120118.epub&can=2&q=)).|
|[Foreign resource without fallback](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_foreign_resource_no_fallback.epub?raw=true)|2|Includes JP2 image, which is a format that is not on the list of Core Media Types|
|[Foreign resource with fallback 1](build/epub20_foreign_resource_with_fallback.epub?raw=true)|2|Includes JP2 image, which is a format that is not on the list of Core Media Types; fallback defined in manifest, identifier in content document|
|[Foreign resource with fallback 2](build/epub20_foreign_resource_with_fallback_noID.epub?raw=true)|2|Includes JP2 image, which is a format that is not on the list of Core Media Types; fallback defined in manifest, no identifier in content document|
|[DTBook](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_dtbook.epub?raw=true)|2|Includes Digital Talking Book content. Adapted from [threepress](https://code.google.com/p/threepress/source/browse/branches/bookworm-caching/library/test-data/data/hauy.epub?r=583), published under [BSD 3](http://opensource.org/licenses/BSD-3-Clause) license.|

Apart from the above files, the dataset also includes:

* the [full source](https://github.com/KBNLresearch/epubPolicyTests/tree/master/content) of each test file (as a directory structure);
* a [bash script](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build.sh) that automatically builds (zips) all directories to *EPUB*s;
* [another bash script](https://github.com/KBNLresearch/epubPolicyTests/blob/master/analyse.sh) that analyses all *EPUB*s with version 3 and 4 of *Epubcheck*;
* the [full *Epubcheck* output](https://github.com/KBNLresearch/epubPolicyTests/tree/master/epubcheckout) of each file.

All files are openly licensed, and by adapting the existing tests it is pretty straightforward to add new ones.

## Analysis with Epubcheck

The first question that we need to answer here is whether *Epubcheck*'s output is sufficiently detailed for our needs. So, as a first step I analysed all files in the dataset with [*Epubcheck*](https://github.com/idpf/epubcheck). I did this using both [*Epubcheck* 3.0.1](https://github.com/IDPF/epubcheck/releases/tag/v3.0.1) (the current stable version) and [the alpha 11 release of *Epubcheck* 4.0.0](https://github.com/IDPF/epubcheck/releases/tag/v4.0.0-alpha11). The full output can be found [here](https://github.com/KBNLresearch/epubPolicyTests/tree/master/epubcheckout). In the following sections I will address each of the objectives of the KB policy.

## Encryption objective

For the 'fake' encrypted file *Epubcheck*'s output contains a *hasEncryption* property. Moreover, the *messages* element in the output contains an error message that refers to the encrypted resource. In *Epubcheck* 3 this is:

```data
ERROR: : OPS/XHTML file OEBPS/Text/pdfMigration.html cannot be decrypted
```

A double-check with a 'real' encrypted *EPUB* (which is proprietary and could not be included in the dataset) confirmed that each encrypted resource produces an error message of the general form:

```data
ERROR: : $fileType file $fileName cannot be decrypted
```

Here, `$fileType` and `$fileName` refer to the file type and name of the affected resource. The 'fake' encrypted file also resulted in some additional error messages about undefined fragment identifiers, but these look like secondary errors that result from *Epubcheck* 's inability to decrypt the encrypted resource.

The behaviour of *Epubcheck* 4 is similar, although the error message is slightly different:

```data
RSC-004, ERROR, [File 'OEBPS/Text/pdfMigration.html' could not be decrypted.],epub20_minimal_encryption.epub
```

The file with the obfuscated fonts also results in a *hasEncryption* entry in *Epubcheck*'s output. *Epubcheck* (both versions 3 and 4) doesn't provide any direct clue that the encryption in this file is limited to some obfuscated fonts. For our policy-based assessment we can therefore ignore the *hasEncryption* entry, and simply check for the presence of "cannot be decrypted" error messages (see above).

## DTBook objective

*Epubcheck*'s output does not give any explicit clue to the presence of *DTBook* content. However, *Epubcheck* 3 does report a read error on the corresponding file resource:

```data 
ERROR: : I/O error reading OEBPS/hauy-2005-1.xml: Stream closed
```

*Epubcheck* 4 does not report this error. A check of the *DTBook* resource confirmed that it is valid against [version 2 of the *DTBook* Document Type Definition](http://www.daisy.org/z3986/2005/dtbook-2005-2.dtd) (I checked this using both [JHOVE](http://jhove.sourceforge.net/) and an [online XML validator](http://www.validome.org/xml/validate/)). This suggests that *Epubcheck* 3 doesn't properly recognise (cannot parse?) *DTBook* content, and incorrectly flags *EPUB*s that hold this as "Not well-formed". The behaviour of *Epubcheck* 4 is correct (see also [this issue report](https://github.com/IDPF/epubcheck/issues/518)). 

## Foreign resources objective

The test dataset contains 3 files with foreign resources (resources that are not on the list of Core Media Types). In the first one I simply replaced a *PNG* image by a *JP2* (and updated the manifest and the reference in the text accordingly). This results in the following validation error (*Epubcheck* 3): 

```data
ERROR: /OEBPS/Text/pdfMigration.html(20): non-standard image resource 'OEBPS/Images/pdfVenn.jp2' of type 'image/jp2'
```

And in *Epubcheck* 4;

```data
MED-003, ERROR, [Non-standard image resource of type image/jp2 found.], OEBPS/Text/pdfMigration.html (20-63) 
```

This error also causes the validation to fail. The *EPUB* specification allows the use of foreign resources, but [only if they have a Core Media fallback](http://www.idpf.org/epub/20/spec/OPF_2.0.1_draft.htm#Section2.3.1.1). I created two additional test files that use the original *PNG* image as a fallback[^3]; I then updated the manifest of these files accordingly. *Epubcheck* validates both files as "Well-formed", but gives no information whatsoever on the presence of foreign resources. Somewhat alarmingly, both *Calibre* and *Readium* failed to read [either](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_foreign_resource_with_fallback.epub?raw=true) of these [files](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_foreign_resource_with_fallback_noID.epub?raw=true) correctly: the (fall back) image was not shown in both cases. As it turns out, [very few *EPUB* readers support manifest fallbacks](https://github.com/IDPF/epubcheck/issues/511), even though this feature has been part of the *EPUB* specification for a long time (at least since *EPUB* 2).  

## Translating the policy to Schematron rules

If *Epubcheck* were able to address all apects of the KB's policy, it would be possible to translate each of its objectives into a *Schematron* rule. As *Epubcheck* doesn't yet provide the required information on foreign resources and *DTBook* content, for now we can only do this for the validity and encryption objectives. The *Schematron* rule for validity is:

```xml
<s:pattern name="wellFormed">
    <s:rule context="/jh:jhove/jh:repInfo">
    <s:assert test="(jh:status = 'Well-formed')">Not well-formed epub</s:assert>
    </s:rule>
</s:pattern>
```

For the encryption objective we have this:

```xml
<!-- This rule rules out encrypted content, but permits font obfuscation-->  
<s:pattern name="encryptedResources">
    <s:rule context="/jh:jhove/jh:repInfo/jh:messages">
    <s:assert test="count(jh:message[contains(.,'cannot be decrypted')]) = 0">Contains encrypted resources</s:assert>
    </s:rule>
</s:pattern>
```

Alternatively, we could have created a rule that uses the *hasEncryption* property here, but that would cause any files with obfuscated fonts to fail the assessment. The corresponding schema (adapted from the BL's [*Flint*](https://github.com/openpreserve/flint) tool) can be found [here](https://github.com/KBNLresearch/epubPolicyValidate/blob/master/schemas/kbPolicy.sch). It is designed to work with *Epubcheck* 3 only [^4].

## Demo

I created a simple [*EPUB* policy-based validation demo](https://github.com/KBNLresearch/epubPolicyValidate). It is a shell script that validates all *EPUB* files in a user-defined directory with *Epubcheck*, and subsequently assesses *Epubcheck*'s output against a user-defined schema. Note that the purpose of the script is just to demonstrate the general procedure; it is not recommended for operational use.

## Possible Epubcheck enhancements

The above tests demonstrate that currently *Epubcheck* is able to cover two aspects of the KB's policy on *EPUB*: validity and encryption. However, its output doesn't provide the information we need on the presence of foreign resources and *DTBook* content. It would be useful if *Epubcheck* could be extended with an option that reports all resources in an *EPUB* with their corresponding media types. This information can be extracted from the *manifest* element of an *EPUB*'s [Package Document](http://www.idpf.org/epub/301/spec/epub-publications.html#sec-package-documents). For a file with both *DTBook* content and a foreign resource (a *JP2* file) this looks something like this:

```xml
<manifest>
    <item href="toc.ncx" id="ncx" media-type="application/x-dtbncx+xml" />
    <item href="Images/pdfVenn.jp2" id="pdfVennJP2" media-type="image/jp2" fallback="pdfVennPNG" />
    <item href="Images/pdfVenn.png" id="pdfVennPNG" media-type="image/png" />
    <item href="hauy-2005-1.xml" id="opf3" media-type="application/x-dtbook+xml" />
</manifest>
```

Some simple *Schematron* rules on the *media-type* attribute would make it possible to filter this for the presence of *DTBook* content (where *media-type* is `application/x-dtbook+xml`) or foreign resources (which have a *media-type* value that is not on the Core Media Types list). Another solution would be to add properties like `hasDTBook` and `hasForeignResources` to *Epubcheck*'s output. This solution is less generic, but possibly more user-friendly.

Finally, the presence of *DTBook* resources [^5] incorrectly causes the validation to fail in *Epubcheck* 3; this has been fixed in *Epubcheck* 4.

## Links

* [Epubcheck](https://github.com/idpf/epubcheck)
* [EPUB KB policy testing dataset](https://github.com/KBNLresearch/epubPolicyTests), includes full source, build and analysis scripts.
* [Policy-based validation demo based on Epubcheck](https://github.com/KBNLresearch/epubPolicyValidate)
* [EPUB 3 Support Grid - comparison of support of EPUB 3.0 features by different  reading systems](http://epubtest.org/compare/)

[^1]: These files are all released under the Creative Commons 3.0 BY-SA license, unless stated otherwise.
[^2]: The 'encryption' in this file is actually fake: I merely replace the original text resource with a [base64](http://linux.die.net/man/1/base64) encoded representation of that file. 
[^3]: They only differ in the way the JP2 image is referenced in the text, as the *EPUB* specification is not completely clear on this.
[^4]: Doesn't yet work with *Epubcheck* 4 because it uses slightly different output messages (could be easily adapted).
[^5]: These are, by the way, pretty rare.

<hr>
Originally published at the [KB Research blog](http://blog.kbresearch.nl/2015/03/13/policy-based-assessment-of-epub-with-epubcheck/)
