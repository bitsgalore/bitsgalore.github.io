---
layout: post
title: Generating lossy access JP2s from lossless preservation masters
tags: [JP2, jpeg-2000, jpylyzer]
comment_id: 78
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2022/03/jm-cote-wiki.jpg" alt="Plumbers Tool Box">
  <figcaption><a href="https://commons.wikimedia.org/wiki/File:France_in_XXI_Century._Intencive_breeding.jpg">Intensive Breeding</a> by Jean Marc Cote, Public domain, via Wikimedia Commons.</figcaption>
</figure>

At the KB we've been using JP2 ([JPEG 2000](https://en.wikipedia.org/wiki/JPEG_2000) Part 1) as our primary image format for digitised newspapers, books and periodicals since 2007. The digitisation work is contracted out to external vendors, who supply the digitised pages as losslessly compressed preservation masters, as well as lossily compressed access images that are used within the [Delpher](https://www.delpher.nl/) platform.

Right now the KB is in the process of [migrating its digital collections to a new preservation system](https://web.archive.org/web/20210215160819/https://www.kb.nl/en/news/2021/dutch-national-library-steps-into-a-new-future-of-digital-archiving). This prompted the question whether it would be feasible to generate access JP2s from the preservation masters in-house at some point in the future, using software that runs inside the preservation system[^6]. As a first step towards answering that question, I created some simple proof of concept workflows, using three different JPEG 2000 codecs. I then tested these workflows with preservation master images from our collection. The main objective of this work was to find a workflow that both meets our current digitisation requirements, and is also sufficiently performant.

<!-- more -->

## Master and access requirements

The following table lists the requirements of our preservation master and access JP2s:

|Parameter|Value (master)|Value (access)|
|:--|:--|:--|
|File format|JP2 (JPEG 2000 Part 1)|JP2 (JPEG 2000 Part 1)|
|Compression type|Reversible 5-3 wavelet filter|Irreversible 7-9 wavelet filter|
|Colour transform|Yes (only for colour images)|Yes (only for colour images)|
|Number of decomposition levels|5|5|
|Progression order |RPCL|RPCL|
|Tile size |1024 x 1024|1024 x 1024|
|Code block size|64 x 64 (2<sup>6</sup> x 2<sup>6</sup>)|64 x 64 (2<sup>6</sup> x 2<sup>6</sup>)|
|Precinct size	|256 x 256 (2<sup>8</sup>) for 2 highest resolution levels; 128 x 128 (2<sup>7</sup>) for remaining resolution levels|256 x 256 (2<sup>8</sup>) for 2 highest resolution levels; 128 x 128 (2<sup>7</sup>) for remaining resolution levels|
|Number of quality layers|11|8|
|Target compression ratio layers|2560:1 [1] ; 1280:1 [2] ;  640:1 [3] ; 320:1 [4] ; 160:1 [5] ; 80:1 [6] ; 40:1 [7] ; 20:1 [8] ; 10:1 [9] ; 5:1 [10] ; - [11]|2560:1 [1] ; 1280:1 [2] ;  640:1 [3] ; 320:1 [4] ; 160:1 [5] ; 80:1 [6] ; 40:1 [7] ; 20:1 [8]|
|Error resilience|Start-of-packet headers; end-of-packet headers; segmentation symbols|Start-of-packet headers; end-of-packet headers; segmentation symbols|
|Sampling rate|Stored in "Capture Resolution" fields|Stored in "Capture Resolution" fields|
|Capture metadata|Embedded as XMP metadata in XML box|Embedded as XMP metadata in XML box|

As the table shows, most parameters are identical in both cases, except:

1. The preservation masters are compressed losslessly, whereas for the access images irreversible (lossy) compression is used, using a fixed compression ratio of 20:1.
2. The preservation masters contain 11 quality layers, whereas the access images only contain 8 quality layers.

## Deriving the access image

In order to derive an access image from a preservation master, two approaches are possible:

1. Create a "subset" from the preservation master that only contains the lower 8 quality layers (i.e. discarding the highest 3 layers).
2. Do a full decode of the preservation master (e.g. to TIFF), and then re-compress the result to lossy JP2.

The main advantage of the first ("subset") approach is its computational efficiency: it only involves some simple reformatting of data from the source image's codestream, without any need to decode or compress the image data. I explored this in [this 2013 blog post]({{ BASE_PATH }}/2013/08/19/optimising-archival-jp2s-derivation-access-copies), and at the time I was able to make it work with Kakadu's "transcode" tool and Aware's JPEG 20000 SDK[^3]. However, its success depends largely on the correct implementation of the quality layers in the preservation masters. For example, if the 8th quality layer in a preservation master was accidentally compressed at some other compression ratio than the expected 20:1 value, the resulting access JP2 could be (much) smaller or larger than expected. Complicating things further, even though [jpylyzer](https://jpylyzer.openpreservation.org/) will tell you both the overall compression ratio of a JP2 as well as the number of quality layers, it does not provide any information about the compression ratios of individual layers[^1].

Because of this, I only explored the full decode + re-compress approach here. Although computationally less efficient than the "subset" approach, it has the advantage that the result is independent of the implementation of the quality layers in the preservation masters.

## Test environment

For my tests I used an ordinary desktop PC with 4 CPU cores, an Intel i5-6500 CPU (3.20GHz) processor and 12 GB RAM. The operating system was Linux Mint 20.1 Ulyssa (which is based op Ubuntu Focal Fossa 20.04).

## Codecs

I initially planned to create a small proof of concept workflow based on [Kakadu](https://kakadusoftware.com/), as I already had some old test scripts for compressing TIFF images to JP2s that follow the KB's master and access requirements. Then my colleague Sam Alloing suggested to have a look at the [Grok codec](https://github.com/GrokImageCompression/grok). Although I had been aware of Grok for some time, I had never got around to take it for a spin, mainly because I haven't been working much on anyting related to JPEG 20000 for the past few years. Since Grok is a fork of [OpenJPEG](https://www.openjpeg.org/), which the KB already uses [to decode JP2 images on the Delpher platform](https://lab.kb.nl/about-us/blog/kb-national-library-netherlands-adopts-openjpeg-delpher-and-more-0), it then made sense to include OpenJPEG as well. So, in the end I used:

- OpenJPEG 2.4.2
- Grok 9.7.3
- Kakadu 7.9[^4]

I compiled both Grok and OpenJPEG from the source code. For Kakadu I used the pre-compiled demonstration binaries.

## Test procedure

For each of the three codecs, I created a simple Bash script that takes an input and an output directory as its arguments. For each JP2 image in the input directory, the script goes through the following steps:

1. Decode (uncompress) the JP2 to uncompressed TIFF.
2. Compress the TIFF to lossy JP2, using (to the maximum extent posssible) the KB's access JP2 requirements.
3. Delete the TIFF file.

Once all input JP2s have been processed, the script then runs the [jprofile tool](https://github.com/KBNLresearch/jprofile/) on the output directory. Jprofile (which uses [jpylyzer](https://jpylyzer.openpreservation.org/) under its hood) uses [Schematron rules](https://www.bitsgalore.org/2012/09/04/automated-assessment-jp2-against-technical-profile) to verify to what extent the generated JP2s conform to the KB access requirements.

The test scripts (which also contain the encoding parameter values for each codec) can be found here:

|Codec|Link to script|
|:--|:--|
|OpenJPEG|<https://github.com/KBNLresearch/jp2totiff/blob/master/mastertoaccess-opj.sh>|
|Grok|<https://github.com/KBNLresearch/jp2totiff/blob/master/mastertoaccess-grok.sh>|
|Kakadu|<https://github.com/KBNLresearch/jp2totiff/blob/master/mastertoaccess-kdu.sh>|

## Performance

I ran each of the scripts on a directory with 26 preservation master JP2s (144 MB) from the KB's collection of digitised books. Before running any of the scripts, I used the following command to empty my machine's cache memory:

```bash
sudo sysctl vm.drop_caches=3
```
I then used the operating system's built-in ["time" tool](https://linux.die.net/man/1/time) to measure the processing time needed by each of the scripts:

```bash
(time ~/kb/jp2totiff/mastertoaccess-grok.sh ./master-1 ./access-1-grok) 2> time-grok.txt
```

The main metrics provided by this command are:

- "real" - the actual amount of time passed between starting the script and its termination.
- "user" - The sum of the processing times of each of the individual processors.

Below table shows the performance statistics for the three scripts:

|Codec|time (real)|time (user)|
|:--|:--|:--|
|OpenJPEG|0m50.715s|1m20.497s|
|Grok|0m22.143s|1m1.308s|
|Kakadu|0m25.507s|0m48.990s|

It's worth noting that each of these figures encompasses a full decode-encode cycle, with some additional overhead added by jprofile, and system commands that remove the temorary TIFF files. I was surprised to see that at 22 seconds, the Grok-based script was even (marginally) faster than the Kakadu-based one, which clocks in at 26 seconds[^5]. The script that uses OpenJPEG is considerably slower at 51 seconds.

##  Conformance to KB access requirements

The next table summarises the jprofile analysis, by listing the deviations from the KB acces requirements for each codec: 

|Codec|Deviations from KB access requirements|
|:--|:--|
|OpenJPEG|XML box missing, resolution box missing, ICC profile missing|
|Grok|XML box missing|
|Kakadu|-|

The OpenJPEG JP2s fall short on three aspects. An XML box with XMP metadata, a resolution box, and an ICC profile are all missing. This is not surprising, as OpenJPEG simply doesn't support these features at this stage. In the Grok JP2s, only the expected XML box is missing. This is because Grok wraps XMP metadata in a so-called "UUID box". This behaviour is consistent with the [ISO/IEC base media file format](https://en.wikipedia.org/wiki/ISO/IEC_base_media_file_format), and is supported by e.g. Exiftool and jpylyzer. Only the Kakadu JP2s are 100% compliant with the requirements. However, since the exact location of XMP metadata doesn't really matter for access, both the Kakadu and the Grok JP2s would be satisfactory for our purposes.

## Conclusions

Although based on only a small sample dataset, this proof of concept demonstrates that both Grok and Kakadu would be suitable for generating lossy access JP2s from our preservation masters. The performance of both codecs turned out to be comparable for the test data used. This means that with Grok we now have an open-source codec that is both sufficiently feature-rich and performant to be a viable alternative to commercial codecs like Kakadu. One potential hurdle for some users might be Grok's build process, which can be slightly involved because it requires very recent versions of [CMake](https://cmake.org/) and [gcc](https://gcc.gnu.org/). However, using [Grok's documentation](https://github.com/KBNLresearch/jp2totiff/blob/master/doc/grok-installation.md) and [these useful additional instructions by Harvard's Bill Comstock](https://wiki.harvard.edu/confluence/display/DigitalImaging/Installing+OpenJPEG+on+Windows+10%2C+Linux%2C+and+MacOS) I found the process easier than expected in the end. I've [documented the full build and installation process that worked for me here](https://github.com/KBNLresearch/jp2totiff/blob/master/doc/grok-installation.md).

## Acknowledgements

Thanks are due to Grok developer Aaron Boxer for fixing two small issues I ran into while running my Grok tests, and Sam Alloing for suggesting to look into Grok.

## Revision history

- 5 July 2022 -  re-ran performance test with added `-threads` option for OpenJPEG, as suggested by Aaron Boxer in the comments

## Further resources

- [Git repository with test scripts](https://github.com/KBNLresearch/jp2totiff)
- [My Grok build and installation instructions](https://github.com/KBNLresearch/jp2totiff/blob/master/doc/grok-installation.md)
- [Bill Comstock, "Installing OpenJPEG (and Grok) on Windows 10, Linux, and MacOS"](https://wiki.harvard.edu/confluence/display/DigitalImaging/Installing+OpenJPEG+on+Windows+10%2C+Linux%2C+and+MacOS)
- [Optimising archival JP2s for the derivation of access copies]({{ BASE_PATH }}/2013/08/19/optimising-archival-jp2s-derivation-access-copies)
- [Jprofile - Automated JP2 profiling for digitisation batches](https://github.com/KBNLresearch/jprofile)

[^1]: Adding this functionality to jpylyzer would require much more in-depth parsing of the codestream data than is currently the case.

[^3]: As of 2022, Aware appears to have switched its focus to the development of biometrical software, and its website does not mention the JPEG 2000 SDK anymore.

[^4]: Note that this a pretty old version.

[^5]: These figures are not 100% comparable, because the Kakadu-based script includes an additional processing step to extract embedded metadata from the source file using ExifTool (Grok does this automatically at the codec level).

[^6]: To be completely clear, at this stage this work is just an exploration of something we might do at some time in the future (or possibly not at all); there are no plans to actually implement this yet.