---
layout: post
title: Automated assessment of JP2 against a technical profile
tags: [jpeg-2000,JP2,jpylyzer,schematron]
comment_id: 57
---

I've already written a number of blog posts on format validation of *JP2* files. Format validation is only a one aspect of a quality assessment workflow. Digitisation guidelines typically impose various constraints on the technical characteristics of preservation and access images. For example, they may state that a preservation master must be losslessly compressed, and that its progression order must be *RPCL*. A *format profile* is a set of such technical constraints. The process that compares the technical characteristics of a file against a format profile is sometimes called [*Policy Driven Validation*][6]. This corresponds to what *JHOVE2* refers to as [*Assessment*][7] (which I think is a better description).

This blog post describes a simple method for doing a rule-based assessment of *JP2* images. It uses [Schematron][8], which is a rule-based validation language, to 'validate' the output of *jpylyzer* against a profile. Before getting into any technical details, let's first have a look at an example of a format profile.

<!-- more -->

## Example format profile

The table below shows the format profile that we'll be using throughout this blog post, which is a typical 'access'-oriented profile using lossy compression. Note that it is provided here for illustrative purposes only!


| **Parameter**                  | **Value**                                                                            |
| ------------------------------ | ------------------------------------------------------------------------------------ |
| File format                    | JP2 (JPEG 2000 Part 1)                                                               |
| Compression type               | Lossy (irreversible 9-7 wavelet filter)                                              |
| Colour transform               | Yes (only for colour images)                                                         |
| Number of decomposition levels | 5                                                                                    |
| Progression order              | RPCL                                                                                 |
| Tile size                      | 1024 x 1024                                                                          |
| Code block size                | 64 x 64                                                                              |
| Precinct size                  | 256 x 256 for 2 highest resolution levels; 128 x 128 for remaining resolution levels |
| Number of quality layers       | 8                                                                                    |
| Target compression ratio       | 20:1                                                                                 |
| Error resilience               | Start-of-packet headers; end-of-packet headers; segmentation symbols                 |
| Grid resolution                | Stored in “Capture Resolution” fields                                                |
| ICC profiles                   | Embedded using “Restricted ICC” method                                               |
| Capture metadata               | Embedded in XML box                                                                  |

## Corresponding properties in *jpylyzer* output

*Jpylyzer* provides information on *all* of the technical characteristics that are listed in the table. You can check this yourself by running *jpylyzer* on any *JP2* file and looking at the resulting output. A few examples:

+ Compression type - value of *transformation* field:

  ```data
  /jpylyzer/properties/contiguousCodestreamBox/cod/transformation
  ```

+ Progression order - value of *order* field:
  
  ```data
  /jpylyzer/properties/contiguousCodestreamBox/cod/order
  ```

+ ICC profiles - value of *meth* field:
  
  ```data
  /jpylyzer/properties/jp2HeaderBox/colourSpecificationBox/meth
  ```

+ Grid resolution - presence of *captureResolutionBox* element:
  
  ```data
  /jpylyzer/properties/jp2HeaderBox/resolutionBox
  ```

## Expressing the profile as a set of assessable rules

In order to assess *jpylyzer*'s output against the profile, we first need to translate the profile to a set of assessable rules. This is where Schematron comes in. Look at parameter 'Compression type' in the table. In the previous section we saw that it corresponds to the *transformation* field in *jpylyzer*'s output. Below is a Schematron rule that asserts if *transformation* has the required value:  

```xml
<s:rule context="/jpylyzer/properties/contiguousCodestreamBox/cod">
<s:assert test="transformation = '9-7 irreversible'">wrong transformation</s:assert>
</s:rule>
```

