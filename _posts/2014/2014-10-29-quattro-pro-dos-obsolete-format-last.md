---
layout: post
title: Quattro Pro for DOS&#58; an obsolete format at last?
tags: [Quattro-Pro,spreadsheets]
comment_id: 40
---

While browsing [ArchiveTeam's File Formats Wiki](http://fileformats.archiveteam.org/wiki/Main_Page) earlier this week, I came across some entries I created there on [Quattro Pro spreadsheets](http://fileformats.archiveteam.org/wiki/Quattro_Pro) two years ago. At the time I had also contributed some old Quattro Pro for DOS spreadsheets ([here](http://opf-labs.org/format-corpus/office/spreadsheet/wq1/) and [here](http://opf-labs.org/format-corpus/office/spreadsheet/wq2/)) from my personal archives to the [OPF format corpus](https://github.com/openplanets/format-corpus). Seeing those files again, I decided to spend an afternoon trying to access them using modern-day software. This turned out to be more challenging than expected. It even made me wonder whether, at long last, I had finally run into a case of the much discussed (but rarely observed) phenomenon of [format obsolescence](https://openpreservation.org/blog/2010/12/22/obsolescence-overrated/). Yes, big words indeed, and if anyone would like to prove me wrong, the comments section below is your friend!

<!-- more -->

## What it's all about

[Quattro Pro](http://en.wikipedia.org/wiki/Quattro_Pro) is a spreadsheet program that was first released in 1988. It's still around today as part of the  [WordPerfect Office suite](http://www.wordperfect.com/gb/product/office-suite/). [A number of file formats](http://fileformats.archiveteam.org/wiki/Quattro_Pro) have been associated with the software. This blog post covers the old Quattro Pro for DOS formats:

* [Quattro Pro for DOS, versions 1-4 (WQ1)](http://fileformats.archiveteam.org/wiki/WQ1)
* [Quattro Pro for DOS, versions 5.0 and 5.5 (WQ2)](http://fileformats.archiveteam.org/wiki/WQ2)

First of all, let's have a look to what extent contemporary spreadsheet software can handle these formats.

## MS Excel

Support for Quattro Pro spreadsheets (including recent versions of the format!) was removed altogether from more recent versions of Excel, as shown by this [overview of file formats that are not supported in Excel 2010](http://office.microsoft.com/en-us/excel-help/file-formats-that-are-supported-in-excel-HP010352464.aspx#BMunsupportedformats). On a side note, this list also includes all versions of [Lotus 1-2-3](http://fileformats.archiveteam.org/wiki/Lotus_1-2-3) (which was once widely used). Older versions of Excel did offer support for the format. According to Microsoft, [Excel 2003 supports Quattro Pro spreadsheets](http://office.microsoft.com/en-us/excel-help/about-opening-and-saving-files-from-other-programs-HP005253843.aspx) (albeit only after installing some converter add-ons from the Microsoft Office Web site[^1]). The website explicitly mentions Quattro Pro for DOS files, although it also says "there are some limitations to opening the worksheets".

## LibreOffice / OpenOffice

According to [this overview](https://wiki.documentfoundation.org/Feature_Comparison:_LibreOffice_-_Microsoft_Office), LibreOffice offers support for [Quattro Pro 6 (WB2)](http://fileformats.archiveteam.org/wiki/WB2) spreadsheets, but it cannot handle the older Quattro Pro for DOS formats. It doesn't mention [newer versions](http://fileformats.archiveteam.org/wiki/Quattro_Pro) of the format either. The situation is the same for [OpenOffice](https://wiki.openoffice.org/wiki/Documentation/OOo3_User_Guides/Getting_Started/File_formats).

## Tests with Quattro Pro X7

With neither Excel, LibreOffice or OpenOffice being able to open my spreadsheets, I went over to the WordPerfect website and [grabbed a trial version of the WordPerfect Office suite](http://www.wordperfect.com/gb/free-trials/) (which includes Quattro Pro X7). I then tried opening some files, all of which are available from the [spreadsheet](https://github.com/openplanets/format-corpus/tree/master/office/spreadsheet) section of the OPF Format Corpus.

### Simple numerical / text data

I started out by opening two versions of a spreadsheet that contains simple numerical and text data. The [first version](https://github.com/openpreserve/format-corpus/blob/master/office/spreadsheet/wq1/KSBASE.WQ1) has the [WQ1 (Quattro Pro for DOS version 1-4)](http://fileformats.archiveteam.org/wiki/WQ1) format. The file opens without problems:

![]({{ BASE_PATH }}/images/2014/10/ksbase_wq1.png)

I also had [another version of that spreadsheet](https://github.com/openpreserve/format-corpus/blob/master/office/spreadsheet/wq2/KSBASE.WQ2) in [WQ2 (Quattro Pro for DOS version 5)](http://fileformats.archiveteam.org/wiki/WQ2) format. Opening this file produced the following result:

![]({{ BASE_PATH }}/images/2014/10/ksbase_wq2.png)

For some reason the numbers in some columns (*A*, *C*, *D*, *G*, *H*, *I*) aren't displayed, but clicking on any of those calls reveals they are actually still there. Changing the formatting properties also makes them visible again, as shown below:

![]({{ BASE_PATH }}/images/2014/10/ksbase_wq2_fixedformatting.png)

So it looks like this is only a formatting issue. 

### Simple formulas, charts

Next I moved on to two other spreadsheets, which are a bit more interesting because they do some simple calculations[^2] and also contain charts. First I opened [KS4001.WQ2](https://github.com/openpreserve/format-corpus/blob/master/office/spreadsheet/wq2/KS4001.WQ2); the screenshot below shows how it is rendered by Quattro Pro:

![]({{ BASE_PATH }}/images/2014/10/ks4001_wq2.png)

The main calculation results are in cells *H17* and *H18*. The blue arrows highlight the cells from which these values are calculated. The calculated results are also correct. As before, two columns *appear* to be empty, but again, clicking these cells reveals that the underlying data (numbers in column *C*, and a calculation result in column *D*) are still present. I really don't remember what the chart originally looked like, but I was pleasantly surprised to see it's still displayed at all!   

### External links

Things got really interesting when I tried opening [this WQ2 spreadsheet](https://github.com/openpreserve/format-corpus/blob/master/office/spreadsheet/wq2/KS4000.WQ2). Upon opening, Quattro Pro comes up with a preview of the file, and a *Hotlinks* dialogue box:

![]({{ BASE_PATH }}/images/2014/10/ks4000_wq2_hotlinks.png)

I highlighted some areas in red; we'll get back to that in a second. I first selected *Open Supporting* in the dialogue box, and pressed *OK*. The result was that both this spreadsheet and [another one (our earlier *KSBASE.WQ2*)](https://github.com/openpreserve/format-corpus/blob/master/office/spreadsheet/wq2/KSBASE.WQ2) were loaded, so apparently it contains a link to that file. After loading, the spreadsheet displays as follows:

![]({{ BASE_PATH }}/images/2014/10/ks4000_wq2_opensupporting.png)

Now, pay special attention to the highlighted cells and compare them against the initial preview. This reveals some pretty dramatic changes:
some of the preview values in rows 4 and 5 are replaced by *Evaluator Stack Error* after the file is fully loaded. Clicking on those cells also results in odd sequences of Unicode characters in the formula bar:

![]({{ BASE_PATH }}/images/2014/10/stackError.png)

My best guess is that these cells contain external references that -for whatever reason- aren't resolved correctly. This in turn also influences the calculation results in cells *H17* and *H18*. I also tried opening the file using the *Update References* and *None* options; in both cases the results are the same as described above.  

With no access to the software that originally created the files, it is impossible to tell why the external references aren't working. It could be a bug of Quattro Pro, but I'm not ruling out that the spreadsheet may simply be faulty (e.g. perhaps the original referenced spreadsheet got replaced by an identically-named file at some point). Nevertheless, the fact that the correct cell values are displayed in the preview, shows that the original data *are* present, and it's rather worrying that Quattro Pro doesn't offer an option to fully load the files without updating/overruling them.

## Implications for long-term access

Apart from Quattro Pro, modern spreadsheet programs offer no support for Quattro Pro for DOS spreadsheets. The most recent version of Quattro Pro still reads both DOS era formats, although there are some problems. Some of these are formatting-related (e.g. cells that contain data showing up as blank), and can be easily remedied. The behaviour of one spreadsheet with an external dependency is much more problematic, especially because Quattro Pro updates the original values (which are stored in the file) after fully loading the spreadsheet. Migrating this spreadsheet to another format would result in the loss of some of the original data. So, based on this (admittedly cursory) analysis it looks like no modern-day software is able to correctly handle the Quattro Pro for DOS formats. Add to this that the Quattro Pro for DOS formats are proprietary with (as far as I'm aware) no publicly available specifications, and I think we have a pretty strong candidate for a format that may be (nearly) obsolete.

## Solutions

Although I haven't explored any concrete solutions for accessing Quattro Pro for DOS spreadsheets, some obvious routes would be:

* Run an old copy of Quattro Pro for DOS (e.g. in a virtual machine) and export the spreadsheet to e.g. the [Lotus 1-2-3](http://fileformats.archiveteam.org/wiki/Lotus_1-2-3) format (which is still reasonably well supported today).
* Run an old version of MS Excel (2003 or earlier) and export the spreadsheet to the [XLS](http://fileformats.archiveteam.org/wiki/XLS) format.

If anyone decides to have a go at this, I'd be very interested to see the results!

## Update: analysis by Euan Cochrane; Lotus 1-2-3 problematic as well?

In response to this blog post, Euan Cochrane has done [some  additional tests with my Quattro Pro files](https://www.webarchive.org.uk/wayback/en/archive/20160105203022/http://openpreservation.org/blog/2014/10/29/opening-johans-quattro-pro-files-quattro-pro-6-win-311/) using Quattro Pro 6 running in an emulated environment. Euan's analysis is highly recommended for any readers of this post. Moreover, trying to open a Lotus 1-2-3 file that Euan created as part of his analysis made me realise that Lotus 1-2-3 spreadsheets may also be more problematic than I initially thought. See my [comment](https://www.webarchive.org.uk/wayback/en/archive/20160105203022mp_/http://openpreservation.org/blog/2014/10/29/opening-johans-quattro-pro-files-quattro-pro-6-win-311/#comment-354) under Euan's blog post.

## Post script February 2019

In the folder from which I recovered this blog post, I also found a note with what looks like a comment to either this post, or Euan's follow-up post (as these comments have disappeared from the OPF site there's no way to tell). As it contains some relevant additional information on the Lotus 1-2-3, I've included it below:

>After I posted the above comment I also found out that IBM officially ceased its support for Lotus 123 on 30 September 2014. See the announcement here:
>
><http://www-01.ibm.com/common/ssi/cgi-bin/ssialias?subtype=ca&infotype=an&appname=iSource&supplier=897&letternum=ENUS913-091>
>
>Here's a recent feature on this from the Register:
>
><https://www.theregister.co.uk/2014/10/02/so_long_lotus_123_ibm_ceases_support_after_over_30_years_of_code/>
>
>This also applies to the Lotus SmartSuite and Organizer products (I'm not familiar with those products, so I have no idea if this has any additional format-related implications).
>
>### What will happen to the Lotus 1-2-3 codebase?
>
>This really makes me wonder what will happen to the old Lotus 1-2-3 codebase. Interestingly, in 2012 IBM discontinued its [IBM Lotus Symphony](http://en.wikipedia.org/wiki/IBM_Lotus_Symphony) suite, after which they donated the codebase of that product to the Apache Software Foundation, who then merged it into Apache OpenOffice.
>
>I'm not aware of any efforts to save the Lotus 1-2-3 codebase, but I think this would be immensely helpful to keep those old spreadsheet formats accessible. I don't know if this is something IBM would be willing to do (e.g. by releasing it as open source). This could also be interesting to of initiatives like the [Document Liberation Project](http://www.documentliberation.org/). If anyone has any info/additional thoughts on this please leave a comment!


[^1]: I wasn't able to locate these converters anywhere on Microsoft's website.

[^2]: Just in case anyone's wondering: these spreadsheets calculate a soil's saturated [hydraulic conductivity](http://en.wikipedia.org/wiki/Hydraulic_conductivity) from field measurement data using the [inverse auger hole method](http://www.samsamwater.com/library/DETERMINING_HYDRAULIC_CONDUCTIVITY_WITH_THE_INVERSED_AUGER_HOLE_AND_INFILTROMETER_METHODS.pdf).

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2014/10/29/quattro-pro-dos-obsolete-format-last/)
