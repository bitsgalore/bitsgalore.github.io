---
layout: post
title: JPEG quality estimation&#58; experiments with a modified ImageMagick heuristic
headImage: "/images/2024/10/bailey-1024.jpg"
headImageAltText: "Photograph of golden retriever dog Bailey sitting at a desk in front of a laptop, bashing her paws away at the laptop's keyboard while wearing a necktie."
description: "This post addresses some of the challenges I ran into while trying to estimate JPEG quality from some low quality images. It also proposes a tentative solution, that is based on a modified version of ImageMagick's JPEG quality heuristic."
tags: [JPEG, ImageMagick, ExifTool]
comment_id: 91
---

<figure class="image">
  <img src="{{ BASE_PATH }}{{ page.headImage }}" alt="{{ page.headImageAltText }}">
  <figcaption><a href="https://imgur.com/a/golden-baileys-story-pictures-XGli7">Bailey AKA the "I have no idea what I'm doing" dog</a>. License unknown.</figcaption>
</figure>


In this post I explore some of the challenges I ran into while trying to estimate the quality level of JPEG images. By quality level I mean the percentage (1-100) that expresses the [lossiness](https://en.wikipedia.org/wiki/Lossy_compression) that was applied by the encoder at the last "save" operation. Here, a value of 1 results in very aggressive compression with a lot of information loss (and thus a very low quality), whereas at 100 almost no information loss occurs at all[^4].

More specifically, I focus on problems with [ImageMagick](https://imagemagick.org/)'s JPEG quality heuristic, which become particularly apparent when applied to low quality images. I also propose a simple tentative solution, that applies some small changes to ImageMagick's heuristic.

<!-- more -->

## Context of this work

I'm currently working on an automated workflow for quality-checking scanned books and periodicals in PDF format. These PDFs are created by external suppliers for The Digital Library for Dutch Literature ([dbnl](https://www.dbnl.org/)), which has been managed by the KB since 2015.

Each book or periodical volume is scanned to PDF. For each publication, 2 versions are made:

1. A "production master" PDF with scans that are encoded at 85% JPEG quality.
2. A (relatively) small access PDF with scans encoded at 50% JPEG quality.

It's important to have some way of verifying the approximate quality of both versions. The production master serves as input for derived EPUB, XML and unformatted text versions. If the scans are compressed too heavily, this adversely affects these derived products. On the other hand, too little compression on the access PDFs will result in files that are impractically large for access.

## Estimating JPEG quality

Probably the best explainer on JPEG quality and its estimation is [this tutorial on Neal Krawetz's Fotoforensics site](https://fotoforensics.com/tutorial.php?tt=estq). The information under the "Estimating Quality" tab is particularly useful. I will return to this on various occasions later in this post.

## Estimating JPEG quality with ImageMagick

[ImageMagick](https://imagemagick.org/) is able to estimate the quality of a JPEG image. For example, let's create a test image at 70% quality:

```
convert -quality 70 wizard: wizard-70.jpg
```

We can then get an estimate of the JPEG quality using:

```
identify -format '%Q\n' wizard-70.jpg
```

Which results in:

```
70
```

However, when I ran this command on some of our access scans, the results were not what I expected. An example is [this image](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/dbnl/260761857-bdf17e14-c697-4f4a-b6d5-e3067e0afc08.jpg), which I extracted from one of our PDFs[^1]. According to ImageMagick[^2], this image is compressed at 92% quality. This JPEG was extracted from a small access PDF, for which I would expect a quality of 50% or less. Its file size is also much smaller than I would expect for a 92% quality image.

## Check with Fotoforensics

To verify this unexpected result I uploaded the image to Neal Krawetz's  [Fotoforensics](https://fotoforensics.com/) service. The [result is available here](https://fotoforensics.com/analysis.php?id=2e0a9f3203e35ece9a23c68c9e6dc7c908891372.353235&show=estq). Fotoforensics estimates the JPEG quality at a paltry 18%. So why does ImageMagick report a value of 92% here?

## ImageMagick uses "92" as a fallback value

From a cursory look at its source code, it seems that ImageMagick [uses 92 as a fallback value](https://github.com/ImageMagick/ImageMagick/blob/f5bdfdd62af7109ad105f8af4e28111e353edecd/MagickCore/property.c#L2725) if it cannot come up with a quality estimate. This makes the interpretation of its quality output needlessly difficult, since it's impossible to differentiate between images that have a true 92% quality, and images for which the quality cannot be established. I [created a ticket](https://github.com/ImageMagick/ImageMagick6/issues/260) on Github when I first came across this issue over a year ago, but thus far it hasn't been fixed.

Even if it was fixed, this still leaves the question *why* ImageMagick's quality heuristic is failing here in the first place. As I'm not proficient in *C*, I didn't pursue things any further when I first ran into this issue.

## ImageMagick's JPEG quality heuristic

This changed when I recently came across [this StackOverflow post](https://stackoverflow.com/a/75204019/1209004), which points to [a Python port of ImageMagick's JPEG quality heuristic](https://gist.github.com/eddy-geek/c0f01dc5401dc50a49a0a821cdc9b3e8#file-jpg_quality_pil_magick-py) by one "Edward O". It is based [on ImageMagicks original code](https://github.com/ImageMagick/ImageMagick6/blob/bf9bc7fee9f3cea9ab8557ad1573a57258eab95b/coders/jpeg.c#L925), and estimates the JPEG compression quality from the quantization tables. This immediately grabbed my attention, since my quality-checking workflow is also implemented in Python. The ability to estimate JPEG quality natively in Python would remove the need to wrap any external tools for this.

A quick test showed that it produced results that were identical to ImageMagick in most cases, but like ImageMagick, it failed to come up with a quality estimate for my problematic JPEG. To find out why, I incorporated Edwards's code into [this test script](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/jpegquality-im-original.py).

## How it works

The algorithm reads the image's quantization tables (usually 2), each of which is a list of 64 quantization coefficients. It first adds up all of these numbers, resulting in variable `qsum`. It then calculates `qvalue`, which is the sum of the quantization coefficients at 2 specific positions in each table (these are then summed for all quantization tables). The main "meat and potatoes" of the heuristic is this loop at the very end of the function:

```python
for i in range(100):
    if ((qvalue < hashes[i]) and (qsum < sums[i])):
        continue
    if (((qvalue <= hashes[i]) and (qsum <= sums[i])) or (i >= 50)):
        return i+1
```

For each iteration, `qvalue` and `qsum` are evaluated against the corresponding values in two hard-coded numerical lists (`hashes[i]` and `sums[i]`). Although I couldn't find any documentation, a little digging showed that the values in the `sums` and  `hashes` lists are derived from the "standard" quantization tables defined in Annex K of [the JPEG standard](http://www.w3.org/Graphics/JPEG/itu-t81.pdf)[^7]. The first *if* block makes sure that as long as `qvalue` is smaller than `hashes[i]` *and* `qsum` is smaller than `sums[i]`, the code will immediately jump to the next iteration, skipping the second *if* block. The second *if* block (which reports the quality estimate as `i+1`) is only evaluated if the test condition in the first block fails.

## Tracing all loop variables

To get a better impression of why the heuristic cannot come up with a meaningful quality estimate for my problematic JPEG, I added a line of code that prints out all variables at the start of each iteration. This gave the following output:

```
i: 0, qvalue: 586, hashes[i]:1020, qsum:24028, sums[i]32640
i: 1, qvalue: 586, hashes[i]:1015, qsum:24028, sums[i]32635
i: 2, qvalue: 586, hashes[i]:932, qsum:24028, sums[i]32266
i: 3, qvalue: 586, hashes[i]:848, qsum:24028, sums[i]31495
i: 4, qvalue: 586, hashes[i]:780, qsum:24028, sums[i]30665
i: 5, qvalue: 586, hashes[i]:735, qsum:24028, sums[i]29804
i: 6, qvalue: 586, hashes[i]:702, qsum:24028, sums[i]29146
i: 7, qvalue: 586, hashes[i]:679, qsum:24028, sums[i]28599
i: 8, qvalue: 586, hashes[i]:660, qsum:24028, sums[i]28104
i: 9, qvalue: 586, hashes[i]:645, qsum:24028, sums[i]27670
i: 10, qvalue: 586, hashes[i]:632, qsum:24028, sums[i]27225
i: 11, qvalue: 586, hashes[i]:623, qsum:24028, sums[i]26725
i: 12, qvalue: 586, hashes[i]:613, qsum:24028, sums[i]26210
i: 13, qvalue: 586, hashes[i]:607, qsum:24028, sums[i]25716
i: 14, qvalue: 586, hashes[i]:600, qsum:24028, sums[i]25240
i: 15, qvalue: 586, hashes[i]:594, qsum:24028, sums[i]24789
i: 16, qvalue: 586, hashes[i]:589, qsum:24028, sums[i]24373
i: 17, qvalue: 586, hashes[i]:585, qsum:24028, sums[i]23946
quality: -1
```

Here we see that at `i=17`, the value of `sums[i]` becomes smaller than `qsum`. As a result, we end up in the second *if* block. This reports the quality factor as `i+1`, but *only* if either of the following conditions is met:

1. both `qvalue` is smaller than or equal to `hashes[i]`, *and* `qsum` is smaller than or equal to `sums[i]`, *or*:
2. the value of `i` is larger than or equal to 50.

In this case, we see that `qvalue` is indeed smaller than `hashes[i]`, but `qsum` is *not* smaller than (or equal to) `sums[i]`. So the first condition is not met. Since `i` equals 17, the second condition is not met either, meaning that the code doesn't come up with a meaningful quality estimate, and reports the fallback value (-1) instead.

## Effect of quality threshold

It's not clear why the heuristic uses the quality threshold value of 50, although [Neal Krawetz points out](https://fotoforensics.com/tutorial.php?tt=estq) that "the JPEG Standard changes algorithms at quality values below 50%"[^6]. As a test, I tried  changing the threshold to 0:

```python
if (((qvalue <= hashes[i]) and (qsum <= sums[i])) or (i >= 0)):
    return i+1
```

With this change, the code now reports a quality value of 18. Incidentally this is identical to the *Fotoforensics* estimate.

## Modified ImageMagick heuristic

Setting the threshold of 0 effectively makes the first condition in the second *if* block superfluous, which means we could simplify things further to:

```python
for i in range(100):
    if ((qvalue < hashes[i]) and (qsum < sums[i])):
        continue
    else:
        return i+1
```

I also noticed that the original ImageMagick code includes a variable that indicates whether the quality estimate is "exact" or "approximate" ([here](https://github.com/ImageMagick/ImageMagick6/blob/bf9bc7fee9f3cea9ab8557ad1573a57258eab95b/coders/jpeg.c#L1030)). I initially assumed here that an "exact" match implies a perfect agreement with the standard JPEG quantization tables. This would be useful information to assess the accuracy of the quality estimate. I created [a test script with a modified version of the ImageMagick heuristic](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/jpegquality-im-modified.py) that incorporates the following changes to the ImageMagick heuristic:

1. Removal of the quality thresold.
2. Added reporting of the "exactness" flag, based on the original ImageMagick code.

## Tests with Pillow and ImageMagick JPEGs

As a first test I created a set of small test images with known quality values. I did this using Python's [Pillow library](https://python-pillow.org/) and [ImageMagick](https://imagemagick.org/). In both cases I generated test images with quality values 5, 10, 25, 50, 75 and 100, respectively. The images are available [here](https://github.com/KBNLresearch/jpeg-quality-demo/tree/main/images/im_pil). Running the script with my modified ImageMagick heuristic gave the following result:

|Q<sub>enc</sub>|Q<sub>est</sub>(Pillow)|Exact(Pillow)|Q<sub>est</sub>(IM)|Exact(IM)|
|:--|:--|:--|:--|:--|
|5|5|True|5|True|
|10|10|True|10|True|
|25|25|True|25|True|
|50|50|True|50|True|
|75|75|True|75|True|
|100|100|True|100|True|

Here *Q<sub>enc</sub>* is the encoding quality, and *Q<sub>est</sub>(Pillow)* and *Q<sub>est</sub>(IM)* represent the script's estimates for the Pillow and ImageMagick images, respectively. *Exact(Pillow)* and *Exact(IM)* represent the reported values of the "exactness" flag. The table shows that the script was able to reproduce the encoding quality with an "exact" match for all test images.

I also ran the script on some of the problematic JPEGs from our dbnl access PDFs. Here, it estimated the JPEG quality at 18%, but without an "exact" match. The JPEGs from the corresponding master PDFs resulted in a 84% quality estimate (which is slightly less than the expected quality of 85%), but again without an "exact" result.

Although the results of these (very limited!) tests look encouraging at first sight, this exercise left me with some doubts and reservations.

## Limitations of ImageMagick's heuristic

Most importantly, ImageMagick's heuristic is based on a comparison of *aggregated* coefficients of the image's quantization tables, which makes it potentially vulnerable to collisions. As an example, for any value of `qsum` (the sum of all coefficients in the quantization table), many possible combinations of quantization coefficients exist that will add up to the same value. The `qvalue` check (which is based on coefficients at specific positions in the quantization table) partially overcomes this, but it does this in a pretty crude way using yet another aggregate measure.

Another thing that bothered me, is that the reasoning behind certain aspects of ImageMagick's heuristic isn't entirely clear to me. Examples are the 50% threshold, the `qvalue` check (again) and the precise meaning of its "exact" vs "approximate" designation.

## Further down the JPEG rabbit hole

When I wrote the first draft of this post, I hoped it would provoke some response from people who are better versed than me in the inner workings of JPEG compression. As happens so often with these things, only a few days after publishing it I came across [this 2008 paper](https://www.sciencedirect.com/science/article/pii/S1742287608000285) which explains how quantization tables in JPEG work. This inspired me to do some more in-depth testing, and ultimately this resulted in an alternative, more straightforward quality estimation method. This is the subject of a [follow-up blog post]({{ BASE_PATH }}/2024/10/30/jpeg-quality-estimation-using-simple-least-squares-matching-of-quantization-tables), which is out now!

## Acknowledgment

Thanks are due to [Eddy O (AKA "eddygeek")](https://github.com/eddy-geek) for creating the [Python port of ImageMagick's JPEG quality heuristic](https://gist.github.com/eddy-geek/c0f01dc5401dc50a49a0a821cdc9b3e8) from which most of the results in this post are derived.

## Annex: other JPEG quality estimation tools and methods

While working on this, I came across a few alternative tools and methods for JPEG quality estimation. Here's a brief overview, which is largely for my own reference (but I imagine others may find it useful as well).

### ExifTool

[ExifTool](https://exiftool.org/) also reports a JPEG quality estimate if its `-JPEGQualityEstimate` option is invoked. For example:

```
exiftool -JPEGQualityEstimate test_im_050.jpg
```
Results in:

```
JPEG Quality Estimate           : 50
```

A peek at the [source code](https://github.com/exiftool/exiftool/blob/4981552ec9bf94a0b5a64a06919b5e4f797c208e/lib/Image/ExifTool/JPEGDigest.pm#L2447) shows it uses a ported version of ImageMagick's heuristic, so it alrgely has the same limitations. As an example, running it on my problematic JPEG results in:

```
JPEG Quality Estimate           : <unknown>
```

### Approximate Ratios method

This method is outlined on [Neal Krawetz’s Fotoforensics site](https://fotoforensics.com/tutorial.php?tt=estq), which also links to a [an inplementation in C](https://www.hackerfactor.com/src/jpegquality.c). A more detailed explanation can be found in Section 3.3.3 of his [Digital Image Analysis and Forensics whitepaper](https://blackhat.com/presentations/bh-dc-08/Krawetz/Whitepaper/bh-dc-08-krawetz-WP.pdf). As Krawetz explains, this method can become unreliable at quality values below 50%, so I didn't consider it suitable for my use case. 

### Cogranne method

A [2018 paper by Rémi Cogranne](https://arxiv.org/abs/1802.00992) describes an alternative method for estimating JPEG quality. It claims to overcome some of the limitations of established methods, such as the ImageMagick heuristic. However, as it is only valid for a quality factor greater than 49, this also isn't ideally suited to my use case.

## Scripts and test data

- [jpeg-quality-demo Github repository](https://github.com/KBNLresearch/jpeg-quality-demo) - Github repo with all scripts and test data that were used in this analysis. 
- [jpegquality-im-modified.py](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/jpegquality-im-modified.py) - Python implementation of modified ImageMagick heuristic.

## Revision history

- 24 October 2024: re-arranged introductory section, and added an explanation on the difference between quality level and image quality.

- 30 October 2024: revised earlier draft in preparation of follow-up post.

[^1]: I used [Poppler](https://poppler.freedesktop.org/)'s *pdfimages* tool to extract the images from the PDF, using the `-all` switch which ensures images are kept in their original format.

[^2]: I used ImageMagick 6.9.10-23.

[^4]: Note that the quality level does not necessarily reflect the *image quality*! As an example, if an image was first compressed at 20% quality and subsequently re-saved at 90%, the image quality will be very low (relative to the source image), despite the high quality level at the last save operation. So the quality level only says something about the *compression process* that was used on the last save.

[^6]: He mentions this in the context of the "Approximate Ratios" quality estimation method (which indeed becomes unreliable for low qualities). It's not clear to me if other methods such as the one used by ImageMagick are also affected by this.

[^7]: More precisely, each value in the `sums` lists represents the sum of all "standard" quantization coefficients for a particular quality level. Similarly, each value in the `hashes` lists represents a value of `qvalue` for a particular quality level in the "standard" tables.

