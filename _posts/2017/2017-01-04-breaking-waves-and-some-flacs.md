---
layout: post
title: Breaking WAVEs (and some FLACs too)
tags: [WAVE,FLAC,optical-media,JHOVE]
comment_id: 26
---

At the KB we have a large collection of offline optical media. Most of these are CD-ROMs, but we also have a sizeable proportion of audio CDs. We're currently in the process of designing a workflow for stabilising the contents of these materials using disk imaging. For audio CDs this involves 'ripping' the tracks to audio files. Since the workflow will be automated to a high degree, basic quality checks on the created audio files are needed. In particular, we want to be sure that the created audio files are complete, as it is possible that some hardware failure during the ripping process could result in truncated or otherwise incomplete files.

To get a better idea of what software tool(s) are best suitable for this task, I created a small dataset of audio files which I deliberately damaged. I subsequently ran each of these files through a set of candidate tools, and then looked which tools were able to detect the faulty files. The first half of this blog post focuses on the [*WAVE*](http://fileformats.archiveteam.org/wiki/WAVE) format; the second half covers the [*FLAC*](http://fileformats.archiveteam.org/wiki/FLAC) format (at the moment we haven't decided on which format to use yet).

<!-- more -->

## *WAVE* dataset

For the *WAVE* dataset I started out with a [small, intact *WAVE* file](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01.wav). Using a Hex editor I then made the following derivatives of this file:

* [frogs-01-last-byte-missing.wav](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01-last-byte-missing.wav) - one byte is missing at the end of the file
* [frogs-01-last-2032-bytes-missing.wav](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01-last-2032-bytes-missing.wav) - a chunk of  2032 bytes is missing at the end of the file
* [frogs-01-byte-missing-at-offset-811537.wav](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01-byte-missing-at-offset-811537.wav) - one byte is missing at offset 811537

## Candidate tools, *WAVE*

The candidate tools I used to analyse the *WAVE* files are:

* [**jhove**](http://jhove.openpreservation.org/) includes a [*WAVE* validation module](http://jhove.openpreservation.org/modules/wave/), which makes it an obvious choice. The tested version is  1.14.6, 2016-05-12.
* [**shntool**](http://www.etree.org/shnutils/shntool/) is a "multi-purpose WAVE data processing and reporting utility". It was first released in 2000. The tested version is 3.0.7.
* [**ffmpeg**](https://ffmpeg.org/) is a popular conversion tool for audio and video formats. The tested version is 3.2.2.
* [**mediainfo**](https://mediaarea.net/en/MediaInfo) is a widely-used feature extraction tool for audiovisual files. The tested version is v0.7.81.

Note that of the above tools, only Jhove and Shntool are designed to detect problems in *WAVE* files. Both Ffmpeg and Mediainfo were primarily designed for other purposes (format conversion and technical metadata extraction), and they were *not* designed to detect defective files! I included these tools here mainly because they are widely used, and I was curious whether they would throw up anything interesting in case of defective files[^2].
I ran the tools with the following command-line arguments (replacing "foo.wav" with the actual file name):

### Jhove

```bash
jhove -m WAVE-hul foo.wav
```

### Shntool

```bash
shntool info foo.wav
```

### Ffmpeg

```bash
ffmpeg -v error -i foo.wav -f null -
```

### Mediainfo

```bash
mediainfo foo.wav
```

I automated this using a simple [shell script](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/runtoolsWAV.sh) that runs each tool on all files, and then writes the output to a set of text files.

## Results, *WAVE*

The full output results of each tool can be found [here](https://github.com/KBNLresearch/detectDamagedAudio/tree/master/outputWAV). 

### Jhove

The 'Status' field in Jhove's output summarises the validation outcome. Here are the results for each file: 

|File|Result|
|:--|:--|
|frogs-01.wav|Status: Well-Formed and valid|
|frogs-01-last-byte-missing.wav|Status: Well-Formed and valid|
|frogs-01-last-2032-bytes-missing.wav|Status: Well-Formed and valid|
|frogs-01-byte-missing-at-offset-811537.wav|Status: Well-Formed and valid|

So, Jhove was unable to detect *any* of the damaged files at all! 

### Shntool

Shntool checks a *WAVE* on six criteria, which are listed in its output under 'Possible problems': 

```
Possible problems:
    File contains ID3v2 tag:    no
    Data chunk block-aligned:   yes
    Inconsistent header:        no
    File probably truncated:    no
    Junk appended to file:      no
    Odd data size has pad byte: n/a
```

The thing to watch here is the 'File probably truncated' item:

|File|Result|
|:--|:--|
|frogs-01.wav|File probably truncated:    no|
|frogs-01-last-byte-missing.wav|File probably truncated:    yes (missing 1 byte)|
|frogs-01-last-2032-bytes-missing.wav|File probably truncated:    yes (missing 2032 bytes|
|frogs-01-byte-missing-at-offset-811537.wav|File probably truncated:    yes (missing 1 byte)|

So, Shntool was able to detect all damaged files.

### Ffmpeg

For our Ffmpeg call we monitor any errors that are sent to the standard error stream. The results:

|File|result|
|:--|:--|
|frogs-01.wav|-|
|frogs-01-last-byte-missing.wav|[pcm_s16le @ 0x3545380] Invalid PCM packet, data has size 3 but at least a size of 4 was expected<br>Error while decoding stream #0:0: Invalid data found when processing input|
|frogs-01-last-2032-bytes-missing.wav|-|
|frogs-01-byte-missing-at-offset-811537.wav|[pcm_s16le @ 0x2768380] Invalid PCM packet, data has size 3 but at least a size of 4 was expected<br>Error while decoding stream #0:0: Invalid data found when processing input|

Interestingly, Ffmpeg reports an error for both files that have 1 byte missing, but it doesn't for the file that has 2023 bytes missing. This suggests that Ffmpeg is *not* suitable for detecting broken *WAVE* files.

### Mediainfo

Mediainfo didn't report errors or warnings for any of these files. This is not surprising, but it does 
confirm that Mediainfo cannot be used for detecting broken *WAVE* files.

## *FLAC* dataset

Analogous to the *WAVE* dataset, I started out with a [small, intact *FLAC* file](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01.flac), which I then butchered into the following derivative files:

* [frogs-01-last-byte-missing.flac](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01-last-byte-missing.flac) - one byte is missing at the end of the file
* [frogs-01-last-1000-bytes-missing.flac](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01-last-1000-bytes-missing.flac) - a chunk of  1000 bytes is missing at the end of the file
* [frogs-01-byte-missing-at-offset-651202.flac](https://github.com/KBNLresearch/detectDamagedAudio/blob/master/data/frogs-01-byte-missing-at-offset-651202.flac) - one byte is missing at offset 651202

## Candidate tools, *FLAC*

The set of candidate tools is identical to the one used for the *WAVE* analysis, with two exceptions:

* [**flac**](https://xiph.org/flac/) is the reference implementation of the *FLAC* format. The tested version is 1.3.0.
* Since Jhove does not include a *FLAC* module, it was not used.

### Flac
    
The Flac tool is able to encode audio to *FLAC*, and decode and analyze *FLAC* files. For this tests I ran it with the * -t* (or *--test*) option:

```bash
flac -t foo.flac
```

This decodes a *FLAC* without writing the decoded data to a file. Any errors during the decoding process are reported to the standard error stream.

## Results, *FLAC*

The full output results of each tool can be found [here](https://github.com/KBNLresearch/detectDamagedAudio/tree/master/outputFLAC). 

### Shntool

Even though Shntool supports *FLAC*, it was not able to detect the missing data in any of the files:

|File|Result|
|:--|:--|
|frogs-01.flac|File probably truncated:    no|
|frogs-01-last-byte-missing.flac|File probably truncated:    no|
|frogs-01-last-1000-bytes-missing.flac|File probably truncated:    no|
|frogs-01-byte-missing-at-offset-651202.flac|File probably truncated:    no|

So, Shntool does not provide any meaningful information on whether a *FLAC* is damaged.

### Ffmpeg

Here are the results for Ffmpeg:

|File|Result|
|:--|:--|
|frogs-01.flac|-|
|frogs-01-last-byte-missing.flac|[flac @ 0x294b860] overread: 1<br>Error while decoding stream #0:0: Invalid data found when processing input|
|frogs-01-last-1000-bytes-missing.flac|[flac @ 0x3c5d860] overread: 1<br>Error while decoding stream #0:0: Invalid data found when processing input|
|frogs-01-byte-missing-at-offset-651202.flac|[flac @ 0x279faa0] overread: 1<br>Error while decoding stream #0:0: Invalid data found when processing input|

So, Ffmpeg was able to identify all damaged *FLAC*s.

### Mediainfo

Similar to the *WAVE* results, Mediainfo again didn't report errors or warnings for any of these files. 

### Flac

Finally the results for the Flac tool:

|File|Result|
|:--|:--|
|frogs-01.flac|-|
|frogs-01-last-byte-missing.flac|ERROR while decoding data<br>state = FLAC__STREAM_DECODER_END_OF_STREAM|
|frogs-01-last-1000-bytes-missing.flac|ERROR while decoding data<br>state = FLAC__STREAM_DECODER_END_OF_STREAM|
|frogs-01-byte-missing-at-offset-651202.flac|ERROR while decoding data<br>state = FLAC__STREAM_DECODER_READ_FRAME|

So, the Flac tool was able to identify all defective files[^1].

## Conclusion

Out of the candidate tools considered here, only Shntool was able to identify all damaged *WAVE* files in this experiment. As a result, this (ancient!) tool still appears to be the best choice for detecting damaged *WAVE* files. Surpringly, Jhove was unable to detect *any* of the damaged files at all, and is probably best avoided for this particular purpose. For *FLAC*, both the Flac tool (*FLAC* reference implementation) and Ffmpeg were able to detect all damaged files, and both appear to be suitable tools.

## Dataset and scripts

All example files, scripts and raw tool output are available here:

<https://github.com/KBNLresearch/detectDamagedAudio>

## Post scriptum: update on MediaInfo and MediaConch

In response to this post the developers of MediaInfo [added support for detecting truncated *WAVE* files](https://github.com/MediaArea/MediaInfoLib/pull/352). This should cover all of the damaged *WAVE* files presented here. Moreover, their Twitter account announced that [detection of *FLAC* flaws](https://twitter.com/MediaArea_Net/status/817303297786867712) is planned for the [MediaConch tool](https://mediaarea.net/MediaConch/), but that they are looking for sponsors for this.

[^1]: On a side note, I noticed that the error stream of the Flac tool sometimes contained a sequence of 21 non-printable '0x08' (backspace0 characters. This is probably a bug. 
[^2]: Also, [this thread on *superuser.com*](http://superuser.com/a/100290/681049) recommends Ffmpeg for checking the integrity of video files.


<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2017/01/04/breaking-waves-and-some-flacs/)