In words, the rule asserts that the value of *transformation* (which is located in */jpylyzer/properties/contiguousCodestreamBox/*) equals *9-7 irreversible*. If the rule fails, this will result in the error message "wrong transformation".

Both the location (*context*) and the test statement are expressed using [XPath syntax][5], which allows more complex tests as well.

## Check that value doesn't exceed threshold

The following rule checks if the compression ratio doesn't exceed a threshold value (this is actually a bit tricky, as for images that don't contain much information very high compression ratios may be obtained without losing quality):  

```xml
<s:rule context="/jpylyzer/properties">
<s:assert test="compressionRatio &lt; 35">Too much compression</s:assert>
</s:rule>
```

(Note that the character reference "&lt" represents "<", which isn't allowed in *XML*.)

## Check if element exists

The following Schematron rule checks if the *captureResolutionBox* element exists:

```xml
<s:rule context="/jpylyzer/properties/jp2HeaderBox/resolutionBox">
<s:assert test="captureResolutionBox">no capture resolution box</s:assert>
</s:rule>
```

## Outcome depends on values of multiple elements

Here's a more complex rule that checks whether a colour transformation (*multipleComponentTransformation*) was used while creating the image. A colour transformation is only possible for colour images, so in order to make this work for grayscale images as well, the rule must take into account that *multipleComponentTransformation* will be 'no' in that case (*nC* represents the number of image components):  

```xml
<s:rule context="/jpylyzer/properties/contiguousCodestreamBox/cod">
<s:assert test=
    "(multipleComponentTransformation = 'yes') and 
        (../../jp2HeaderBox/imageHeaderBox/nC = '3') 
    or (multipleComponentTransformation = 'no') and 
        (../../jp2HeaderBox/imageHeaderBox/nC = '1')">
    no colour transformation</s:assert>
</s:rule>
```

## Multiple element instances

Our profile states that the precinct size must be 256 x 256 for the  2 highest resolution levels, and 128 x 128 for the remaining ones. These occur as multiple instances of the *precinctSize* and *precinctSizeY* element in *jpylyzer*'s output, which we can handle as follows (note: for 5 decomposition levels we will have 6 resolution levels):  

```xml
<s:rule context="/jpylyzer/properties/contiguousCodestreamBox/cod">
<s:assert test="precinctSizeY[1] = '128'">precinctSizeY doesn't match profile</s:assert>
<s:assert test="precinctSizeY[2] = '128'">precinctSizeY doesn't match profile</s:assert>
<s:assert test="precinctSizeY[3] = '128'">precinctSizeY doesn't match profile</s:assert>
<s:assert test="precinctSizeY[4] = '128'">precinctSizeY doesn't match profile</s:assert>
<s:assert test="precinctSizeY[5] = '256'">precinctSizeY doesn't match profile</s:assert>
<s:assert test="precinctSizeY[6] = '256'">precinctSizeY doesn't match profile</s:assert>
</s:rule>
```

## The full profile as a schema

A sample schema that covers all aspects of the example format profile is available [here][9].

## Assessment of *jpylyzer* output against the schema

For the actual assessment (or validation) of *jpylyzer* output against the schema a couple of options exist. Probably the most widely-used one is the [ISO Schematron reference implementation][10]. Validation using that software involves a number of successive *XSLT* stylesheet transformations. A more accessible (but probaby less performant) alternative is the [*Probatron* command-line executable][2]. Using *Probatron*, asssessment of a *JP2* would typically involve the following two steps:

### 1. Run *jpylyzer*

For example:


```bash
jpylyzer balloon.jp2 > balloon_jp2.xml
```

### 2. Validate *jpylyzer*'s output against the schema

Example:


```bash
java -jar probatron.jar balloon\_jp2.xml profile.sch > balloon_jp2_assessment.xml
```

## Example output

The above procedure produces an *XML* file that contains a *failed assert* element for each test that failed. For example, the output below is generated if the number of layers is wrong:

```xml
<svrl:failed-assert test="layers = '8'" location="/jpylyzer[1]/properties[1]/contiguousCodestreamBox[1]/cod[1]" line="45" col="550">
<svrl:text>wrong number of layers</svrl:text>  
</svrl:failed-assert>
```

## Demo

I created a small [demo][4] that illustrates the assessment procedure. It includes two *JP2* images, the full schema of the example profile of this blog post, and a Windows batch file. For the moment it is located in my personal Github, but the schemas will probably be included in upcoming *jpylyzer* releases. To use the demo, just [download the ZIP file][11], unzip it, open the batch file in a text editor and  follow the instructions at the top of the file.

## Final note

Although this blog post only covers the assessment of *JP2* images using *jpylyzer*, the same procedure can be used for other formats and tools (provided that the tools are capable of producing *XML* output). Second, knowing that a *JP2* is valid and  conforms to a technical profile is certainly important, but it doesn't say anything about the (quality of the) actual image content. So in an operational setting this will often  require additional checks (e.g. a pixel-wise comparison between source and destination images).

## Acknowledgement

Big thanks go out to Adam Retter (The National Archives) for his suggestion to use *Schematron*, [just as I was struggling to make this work in *XSD*][schemaHell]. Adam also shared some of his own *Schematron* schemas with me, which were a starting point for the work presented here.

## Useful links

+ [Schematron][3]  
+ [XPath syntax][5]  
+ [Jpylyzer][1]  
+ [Probatron][2]  
+ [Demo: check if JP2 file matches a technical profile][4]  
+ [Demo (download as ZIP)][11]

## Post script, February 2019

Since this post was originally published, *jpylyzer*'s output format has changed slightly: from version 1.14.0 onward, all output elements have an associated namespace. This means that the Schematron rules must be adapted accordingly. A [set of example Schematron defintions that work with current versions of *jpylyzer* can be found here](https://github.com/KBNLresearch/jprofile/tree/master/jprofile/schemas). They are part of [*jprofile*](https://github.com/KBNLresearch/jprofile), a simple tool that we use at the KB to assess JP2s from external suppliers. The source code of *jprofile* also demonstrates how to do this type of assessment in Python.

[1]: http://jpylyzer.openpreservation.org/
[2]: http://www.probatron.org/probatron4j.html
[3]: http://www.schematron.com/
[4]: https://github.com/bitsgalore/jpylyzerProfileDemo
[5]: http://www.w3schools.com/xpath/xpath_syntax.asp
[6]: http://wiki.opf-labs.org/pages/viewpage.action?pageId=6062098
[7]: https://bitbucket.org/jhove2/main/wiki/Glossary
[8]: http://en.wikipedia.org/wiki/Schematron
[schemaHell]: https://twitter.com/bitsgalore/status/232829808608948224
[9]:https://github.com/bitsgalore/jpylyzerProfileDemo/blob/master/demoAccessLossy.sch
[10]:http://www.schematron.com/implementation.html
[11]:https://github.com/bitsgalore/jpylyzerProfileDemo/zipball/master
<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2012/09/04/automated-assessment-jp2-against-technical-profile/)
