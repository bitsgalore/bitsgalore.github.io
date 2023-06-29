---
layout: post
title: VeraPDF parse status as a proxy for PDF rendering&colon; experiments with the Synthetic PDF Testset
tags: [PDF, format-validation, VeraPDF, JHOVE]
comment_id: 89
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/06/barnum-bailey.jpg" alt="Vintage lithograph circus poster that shows a circus ring. In the front is a woman in a red dress, standing on horseback. Behind her there are more horses, with a variety of circus artists, including acrobats and jugglers, performing on horseback as well. In the background acrobats are walking on a tightrope.">
  <figcaption><a href="https://www.flickr.com/photos/boston_public_library/6554392117/in/album-72157629549177588/">"The Barnum & Bailey greatest show on earth"</a>. Used under <a href="https://creativecommons.org/licenses/by/2.0/">CC BY-BY 2.0</a>, via Boston Public Library.</figcaption>
</figure>


Last month I [wrote this post]({{ BASE_PATH }}/2023/05/25/identification-of-pdf-preservation-risks-with-verapdf-and-jhove), which addresses the use of [JHOVE](https://jhove.openpreservation.org/) and [VeraPDF](https://verapdf.org/) for identifying preservation risks in PDF files. In the concluding section I suggested that VeraPDF's parse status might be used as a rough "validity proxy" to identify malformed PDFs. But does VeraPDF's parse status actually have any predictive value for rendering? And how does this compare to what JHOVE tells us? This post is a first attempt at answering these questions, using data from the [Synthetic PDF Testset for File Format Validation](https://www.radar-service.eu/radar/en/dataset/JtlOdwQquZWDqQdq) by Lindlar, Tunnat and Wilson.

<!-- more -->

## Goal and objectives of this post

The main goal of this post is to get more insight into the associations between VeraPDF parse errors and rendering behaviour, and to see if the occurrence of these parse errors can be used as a rough "validity proxy" to identify malformed PDFs. For this I propose a tentative methodology, and apply this to a previously published annotated dataset of synthetic PDF files. I then try to use the results to answer the following questions:

1. To what extent is VeraPDF's reported parse status associated with rendering behaviour in Adobe Acrobat?
2. How does this compare against the occurrence of JHOVE validation errors?
3. What are the implications of the answers to 1. and 2. for preservation workflows?

## Synthetic PDF Testset for File Format Validation

In 2017, Lindlar, Tunnat and Wilson published the [Synthetic PDF Testset for File Format Validation](https://www.radar-service.eu/radar/en/dataset/JtlOdwQquZWDqQdq). It is a corpus of 88 small, hand-crafted PDFs, that each violate the PDF format specification in different ways. The focus of the dataset is on "basic structure violations at the header, trailer, document catalog, page tree node and cross-reference levels", and it also includes "violations at the page node, page resource and stream object level". The dataset comes with a descriptive spreadsheet, which includes a column with rendering behaviour of each file in Adobe Acrobat Professional XI Pro (11.0.15). The dataset was specifically created for testing the validation functionality of JHOVE's PDF module. A detailed discussion of this dataset can be found in the authors' [2017 iPRES paper](https://phaidra.univie.ac.at/detail/o:931074).

## Running JHOVE and VeraPDF

As a first test, I ran all PDFs in the dataset through JHOVE (v 1.28.0) and VeraPDF (v 1.22.3). For JHOVE I used the following generic command line arguments:

```
jhove -m PDF-hul -h XML -i whatever.pdf whatever-jhove.xml
```

And for VeraPDF:

```
verapdf --off --addlogs --extract whatever.pdf > whatever-vera.xml
```

Here, the `--off` switch disables (PDF/A or PDF/UA) validation, and the `--addlogs` switch ensures that information about any run-time warnings is included in the output. The `--extract` switch enables feature extraction[^1]. The raw JHOVE and VeraPDF output files are available [here](https://github.com/KBNLresearch/pdf-characterisation/tree/main/output/lindlar-tunnat-wilson). I created [this script](https://github.com/KBNLresearch/pdf-characterisation/blob/main/scripts/jhove-verapdf-validation-run.py), which runs JHOVE and VeraPDF for all files in the dataset, and creates a [comma-delimited file](https://github.com/KBNLresearch/pdf-characterisation/blob/main/output/lindlar-tunnat-wilson/data.csv) with the following output metrics for each PDF:

1. The JHOVE validation status
2. A Boolean flag that is *True* if VeraPDF reported one or more parse errors, and *False* otherwise
3. A Boolean flag that is *True* if VeraPDF reported one or more warnings, and *False* otherwise

## JHOVE validation status vs VeraPDF parse errors and warnings

First, let's look at to what degree the JHOVE and VeraPDF results are interrelated. For this, I created two contingency tables that cross-tabulate the JHOVE and VeraPDF results. First, here are the JHOVE validation status outcomes[^2] versus the VeraPDF parse error flags:

| JHOVE status               |   VeraPDF parse errors |   No VeraPDF parse errors |   All |
|:---------------------------|-----------------------:|--------------------------:|------:|
| Not well-formed            |                     62 |                         7 |    69 |
| Well-Formed, but not valid |                      2 |                         4 |     6 |
| Well-Formed and valid      |                      4 |                         9 |    13 |
| All                        |                     68 |                        20 |    88 |

Most PDFs that JHOVE considers "Not well-formed" also result in VeraPDF parse errors, which isn't unexpected. Interestingly, out of the 13 PDFs that JHOVE considers "Well-Formed and valid", 4 nevertheless result in VeraPDF parse errors. Most of the PDFs that JHOVE considers "Well-Formed, but not valid" don't result in veraPDF parse errors.

Here's the contingency table of JHOVE validation status outcomes versus the VeraPDF warning flags:

| JHOVE status               |   VeraPDF warnings |   No VeraPDF warnings |   All |
|:---------------------------|-------------------:|----------------------:|------:|
| Not well-formed            |                 62 |                     7 |    69 |
| Well-Formed, but not valid |                  2 |                     4 |     6 |
| Well-Formed and valid      |                  5 |                     8 |    13 |
| All                        |                 69 |                    19 |    88 |

The results are almost identical to those of the VeraPDF parse errors. The only difference here, is that out of the PDFs that are "Well-Formed and valid" according to JHOVE, 5 resulted in VeraPDF warnings, whereas only 4 resulted in VeraPDF parse errors.

We can also use statistical measures to characterise the association between the JHOVE and VeraPDF metrics. A suitable measure for nominal-scale variables is [Cramér's *V*](https://en.wikipedia.org/wiki/Cram%C3%A9r%27s_V). Cramér's *V* can have any value from 0 to 1, where 0 indicates no association between two variables, and 1 a complete association. The following table shows the calculated Cramér's V[^3] values between JHOVE status and VeraPDF parse errors, and JHOVE status and VeraPDF warnings, respectively:

|           Metrics                  |Cramér's V |p      |df |N  |
|:-----------------------------------|----------:|------:|--:|--:|
|JHOVE status vs VeraPDF parse errors| 0.56      |<.001  |2  |88 |
|JHOVE status vs VeraPDF warnings    | 0.51      |<.001  |2  |88 |

The *p*-values were calculated from a [Chi-square test](https://en.wikipedia.org/wiki/Chi-squared_test) of independence between the JHOVE and VeraPDF metrics. Put simply, for each metric pair, they denote the probability of the observed metric combinations occurring for the null hypothesis that the metrics are independent. Since both *p*-values are very small, this adds credence that the null hypothesis must be rejected, and that both metrics are in fact related. The Cramér's *V* values express the strength of this association, confirming the relatively strong association between JHOVE's validation status and the occurrence of VeraPDF parse errors and warnings. The *df* column shows the degrees of freedom of the Chi-square distribution.

## Simplifying the ground truth

Since the spreadsheet that is part of the PDF Testset includes information on rendering behaviour in Acrobat, we can use this as "ground truth" to test to what degree the JHOVE and VeraPDF results are indicative of rendering problems. The rendering outcomes in the spreadsheet[^4] are not based on any controlled vocabulary. For about half of the files, rendering behaviour is described as a simple "Yes" or "No". For the remaining files, it is described in more elaborate terms, such as:

- "Yes, but missing content"
- "Yes. Wrong font"
- "Yes. Adobe tries to change something when opening". 

This multitude of very specific (and often unique) values makes it hard to do any meaningful (quantitative) analysis. Because of this,  I simplified the rendering outcomes into 3 discrete classes:

- Does not render
- Renders with issues
- Renders normally

[This file](https://github.com/KBNLresearch/pdf-characterisation/blob/main/misc/lindlar-tunnat-wilson/lindlar-tunnat-wilson-jhove-vera-rendering.csv) contains, for all PDFs in the dataset, the simplified rendering outcomes, along with the JHOVE and VeraPDF results.

## JHOVE and VeraPDF results vs rendering behaviour

As a first step towards exploring any associations between the JHOVE and VeraPDF metrics and rendering behaviour, I used this file to create some more contingency tables. The first one shows the joint frequencies of rendering outcomes against JHOVE validation status:

| Rendering \ JHOVE status | Not well-formed | Well-Formed, but not valid | Well-Formed and valid | All |
|:-------------------------|----------------:|---------------------------:|----------------------:|----:|
| Does not render          |              31 |                          1 |                     0 |  32 |
| Renders with issues      |              27 |                          2 |                     9 |  38 |
| Renders normally         |              11 |                          3 |                     4 |  18 |
| All                      |              69 |                          6 |                    13 |  88 |


Out of the 32 files in the "Does not render" category, JHOVE considers 31 as "Not well-formed", and 1 as "Well-Formed, but not valid". JHOVE was less successful at identifying files in the "Renders with issues" category. Out the 38 files, 9 (24%) are "Well-Formed and valid". One striking result is that from the 18 files in the "Renders normally" category, the majority (11, or 61%) are "Not well-formed" according to JHOVE. 

Here's the table for the veraPDF parse errors:

| Rendering \ VeraPDF status | Parse errors | No parse errors | All |
|:---------------------------|-------------:|----------------:|----:|
| Does not render            |           30 |               2 |  32 |
| Renders with issues        |           31 |               7 |  38 |
| Renders normally           |            7 |              11 |  18 |
| All                        |           68 |              20 |  88 |

VeraPDF's parse errors output is slightly less effective at identifying PDFs in the "Does not render" category: 30 of these result in parse errors, and 2 in no parse errors. From the 38 files in the "Renders with issues" category, 7 (18%) don't result in parse errors. Interestingly, 61% of the files in the "Renders normally" category (11 out of 18) does not result in parse errors. This is a marked contrast with the JHOVE, which shows the opposite behaviour by considering 61% of these files "Not well-formed".

And finally here's the table for VeraPDF's warnings:

| Rendering \ VeraPDF status | Warnings | No warnings |   All |
|:---------------------------|---------:|------------:|------:|
| Does not render            |       30 |           2 |    32 |
| Renders with issues        |       31 |           7 |    38 |
| Renders normally           |        8 |          10 |    18 |
| All                        |       69 |          19 |    88 |

The pattern here is nearly identical to the parse errors[^6].

As before, we can use the Cramer's *V* statistic to express the strength of the associations between the tool metrics and the rendering results:

|     Tool metric       | Cramér's V | p    |df |N |
|:----------------------|-----------:|-----:|--:|--:|
| JHOVE status          |      0.23  |.011  |4  |88 |
| VeraPDF parse errors  |      0.46  |<.001 |2  |88 |
| VeraPDF warnings      |      0.41  |<.001 |2  |88 |

The low *p*-values (again from a Chi-square test) indicate that all tool metrics are related to the rendering results. The strength of these associations is expressed through Cramér's *V*. From the table we see that both VeraPDF metrics show a stronger association with rendering behaviour, compared to JHOVE. This pattern persists (although to a slightly lesser degree) if we adjust for the fact that JHOVE's validation status can have 3 possible values, whereas the VerapDF metrics are simply binary flags[^5]. The large *V* value for the VeraPDF parse errors metric is mostly caused by the large overlap between the files in the "No parse errors" and "Renders normally" categories.

## Interpretation and conclusions

### VeraPDF vs JHOVE

Given the higher Cramér's *V* values, it would be tempting to conclude that VeraPDF is the "better" tool here compared to JHOVE. However, such a conclusion would be overly simplistic. The higher *V* values are primarily the result of the association between the "Renders normally" rendering class and VeraPDF's "No parse errors" class[^7]. By contrast, the majority of these "Renders normally" PDFs fall into JHOVE's "Not well-formed" class. This does not indicate any shortcoming on JHOVE's part. Instead, it simply suggests that JHOVE is more sensitive to (minor) deviations from the PDF specification that don't have a direct impact on rendering. This is exactly what we would expect if the rendering (viewer) application follows [Postel's Law](https://blog.dshr.org/2009/01/postels-law.html) (also known as the robustness principle), which states:

> be conservative in what you do, be liberal in what you accept from others

Unlike most rendering applications (which are designed to be forgiving when it comes to small deviations from the specifications), JHOVE is meant to be picky. The primary purpose of VeraPDF's parser is to provide the information needed to check conformance against PDF profiles (such as PDF/A and PDF/UA), and to extract technical features. It was *not* designed with JHOVE-style low-level (PDF 1.x ... PDF 2.0) format validation in mind. Therefore, its behaviour may be more similar to the parsers that are used by rendering applications.

### Implications for digital preservation workflows

Whether this behaviour is desirable within a digital preservation workflow highly depends on the specific purpose for which we want to "validate". If the objective is to identify PDFs that are seriously malformed, VeraPDF's parse status provides a simple and most likely sufficient indicator. JHOVE's validation output gives more detail, but its interpretation can be difficult. Also, since JHOVE hasn't yet been made up-to-date to [neither PDF 1.7](https://jhove.openpreservation.org/modules/pdf/) nor PDF 2.0, JHOVE's results for those PDF versions may be misleading. 

With that said, Postel's law also implies that more subtle rendering issues may go unnoticed. Since the PDF standard is so feature-rich, PDF rendering software is typically designed to ignore any unknown features in a file. This can result in incomplete rendering, without any errors being reported in the process. Since VeraPDF's parser appears to behave similarly to the parsers used by rendering applications, we should expect that subtle deviations from the standards may not result in any VeraPDF parse errors.

## Limitations

The above analysis is a first attempt at exploring the association between VeraPDF's parse errors and rendering behaviour. It's important to keep in mind the following limitations:

- It is based exclusively on small, synthetic test files. It's unclear how these results translate to ordinary PDFs "in the wild", which are typically much more complex.
- The ground truth (rendering results) was taken at face value from the 2017 data set, which was based on a version of Adobe Acrobat that is now outdated. Results may be different for more recent versions, or other rendering applications.

## JHOVE, VeraPDF and the future of PDF validation

While working on this write-up, the Open Preservation Foundation and the PDF Association [released](https://openpreservation.org/news/development-preview-pdf-file-checker-based-on-the-arlington-pdf-model/) a first development preview of a new veraPDF-powered PDF-checker that is based on the [Arlington PDF model](https://github.com/pdf-association/arlington-pdf-model). This software is capable of analyzing PDF files against the full PDF 2.0 (and earlier) specifications. An earlier draft of this post included an overview of the current PDF validation software landscape, with some reflections on where things may be heading. However, I ultimately left this out, as it would make this post too unwieldy. I will most likely address this in another follow-up post.

## Feedback welcome

As always, feedback to this post is highly appreciated. This includes (but is not limited to):

- The idea of using VeraPDF parse status a rough proxy for PDF validity.
- Comments, suggestions or criticism related to the methodology I used for analyzing the Synthetic PDF Testset.

## Further resources

- [Github repo with analysis scripts and raw data](https://github.com/KBNLresearch/pdf-characterisation)

- [Script for running JHOVE and VeraPDF](https://github.com/KBNLresearch/pdf-characterisation/blob/main/scripts/jhove-verapdf-validation-run.py)

- [Analysis script (contingency tables + Cramér's V calculation](https://github.com/KBNLresearch/pdf-characterisation/blob/main/scripts/jhove-verapdf-validation-analyze.py)

- [Synthetic PDF Testset for File Format Validation](https://www.radar-service.eu/radar/en/dataset/JtlOdwQquZWDqQdq)
 
- [A PDF Test-Set for Well-Formedness Validation in JHOVE - The Good, the Bad and the Ugly](https://phaidra.univie.ac.at/detail/o:931074) - 2017 iPRES paper by Lindlar, Tunnat & Wilson

- [How Valid is your Validation? A Closer Look Behind the Curtain of JHOVE](https://doi.org/10.2218/ijdc.v12i2.578) - 2017 International Journal of Digital Curation paper by Lindlar & Tunnat.

[^1]: Actually, feature extraction is not really needed for the purpose of this analysis.

[^2]: One PDF caused an exception in JHOVE, which resulted in the "unknown" validation status. For convenience I recoded this to "Not well-formed" for all analyses here.

[^3]: Using [Bergsma's bias-corrected version](https://doi.org/10.1016%2Fj.jkss.2012.10.002) ([non-paywalled copy](https://stats.lse.ac.uk/bergsma/pdf/cramerV3.pdf)), following the implemention given [here](https://stackoverflow.com/questions/20892799/using-pandas-calculate-cram%c3%a9rs-coefficient-matrix/39266194#39266194).

[^4]: Column "Adobe Professional XI Pro (11.0.15) - can file be opened?".

[^5]: As a test I re-calculated the statistics after lumping JHOVE's 'Well-Formed, but not valid' and "Not well-formed" classes. This did result in a slightly higher value of *V* (0.28). Lumping JHOVE's "Well-Formed, but not valid" and "Well-Formed and valid" classes raised *V* even further (0.32), but this is still less than the values for the VeraPDF metrics.

[^6]: The only difference is one file in the "Renders normally" category that resulted in a warning, without resulting in any parse errors.

[^7]: And to a lesser extent VeraPDF's "No warnings" class.
