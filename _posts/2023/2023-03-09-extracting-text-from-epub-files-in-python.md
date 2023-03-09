---
layout: post
title: Extracting text from EPUB files in Python
tags: [EPUB, Apache-Tika, python]
comment_id: 85
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/03/clockwork-extraction.jpg" alt="Street scene showing crowd gathered around an open carriage in which a dentist performs a tooth extraction on a patient. Next to the patient a man is banging on a large drum.">
  <figcaption>Clockwork picture of an itinerant dentist performing an extraction in French rural scene, wood frame, metal workings, first half 19th century. <a href="https://wellcomecollection.org/works/hwpe3cxp">Science Museum, London</a>. <a href="https://creativecommons.org/licenses/by/4.0/">Attribution 4.0 International (CC BY 4.0)</a> (cropped from original).</figcaption>
</figure>

This blog post provides a brief introduction to extracting unformatted text from EPUB files. The occasion for this work was a request by my Digital Humanities colleagues who are involved in the [SANE (Secure ANalysis Environment) project](https://www.surf.nl/en/news/sane-secure-data-environment-for-social-sciences-and-humanities). The work on this project includes a use case that will use the SANE environment to analyse text from novels in EPUB format. My colleagues were looking for some advice on how to implement the text extraction component, preferably using a Python-based solution.

So, I started by making a shortlist of potentially suitable tools. For each tool, I wrote a minimal code snippet for processing one file. Based on this I then created some simple demo scripts that show how each tool is used within a processing workflow. Next, I applied these scripts to two data sets, and used the results to obtain a first impression of the performance of each of the tools.  


<!-- more -->

## Evaluated tools

I evaluated the following tools:

1. [**Tika-python**](https://github.com/chrismattmann/tika-python). This is a Python wrapper for [Apache Tika](https://tika.apache.org/) (which itself is a Java application). Apache Tika is a toolkit for text and metadata extraction from a wide range of file formats, including EPUB.

2. [**Textract**](https://github.com/deanmalmgren/textract). This offers text extraction functionality that is similar to Tika, but unlike Tika, Textract is natively written in Python.

3. [**EbookLib**](https://github.com/aerkalov/ebooklib). This is a Python library for reading and writing E-books in various formats, including EPUB (both EPUB 2 en EPUB 3). EbookLib is also the E-book library that is used by Textract.

The following table shows the versions of these tools that I used in my tests:

|Software|Version|
|:--|:--|
|Tika-python|2.6.0|
|Textract|1.6.5|
|EbookLib|0.18|

## Test environment and data

For all of my tests I used a simple desktop PC running Linux Mint 20.1 (Ulyssa), MATE edition, with Python 3.8.10.

I used two data sets:

1. A selection of 15 files in [EPUB 2.0.1](https://idpf.org/epub/201) format from the KB's [DBNL](https://www.dbnl.org) (Digital Library for Dutch Literature) collection.
2. A selection of 10 files in [EPUB 3.2](https://www.w3.org/publishing/epub3/epub-spec.html) format from [Standard Ebooks](https://standardebooks.org/).

All files in both data sets are structurally valid EPUB (2.0.1 / 3.2): validation with [EPUBCheck](https://github.com/w3c/epubcheck) 4.2.6 didn't result in any reported errors or warnings[^1].

## Tika-python

After installing Tika-python, as a first test I tried to write a minimal code snippet that extracts the text from one single EPUB, and then writes the result as UTF-8 encoded text to a file. Following Tika-python's [README](https://github.com/chrismattmann/tika-python/blob/master/README.md) (the example under "Parser Interface"), I started out with with this:

```python
#! /usr/bin/env python3

import tika
from tika import parser

fileIn = "berk011veel01_01.epub"
fileOut = "berk011veel01_01.txt"

parsed = parser.from_file(fileIn)
content = parsed["content"]

with open(fileOut, 'w', encoding='utf-8') as fout:
    fout.write(content)
```

### Metadata strings in text output

Inspection of the resulting output file showed a succession of text strings with the names of embedded fonts towards the end of the file. As an example: 

```
Charis SIL Bold Italic

::
::

Charis SIL Small Caps
```

When I ran Tika (the Java application) directly without using the Tika-python wrapper, results were as expected. A closer inspection of the Tika-python source code showed that Tika-python's parsing of the Tika output doesn't quite work the way it should, with the result that extracted metadata is erroneously included in the text output.

### Workaround: set service to text

Fortunately there's a simple workaround for this. In the parser function call, just add the "service" parameter and set its value to "text", as shown here:

```python
#! /usr/bin/env python3

import tika
from tika import parser

fileIn = "berk011veel01_01.epub"
fileOut = "berk011veel01_01.txt"

parsed = parser.from_file(fileIn, service='text')
content = parsed["content"]

with open(fileOut, 'w', encoding='utf-8') as fout:
    fout.write(content)
```

With this change, the font-related text strings were no longer reported.

### Image tags and alt-text strings

Unfortunately, setting the "service" parameter in this way has the unexpected side-effect that the text output now includes tags with alt-text descriptions for any images in the file. For example:

```
[image: cover]


Aster Berkhof

Veel geluk, professor!


[image: DBNL]
```

### Different behaviour between Tika app and TikaServer

I initially thought this was also a bug in Tika-python, but it turns out this isn't the case. Using the Tika Java application directly:

```
java -jar ~/tika/tika-app-2.6.0.jar -t berk011veel01_01.epub  > berk011veel01_01-app.txt
```

This resulted in an output file with no alt-text strings. However, Tika-python doesn't wrap around Tika-app, but instead around [TikaServer](https://cwiki.apache.org/confluence/display/TIKA/TikaServer). After starting TikaServer, I used the command below to processes the same EPUB:

```
curl -T berk011veel01_01.epub  http://localhost:9998/tika  --header "Accept: text/plain" > berk011veel01_01-server.txt
```

The resulting file also included the offending image tags and alt-text strings. So, the Tika application and TikaServer behave differently. After reporting [an issue](https://issues.apache.org/jira/browse/TIKA-3969?filter=12326160) for this, I received a confirmation from Tika's lead developer:

> There's a subtle difference in the handlers used in tika-app and tika-server. We're using the "RichTextContentHandler" in server but not in app. I think I've known about this for a while, but we'll be breaking behaviour for whichever one we fix.

I also created a [separate issue](https://github.com/chrismattmann/tika-python/issues/389) at Tika-python for the inclusion of metadata in the text output. Unfortunately this issue is closely related to (and partly the result of) the upstream issue in TikaServer. So until that upstream issue is fixed, the current (slightly confusing) situation will most likely persist.

### OCR if Tesseract is installed

By default, Tika applies optical character recognition (OCR) to any images in an EPUB if the [Tesseract](https://github.com/tesseract-ocr/tesseract) software is installed, and includes the OCR output in the extracted text. In many cases (at least for ours!) this might not be the desired behaviour. I only found out about this weeks after doing the original tests that are described in this post. Re-running some of the tests suddenly resulted in slightly larger output files, with text output that wasn't originally there. It turns out that the root cause was that I had installed some software that installs Tesseract as a dependency (but I wasn't aware of this). It's possible to [disable OCR](https://cwiki.apache.org/confluence/display/TIKA/TikaOCR#TikaOCR-disable-ocr) in the Java application and TikaServer using a [command-line option](https://tika.apache.org/1.9/configuring.html#Using_a_Tika_Configuration_XML_file) that points to a  configuration file. I haven't found a way to do this in Tika-python. The safest option might be to make sure that Teseract is not installed, or to rename Tesseract's installation folder.

## Textract

As with Tika-python, as a first test I again created a minimal code snippet for processing one EPUB file:

```python
#! /usr/bin/env python3

import textract

fileIn = "berk011veel01_01.epub"
fileOut = "berk011veel01_01.txt"

content = textract.process(fileIn, encoding='utf-8').decode()

with open(fileOut, 'w', encoding='utf-8') as fout:
        fout.write(content)
```

For the very first EPUB file (from the DBNL collection) this resulted in an empty output file. Results were similar for most other DBNL EPUBs, and Textract only managed to extract a handful of words at most. Results were considerably better for the "Standard Ebooks" files, with output that was similar to Tika-python in most cases. I [reported](https://github.com/deanmalmgren/textract/issues/455) this issue with the developers.

## EbookLib

I mainly included EbookLib, because Textract uses it "under the hood" for EPUB, and I was curious if using it directly would give me similar results as Textract. Based on its [documentation](https://docs.sourcefabric.org/projects/ebooklib/en/latest/tutorial.html#reading-epubdocs.sourcefabric.org/projects/ebooklib/en/latest/tutorial.html#reading-epub) I created the following minimal code snippet:

```python
#! /usr/bin/env python3

from html.parser import HTMLParser
import ebooklib
from ebooklib import epub

fileIn = "berk011veel01_01.epub"
fileOut = "berk011veel01_01.txt"

book = epub.read_epub(fileIn)
content = ""

for item in book.get_items():
    if item.get_type() == ebooklib.ITEM_DOCUMENT:
        bodyContent = item.get_body_content().decode()
        f = HTMLFilter()
        f.feed(bodyContent)
        content += f.text

with open(fileOut, 'w', encoding='utf-8') as fout:
        fout.write(content)
```

Compared to Tika-python and Textract, the EbookLib script is a bit more involved, as EbookLib doesn't provide any high-level text extraction functions. Instead, the user must iterate over all document items, extract the (X)HTML, and then convert that to unformatted text. At first glance, tests with the DBNL and Standard Ebooks EPUBs didn't result in any issues, and the results were similar to Tika-python.

## Demonstration scripts

Based on the above minimal code snippets, I created [three simple demonstration scripts](https://github.com/KBNLresearch/textExtractDemo) for Python-tika, Textract and Ebooklib. Each of these scripts extracts the text of each EPUB file in a user-defined input directory. The extracted text is then written to a user-defined output directory. Each script also writes a file with word counts for the extraction results, which is useful for a rough comparison of the different tools.

I ran each script twice, using the DBNL and Standard Ebooks data sets as input, respectively.

## Word counts

The table below shows the resulting word counts for the books in the DBNL data set:

|File name|Words (Tika)|Words (Textract)|Words (EbookLib)|
|:--|:--|:--|:--|
|eern001lief01_01.epub|25450|1|25446|
|spro002mure01_01.epub|50553|0|50549|
|berk011veel01_01.epub|67978|0|67974|
|sche034drie01_01.epub|203853|3|203352|
|jous010supe01_01.epub|202495|0|202491|
|dele035wegv01_01.epub|76536|0|76530|
|verv017eerl01_01.epub|33844|0|33840|
|dhae007euro01_01.epub|394455|2|394400|
|gomm002uurw01_01.epub|43754|0|43731|
|gang009lalb01_01.epub|28453|4|28381|
|geel005bloe01_01.epub|76316|0|76312|
|hart008droo02_01.epub|77283|0|77279|
|eede003vand04_01.epub|120481|6|120310|
|meij031tuss02_01.epub|145678|4|145665|
|maas013blau01_01.epub|55099|0|55093|

Note the extreme (near zero) word counts for Textract. The results for Tika and EbookLib are roughly the same.

Running the scripts on the Standard Ebooks EPUBs gave the following result: 

|File name|Words (Tika)|Words (Textract)|Words (EbookLib)|
|:--|:--|:--|:--|
|william-shakespeare_king-lear.epub|28442|18621|28430|
|david-garnett_lady-into-fox.epub|25240|25223|25228|
|joseph-conrad_heart-of-darkness.epub|38717|38698|38705|
|anthony-trollope_the-dukes-children.epub|223014|222995|223002|
|agatha-christie_the-mysterious-affair-at-styles.epub|57401|57229|57271|
|edgar-allan-poe_the-narrative-of-arthur-gordon-pym-of-nantucket.epub|71931|71837|71863|
|p-g-wodehouse_short-fiction.epub|212224|212182|212212|
|robert-louis-stevenson_the-strange-case-of-dr-jekyll-and-mr-hyde.epub|26370|26345|26358|
|h-g-wells_the-time-machine.epub|33044|33024|33032|
|thorstein-veblen_the-theory-of-the-leisure-class.epub|106537|106515|106525|

In this case, all three tools resulted in similar word counts. The exception here is the "King Lear" EPUB, which for Textract gave a word count that was about 10 thousand lower than for the other tools. I haven't looked in detail where this difference is coming from exactly, but it confirms that in its current state, Textract isn't a suitable tool for our purposes.

## Table of Contents

Depending on the structure of the source EPUB, the extraction result may or may not contain a table of contents. In [EPUB 2](https://idpf.org/epub/20/spec/OPF_2.0.1_draft.htm), the table of contents is implemented as an XML-formatted "Navigation Control File" (NCX). The NCX was replaced by the ["Navigation Document"](https://www.w3.org/publishing/epub3/epub-overview.html#sec-nav-nav-doc) (which is an XHTML file) in EPUB 3. Neither Tika nor EbookLib extract NCX resources, but both do extract Navigation Documents. Consequently, in most cases the extraction result only includes a table of contents for EPUB 3 files. Textract extracts neither the NCX nor the Navigation Document.

## Conclusions

Based on these tests, both Tika-python and EbookLib look like potentially suitable Python-based tools for extracting unformatted text from EPUB files. Out of these, Tika-python provides the most straightforward interface. Tika also supports a wide range of other file formats, so any code based on Tika's text extraction can be easily extended to other formats later.

The inclusion of tags and alt-text descriptions for images in Tika's output may be a problem though. As an example, imagine a researcher who uses Tika-python to analyse the emergence of certain words or phrases through time using EPUB versions of 19th century books. Any alt-text descriptions in such materials would most likely be contemporary, and as such they would "pollute" the original "signal" (19th century text) with modern language. So, prospective users of Tika-python should carefully review whether this behaviour is acceptable for their use case. The inclusion of optical character recognition output from embedded images in the extraction result can also result in some unexpected surprises, so it's important that users are aware of Tika's default behaviour in this regard.

EbookLib doesn't have these drawbacks, but the absence of a high-level text extraction interface does require some more work on the user's side. Also, since EbookLib only supports a limited number of Ebook formats, extending any code based on it to other file formats will be less straightforward.

In its current form, Textract is not suitable for our use case.

## Limitations

It's important to highlight the limitations of this analysis. First, it is based on only two small, homogeneous data sets, both of which only contain structurally valid EPUB files. It's unclear how well these results translate to more heterogeneous collections (which often contain files that violate the format specifications in various ways). Second, the main objective here was to obtain a broad impression of the behaviour of the tested tools. The scope didn't include an in-depth analysis of the accuracy and completeness of the extraction results. Finally, I didn't look into the computational performance of the tested tools. As the SANE use case will only involve processing a limited number of files, performance isn't important here.

## Link to demo scripts

EPUB text extraction demo:

<https://github.com/KBNLresearch/textExtractDemo>

[^1]: For convenience I actually used the EPUBCheck Python wrapper: <https://github.com/titusz/epubcheck/>.


