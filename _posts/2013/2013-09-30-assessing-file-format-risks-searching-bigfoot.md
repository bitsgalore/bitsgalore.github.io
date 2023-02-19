---
layout: post
title: Assessing file format risks&#58; searching for Bigfoot?
tags: [preservation-risks]
comment_id: 49
---

Last week someone pointed my attention to a recent *iPres* paper by Roman Graf and Sergiu Gordea titled "[A Risk Analysis of File Formats for Preservation Planning](http://purl.pt/24107/1/iPres2013_PDF/A%20Risk%20Analysis%20of%20File%20Formats%20for%20Preservation%20Planning.pdf)". The authors propose a methodology for assessing preservation risks for file formats using information in publicly available information sources. In short, their approach involves two stages:

1. Collect and aggregate information on file formats from data sources such as [PRONOM](http://www.nationalarchives.gov.uk/PRONOM), [Freebase](http://www.freebase.com/) and [DBPedia](http://dbpedia.org)

2. Use this information to compute scores for a number of pre-defined risk factors (e.g. the number of software applications that support the format, the format's complexity, its popularity, and so on). A weighted average of these individual scores then gives an overall risk score.

This has resulted in the "File Format Metadata Aggregator" (*FFMA*), which is an expert system aimed at establishing a "*well structured knowledge base with defined rules and scored metrics that is intended to provide decision making support for preservation experts*".

<!-- more -->

The paper caught my attention for two reasons: first, a number of years ago some colleagues at the *KB* developed a [method for evaluating file formats](http://www.kb.nl/hrd/dd/dd_links_en_publicaties/publicaties/KB_file_format_evaluation_method_27022008.pdf) that is based on a similar way of looking at preservation risks. Second, just a few weeks ago I found out that the University of North Carolina is also working on [a method for assessing "File Format Endangerment"](http://www.ils.unc.edu/digccurr/ct_poster/Ryan.pdf) which seems to be following a similar approach. Now let me start by saying that I'm extremely uneasy about assessing preservation risks in this way. To a large extent this is based on experiences with the *KB*-developed method, which is similar to the assessment method behind *FFMA*. I will use the remainder of this blog post to explain my reservations.

## Criteria are largely theoretical

*FFMA* implicitly assumes that it is possible to assess format-specific preservation risks by evaluating formats against a list of pre-defined criteria. In this regard it is similar to (and builds on) the logic behind, to name but two examples, [Library of Congress' Sustainability Factors](http://www.digitalpreservation.gov/formats/sustain/sustain.shtml) and [UK National Archives' format selection criteria](http://www.nationalarchives.gov.uk/documents/selecting-file-formats.pdf). However, these criteria are largely based on theoretical considerations, without being backed up by any empirical data. As a result, their predictive value is largely unknown.

## Appropriateness of measures

Even if we agree that criteria such as software support and the existence of migration paths to some alternative format are important, how exactly do we measure this? It is pretty straightforward to simply count the number of supporting software products or migration paths, but this says nothing about their *quality* or suitability for a specific task. For example, *PDF* is supported by a plethora of software tools, yet it is well known that few of them support *every* feature of the format (possibly even none, with the exception of Adobe's implementation). Here's another example: quite a few (open-source)  software tools support the *JP2* format, but for this many of them (including *ImageMagick* and *GraphicsMagick*) rely on [*JasPer*](http://www.ece.uvic.ca/~frodo/jasper/), a JPEG 2000 library that is notorious for its poor performance and stability. So even if a format is supported by lots of tools, this will be of little use if the quality of those tool are poor.

## Risk model and weighting of scores

Just as the employed criteria are largely theoretical, so is the computation of the risk scores, the weights that are assigned to each risk factor, and they way the individual scores are aggregated into an overall score. The latter is computed as the weighted sum of all individual scores, which means that a poor score on, for example, *Software Count* can be compensated by a high score on other factors. This doesn't strike me as very realistic, and it is also at odds with e.g. [David Rosenthal's view](http://blog.dshr.org/2009/01/are-format-specifications-important-for.html) of formats with open source renderers being immune from format obsolescence.

## Accuracy of underlying data

A cursory look at the [web service implementation of *FFMA*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/) revealed some results that make me wonder about the data that are used for the risk assessment. According to *FFMA*:

* [*PNG*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=png&configName=&classificationName=), [*JPG*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=jpg&configName=&classificationName=) and [GIF](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=gif&configName=&classificationName=) are uncompressed formats (they're not!);
* [*PDF*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=pdf&configName=&classificationName=) is *not* a compressed format (in reality text in *PDF* nearly always uses [Flate compression](http://en.wikipedia.org/wiki/DEFLATE), whereas a [whole array of compression methods](http://www.prepressure.com/pdf/basics/compression) may be used for images);
* [*JP2*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=jp2&configName=&classificationName=) is not supported by *any* software (Software Count=0!), it doesn't have a *MIME* type, it is frequently used, and it is supported by web browsers (all wrong, although arguably *some* browser support exists if you account for external plugins);
* [*JPX*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=jpf&configName=&classificationName=) is *not* a compressed format and it is less complex than *JP2* (in reality it is an extension of *JP2* with added complexity).

To some extent this may also explain the peculiar ranking of formats in Figure 6 of the paper, which marks down *PDF* and *MS Word* (!) as formats with a lower risk than *TIFF* (*GIF* has the overall lowest score).

## What risks?

It is important that the concept of 'preservation risk' as addressed by *FFMA* is closely related to (and has its origins in) the idea of formats becoming obsolete over time. This idea is controversial, and the authors do acknowledge this by defining preservation risks in terms of the "*additional effort required to render a file beyond the capability of a regular PC setup in [a] particular institution*". However, in its current form *FFMA* only provides generalized information about formats, without addressing specific risks *within* formats. A good example of this is *PDF*, which may contain various features that are [problematic]({{ BASE_PATH }}/2012/07/26/pdf-inventory-long-term-preservation-risks) for long-term preservation. Also note how *PDF* is marked as a [*low-risk* format](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=pdf&configName=&classificationName=), despite the fact that it can be a container for *JP2* which is considered [*high-risk*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/rest/loddataanalysis/html/riskscorereport?name=jp2&configName=&classificationName=). So doesn't that imply that  a *PDF* that contains *JPEG 2000* compressed images is at a higher risk?

## Encyclopedia replacing expertise?

A possible response to the objections above would be to refine *FFMA*: adjust the criteria, modify the way the individual risk scores are computed, tweak the weights, change the way the overall score is computed from the individual scores, and improve the underlying data. Even though I'm sure this could lead to some improvement, I'm eerily reminded here of [this recent <strike>rant</strike> blog post](http://www.openplanetsfoundation.org/blogs/2013-09-13-registries-we-need) by Andy Jackson, in which he shares his concerns about the archival community's preoccupation with format, software, and hardware registries. Apart from the question whether the existing registries are actually helpful in solving real-world problems, Jackson suggests that:

> Maybe we don't know what information we need?

and:

> Maybe we don't even know who or what we are building registries for?

He also wonders:

> Are we trying to replace imagination and expertise with an encyclopedia?

I think these comments apply equally well to the recurring attempts at reducing format-specific preservation risks to numerical risk factors, scores and indices. This approach simply doesn't do justice to the subtleties of practical digital preservation. Worse still, I see a potential danger of non-experts taking the results from such expert systems at face value, which can easily lead to ill-judged decisions. Here's an example.

## KB example

About five years some colleagues at the *KB* developed a "quantifiable file format risk assessment method", which is described in [this report](http://www.kb.nl/hrd/dd/dd_links_en_publicaties/publicaties/KB_file_format_evaluation_method_27022008.pdf). This method was [applied](http://www.kb.nl/hrd/dd/dd_links_en_publicaties/publicaties/Alternative_File_Formats_for_Storing_Masters_2_1.pdf) to decide which still image format was the best candidate to replace the then-current format for digitisation masters. The outcome of this was used to justify a change from uncompressed *TIFF* to *JP2*. It was only much later that we found out about a host of practical and standard-related problems with the format, some of which are discussed [here](http://jpeg2000wellcomelibrary.blogspot.nl/2010/12/guest-post-ensuring-suitability-of-jpeg.html) and [here](http://www.dlib.org/dlib/may11/vanderknijff/05vanderknijff.html). *None* of these problems were accounted for by the earlier risk assessment method (and I have a hard time seeing how they ever could be)! The risk factor approach of *GGMA* is covering similar ground, and this adds to my scepticism about addressing preservation risks in this manner.

## Final thoughts

Taking into account the problems mentioned in this blog post, I have a hard time seeing how scoring models such as the one used by *FFMA* would help in solving practical digital preservation issues. It also makes me wonder why this idea keeps on being revisited over and over again. Similar to the format registry situation, is this perhaps another manifestation of the "*trying to replace imagination and expertise with an encyclopedia* phenomenon? What exactly is the point of classifying or ranking formats according to perceived preservation "risks" if these "risks" are largely based on theoretical considerations, and are so general that they say next to nothing about individual file (format) instances? Isn't this all a bit like [searching for Bigfoot](http://www.searchingforbigfoot.com/)? Wouldn't the time and effort involved in these activities be better spent on trying to solve, document and publish concrete format-related problems and their solutions? Some examples can be found [here](http://unsustainableideas.wordpress.com/2012/10/15/ppt-4-adventure-learning/) (accessing old Powerpoint 4 files), [here](http://notepad.benfinoradin.info/2013/09/12/it-takes-a-village-to-save-a-hard-drive/) (recovering the contents of an old Commodore Amiga hard disk), [here](http://anjackson.github.io/keeping-codes/experiments/BBC%20Micro%20Data%20Recovery.html) (BBC Micro Data Recovery), or even [here](http://wiki.opf-labs.org/display/TR/OPF+File+Format+Risk+Registry) (problems with contemporary formats)?

I think there could also be a valuable role here for some of the *FFMA*-related work in all this: the aggregation component of *FFMA* looks really useful for the automatic discovery of, for example, software applications that are able to read a specific format, and this could be could be hugely helpful in solving real-world preservation problems.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/09/30/assessing-file-format-risks-searching-bigfoot/)
