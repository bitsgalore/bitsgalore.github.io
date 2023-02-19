---
layout: post
title: Why PDF/A validation matters, even if you don't have PDF/A
tags: [PDF,VeraPDF,preservation-risks]
comment_id: 34
---

This is the first instalment of a 2-part blog. It was prompted by the upcoming Digital Preservation Coalition briefing [*When is a PDF not a PDF?*](http://www.dpconline.org/events/details/95-preserving-pdfs-jul15), for which I was asked to prepare a presentation. My initial idea was to give an overview of the work we did on PDF preservation risk assessment using a PDF/A validator in the [SCAPE](http://www.scape-project.eu/) project. Most of this has already been [covered]({{ BASE_PATH }}/2012/12/19/identification-pdf-preservation-risks-apache-preflight-first-impression) by a [series]({{ BASE_PATH }}/2013/07/25/identification-pdf-preservation-risks-sequel) of [earlier blog posts]({{ BASE_PATH }}/2014/01/27/identification-pdf-preservation-risks-analysis-govdocs-selected-corpus). Those blogs very much represent different stages of a work in progress, and I think this makes them somewhat challenging for readers who are new to the subject.

<!-- more -->

The purpose of this 2-part blog is twofold: first it is an attempt to give an accessible overview of the earlier work on PDF preservation risks, stressing the importance of PDF/A validator tools in detecting these risks. Second, it  provides some tentative suggestions of how the ongoing work on the new [VeraPDF](http://verapdf.org/) PDF/A validator could close some of the gaps and limitations of the SCAPE work.

## Preservation risks of PDF

The PDF format has a number of features that don't sit well with the aims of long-term preservation and accessibility. This includes encryption and password protection, external dependencies (e.g. fonts that are not embedded in a document), and reliance on external software. More details can be found in the [PDF entry of the OPF File Format Risk Registry](http://wiki.opf-labs.org/display/TR/Portable+Document+Format). Below are some examples; I included download links, so you can try them out for yourself.

### Document Open password

If you try to open file [*encryption_openpassword.pdf*](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/encryption_openpassword.pdf?raw=true) in Adobe Acrobat, you end up with this dialog:

![]({{ BASE_PATH }}/images/2015/07/openpassword.png)

Without the password, the file cannot be opened at all.

### Print password

File [*encryption_noprinting.pdf*](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/encryption_noprinting.pdf?raw=true) can be opened normally, but you cannot print it:

![]({{ BASE_PATH }}/images/2015/07/printpassword.png)

### Embedded Quicktime movie

File [*embedded\_video\_quicktime.pdf*](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/embedded_video_quicktime.pdf?raw=true) contains multimedia content in [Quicktime](http://fileformats.archiveteam.org/wiki/Quicktime) format. Acrobat cannot render this format natively, and relies on an external player. This is what happened when I opened the file on my PC:  

![]({{ BASE_PATH }}/images/2015/07/embeddquicktime.png)

After I clicked on *Get Media Player*, I was taken [here](http://cgi.adobe.com/special/acrobat/mediaplayerfinder/mediaplayerfinder.cgi?):

![]({{ BASE_PATH }}/images/2015/07/embeddquicktime2.png)

I wasn't able to configure Acrobat to use a media player that supports Quicktime [^1].

### External reference to multimedia file

The file [*movie.pdf*](https://web.archive.org/web/20100714002808/http://acroeng.adobe.com/Test_Files/movie/movie.pdf) contains references to external multimedia files. If you click on any of them you get an error like this one:

![]({{ BASE_PATH }}/images/2015/07/movieexternal.png)

### Font not embedded

File [*calistoMTNoFontsEmbedded.pdf*](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/calistoMTNoFontsEmbedded.pdf?raw=true) uses *Calisto MT*, but the font is not embedded. Since *Calisto MT* is a Windows system font, the file looks fine on my Windows PC: 

![]({{ BASE_PATH }}/images/2015/07/fontsorig.png)

The font does not come pre-installed with common Linux distros, and as a result the file looks quite a bit different on my Linux machine:

![]({{ BASE_PATH }}/images/2015/07/fontslinux.png)

### 3D content

The file [*digitally_signed_3D_Portfolio.pdf*](https://github.com/openpreserve/format-corpus/blob/master/pdfCabinetOfHorrors/digitally_signed_3D_Portfolio.pdf?raw=true) contains 3D artwork. Acrobat correctly renders the 3D content, which can be manipulated interactively by the user:

![]({{ BASE_PATH }}/images/2015/07/3dacrobat.png)

However, Acrobat aside, the majority of PDF readers don't support 3D content, with the result that in other readers you may end up with something like this:

![]({{ BASE_PATH }}/images/2015/07/3dsumatra.png)

## Detecting risky features

Archives or libraries may want to check their PDFs for one or more features like those shown above. Reasons for doing so include:

* Pre-ingest checks against an institutional policy (e.g. an archive may not accept PDFs that are password protected)

* Profiling of existing collections for preservation risks (e.g. embedded multimedia content in hard-to-render formats)

For this quite a few useful software tools are already available. For example, [qpdf](https://github.com/qpdf/qpdf) gives detailed information about encryption and password protection:

![]({{ BASE_PATH }}/images/2015/07/encryptqpdf.png)

Similarly, the [pdffonts](http://www.linuxcommand.org/man_pages/pdffonts1.html) tool that is part of [xpdf](http://www.foolabs.com/xpdf/) is useful for checking whether fonts in a PDF are embedded:

![]({{ BASE_PATH }}/images/2015/07/fontsxpdf.png)

As the number of features you want to check for increases, this approach becomes increasingly cumbersome: most of tools only cover *some* features, so you rapidly end up having to deal with a multitude of software tools and output formats. So you may ask yourself if there's a way to do this more efficiently.

## PDF/A validation

This is where [PDF/A](https://en.wikipedia.org/wiki/PDF/A) enters the picture. The PDF/A standards are nothing more than a set of profiles that impose some restrictions on a PDF, ruling out features that are not well-suited to long-term accessibility. Unsurprisingly, these include the very same features that we are interested in here, such as encryption, non-embedded fonts, multimedia content, and so on. Several tools exist that compare a PDF against PDF/A and report any deviations. These PDF/A validators are typically used to verify "true" PDF/A files; however, they can also be used to detect user-specified risky features in regular PDFs.

The professional version of Adobe Acrobat has a PDF/A validator built into its [Preflight](http://help.adobe.com/en_US/acrobat/X/pro/using/WS58a04a822e3e50102bd615109794195ff-7b82.w.html) tool. After opening a PDF in Acrobat, it allows you to verify its compliance with a number of profiles, including PDF/A (currently A-1, 2 and 3):

![]({{ BASE_PATH }}/images/2015/07/acrobatpreflight1.png)

This results in output as shown here:

![]({{ BASE_PATH }}/images/2015/07/acrobatpreflight2.png) 

[This PDF](http://acroeng.adobe.com/Test_Files/classic_multimedia//Jpeg_linked.pdf)[^2] (which isn't a PDF/A) violates the PDF/A-1a profile in several ways, but supposing we're only interested in encryption and non-embedded fonts, the relevant information can be extracted from Preflight's output quite easily. This example demonstrates the overall feasibility of identifying preservation risks with a PDF/A validator, but it is not scalabe to situations where you need to verify large volumes of PDFs. This will be the main focus of the [second part]({{ BASE_PATH }}/2015/07/08/why-pdfa-validation-matters-part-2) of this blog series.

[^1]: Acrobat's *Preferences* do include some options for configuring behavior with multimedia content (explained [here](https://helpx.adobe.com/acrobat/using/playing-video-audio-multimedia-formats.html)), but the list of media players in the *Preferred Media Player* dropdown list only included Windows Media Player and Adobe Flash Player. Neither of these support Quicktime. VLC Media player *does* support Quicktime, but it is not included in the dropdown list, leaving me no way to configure it. Bummer!

[^2]: At the time of writing the Acrobat Engineering site was down, and this particular PDF is not included in any Wayback crawls either. Bummer again!

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2015/07/07/why-pdfa-validation-matters-even-if-you-dont-have-pdfa/)
