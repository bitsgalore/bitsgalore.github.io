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


In this post I explore some of the challenges I ran into while trying to estimate the quality level of JPEG images. By quality level I mean the percentage (1-100) that expresses the [lossiness](https://en.wikipedia.org/wiki/Lossy_compression) that was applied by the encoder at the last "save" operation. Here, a value of 1 results in very aggressive compression with a lot of information loss (and thus very low quality), whereas at 100 almost no information loss occurs at all[^4].

More specifically, I focus problems with [ImageMagick](https://imagemagick.org/)'s JPEG quality heuristic, which become particularly apparent when applied to low quality images. I also propose a simple tentative solution, that applies some small changes to ImageMagick's heuristic.

<!-- more -->

## Context of this work

I'm currently working on an automated workflow for quality-checking scanned books and periodicals in PDF format. These PDFs are created by external suppliers for The Digital Library for Dutch Literature ([dbnl](https://www.dbnl.org/)), which has been managed by the KB since 2015.

Each book or periodical volume is scanned to PDF. For each publication, 2 versions are made:

1. A "production master" PDF with scans that are encoded at 85% JPEG quality.
2. A (relatively) small access PDF with scans encoded at 50% JPEG quality.

It's important to have some way of verifying the approximate quality of both versions. The production master serves as input for derived EPUB, XML and unformatted text versions. If the scans are compressed too heavily, this adversely affects these derived products. On the other hand, too little compression on the access PDFs will result in files that are impractically large for access.

## Estimating JPEG quality

Probably the best explainer on JPEG quality and its estimation is [this tutorial on Neil Krawetz's Fotoforensics site](https://fotoforensics.com/tutorial.php?tt=estq). The information under the "Estimating Quality" tab is particularly useful. I will return to this on various occasions later in this post.

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

To verify this unexpected result I uploaded the image to Neil Krawetz's  [Fotoforensics](https://fotoforensics.com/) service. The [result is available here](https://fotoforensics.com/analysis.php?id=2e0a9f3203e35ece9a23c68c9e6dc7c908891372.353235&show=estq). Fotoforensics estimates the JPEG quality at a paltry 18%. So why does ImageMagick report a value of 92% here?

## ImageMagick uses "92" as a fallback value

From a cursory look at its source code, it seems that ImageMagick [uses 92 as a fallback value](https://github.com/ImageMagick/ImageMagick/blob/f5bdfdd62af7109ad105f8af4e28111e353edecd/MagickCore/property.c#L2725) if it cannot come up with a quality estimate. This makes the interpretation of its quality output needlessly difficult, since it's impossible to differentiate between images that have a true 92% quality, and images for which the quality cannot be established. I [created a ticket](https://github.com/ImageMagick/ImageMagick6/issues/260) on Github when I first came across this issue over a year ago, but thus far it hasn't been fixed.

Even if it was fixed, this still leaves the question *why* ImageMagick's quality heuristic is failing here in the first place. As I'm not proficient in *C*, I didn't pursue things any further when I first ran into this issue.

## ImageMagick's JPEG quality heuristic

This changed when I recently came across [this StackOverflow post](https://stackoverflow.com/a/75204019/1209004), which points to [a Python port of ImageMagick's JPEG quality heuristic](https://gist.github.com/eddy-geek/c0f01dc5401dc50a49a0a821cdc9b3e8#file-jpg_quality_pil_magick-py) by one "Edward O". It is based [on ImageMagicks original code](https://github.com/ImageMagick/ImageMagick6/blob/bf9bc7fee9f3cea9ab8557ad1573a57258eab95b/coders/jpeg.c#L925), and estimates the JPEG compression quality from the quantization tables. This immediately grabbed my attention, since my quality-checking workflow is also implemented in Python. The ability to estimate JPEG quality natively in Python would remove the need to wrap any external tools for this.

A quick test showed that it produced results that were identical to ImageMagick in most cases, but like ImageMagick, it failed to come up with a quality estimate for my problematic JPEG. To find out why, I incorporated Edwards's code into [this test script](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/test-jpegquality-im-original.py).

## How it works

The algorithm reads the image's quantization tables (usually 2), each of which is a list of 64 quantization values. It first adds up all of these numbers, resulting in variable `qsum`. It then calculates `qvalue`, which is the sum of the quantization values at 2 specific positions in each table (these are then summed for all quantization tables). The main "meat and potatoes" of the heuristic is this loop at the very end of the function:

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

It's not clear why the heuristic uses the quality threshold value of 50, although [Neil Krawetz points out](https://fotoforensics.com/tutorial.php?tt=estq) that "the JPEG Standard changes algorithms at quality values below 50%"[^6]. As a test, I tried  changing the threshold to 0:

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

I also noticed that the original ImageMagick code includes a variable that indicates whether the quality estimate is "exact" or "approximate" ([here](https://github.com/ImageMagick/ImageMagick6/blob/bf9bc7fee9f3cea9ab8557ad1573a57258eab95b/coders/jpeg.c#L1030)). I initially assumed here that an "exact" match implies a perfect agreement with the standard JPEG quantization tables. This would be useful information to assess the accuracy of the quality estimate. I created [a test script with a modified version of the ImageMagick heuristic](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/test-jpegquality-im-modified.py) that incorporates the following changes to the ImageMagick heuristic:

1. Removal of the quality thresold.
2. Added reporting of an "exactness" flag, based on the original ImageMagick code.

## Tests with Pillow and ImageMagick JPEGs

As a first test I created a set of small test images with known quality values. I did this using Python's [Pillow library](https://python-pillow.org/) and [ImageMagick](https://imagemagick.org/). In both cases I generated test images with quality values 5, 10, 25, 50, 75 and 100, respectively. The images are available [here](https://github.com/KBNLresearch/jpeg-quality-demo/tree/main/images/im_pil).

I then ran my script with my modified ImageMagick heuristic on these images. For all of them, the script successfully reproduced the encoding quality with an "exact" match.

I also ran the script on some of the problematic JPEGs from our dbnl  access PDFs. Here, it estimated the JPEG quality at 18%, but without an "exact" match. The JPEGs from the corresponding master PDFs resulted in a 84% quality estimate (which is slightly less than the expected quality of 85%), but again without an "exact" result.

The results of these (very limited!) tests look encouraging at first sight. Despite this, the exercise left me with some doubts and reservations.

## Limitations of ImageMagick's heuristic

Most importantly, ImageMagick's heuristic is based on a comparison of *aggregated* values of the image's quantization tables, which makes it potentially vulnerable to collisions. As an example, for any value of `qsum` (the sum of all values in the quantization table), many possible combinations of quantization values exist that will add up to the same value. The `qvalue` check (which is based on values at specific positions in the quantization table) partially overcomes this, but it does this in a pretty crude way using yet another aggregate measure.

## Direct comparison with standard quantization tables

The ImageMagick heuristic is based on the "standard" quantization tables that are defined in the JPEG standard, and its overall objective seems to be to match an image's quantization tables with the most similar "standard" quantization tables. Since the quality level of each of these "standard" tables is known, this then provides the quality estimate.

ImageMagick's heuristic does all this in a rather indirect way, possibly to avoid the computational cost of comparing the image's quantization tables against 100 or 200 "standard" tables. As an alternative, I explored a different, more straightforward approach that does *exactly* this[^3].

## Scaling of "standard" tables for each quality

To understand how this works, it's first important to know that Annex K in the [JPEG standard](http://www.w3.org/Graphics/JPEG/itu-t81.pdf) describes two "standard" "quantization tables for luminance and chrominance. Here's the luminance table:

|:--|:--|:--|:--|:--|:--|:--|:--|
|16|11|10|16|24|40|51|61|
|12|12|14|19|26|58|60|55|
|14|13|16|24|40|57|69|56|
|14|17|22|29|51|87|80|62|
|18|22|37|56|68|109|103|77|
|24|35|55|64|81|104|113|92|
|49|64|78|87|103|121|120|101|
|72|92|95|98|112|100|103|99]

This is the base table that is valid for quality level 50. The tables for all other quality levels can be derived from this base table using Equations 1 and 2 from [this paper by Kornblum (2008)](https://www.sciencedirect.com/science/article/pii/S1742287608000285). First, for each quality level *Q* we can calculate a corresponding scaling factor *S*:

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow>
    <mi>S</mi>
    <mo>=</mo>
    <mfrac>
      <mrow>
        <mn>5000</mn>
      </mrow>
      <mrow>
        <mi>Q</mi>
      </mrow>
    </mfrac>
    <mspace width="6em"/>
    <mo>(</mo>
    <mi>Q</mi>
    <mo>&lt;</mo>
    <mn>50</mn>
    <mo>)</mo>
  </mrow>
</math>

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow>
    <mi>S</mi>
    <mo>=</mo>
    <mn>200</mn>
    <mo>&middot;</mo>
    <mi>Q</mi>
    <mspace width="5em"/>
    <mo>(</mo>
    <mi>Q</mi>
    <mo>&ge;</mo>
    <mn>50</mn>
    <mo>)</mo>
  </mrow>
</math>

*S* is then used to calculate scaled quantization values *T<sub>s</sub>* from the base values *T<sub>b</sub>* using the following equation:

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow>
    <msub>
      <mi>T</mi>
      <mi>s</mi>
    </msub>
    <mo>[</mo>
    <mi>i</mi>
    <mo>]</mo>
    <mo>=</mo>
    <mo>min</mo>
    <mo>(</mo>
    <mo>&#x230A;</mo>
    <mfrac>
      <mrow>
        <mi>S</mi>
        <mo>&middot;</mo>
        <msub>
          <mi>T</mi>
          <mi>b</mi>
        </msub>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>+</mo>
        <mn>50</mn>
      </mrow>
      <mrow>
        <mn>100</mn>
      </mrow>
    </mfrac>
    <mo>&#x230B;</mo>
    <mo>,</mo>
    <mn>1</mn>
    <mo>)</mo>
  </mrow>
</math>

Where *i* is the *i*th element in the table. Note the [floor brackets](https://en.wikipedia.org/wiki/Floor_and_ceiling_functions), which mean the expression inside them is rounded down to the nearest integer number. For 8-bit quantization tables (which is the most common situation) these values also need to be capped at a maximum of 255.
 
As an example, applying these equations to *Q=75* results in a scaling factor *S* of 15000, and the quantization values become: 

|:--|:--|:--|:--|:--|:--|:--|:--|
|8|6|5|8|12|20|26|31|
|6|6|7|10|13|29|30|28|
|7|7|8|12|20|29|35|28|
|7|9|11|15|26|44|40|31|
|9|11|19|28|34|55|52|39|
|12|18|28|32|41|52|57|46|
|25|32|39|44|52|61|60|51|
|36|46|48|49|56|50|52|50|

This way it is possible to calculate the quantization tables for all 100 quality levels. For the chrominance tables the same procedure can be used.

## Estimating quality from scaled tables

For a given JPEG file, the quality can then be estimated by comparing its quantization tables against each of the scaled tables that are derived from the standard tables. As a basis for this comparison we can calculate, for each quality level, the sum of squared errors between the values in the image's quantization table and the corresponding values in the scaled standard table. For an image with 2 quantization tables (one for luminance and another one for chrominance) this is given by:

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow>
    <mi>SSE</mi>
    <mo>=</mo>
    <munderover>
      <mo>&sum;</mo>
      <mrow>
        <mi>i</mi>
        <mo>=</mo>
        <mn>1</mn>
      </mrow>
      <mn>64</mn>
    </munderover>
    <mo>(</mo>
    <msup>
      <mrow>
        <mo>(</mo>
        <msub>
          <mi>T</mi>
          <mi>lum</mi>
        </msub>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>-</mo>
        <msub>
          <mi>T</mi>
          <mrow>
            <mi>lum</mi>
            <mo>,</mo>
            <mi>s</mi>
          </mrow>
        </msub>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>)</mo>
      </mrow>
      <mn>2</mn>
    </msup>
    <mo>+</mo>
    <msup>
      <mrow>
        <mo>(</mo>
        <msub>
          <mi>T</mi>
          <mi>chrom</mi>
        </msub>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>-</mo>
        <msub>
          <mi>T</mi>
          <mrow>
            <mi>chrom</mi>
            <mo>,</mo>
            <mi>s</mi>
          </mrow>
        </msub>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>)</mo>
      </mrow>
      <mn>2</mn>
    </msup>
    <mo>)</mo>
  </mrow>
</math>

Here, *T<sub>lum</sub>* and *T<sub>chrom</sub>* represent the values for luminance and chrominance from the image's quantization table, and *T<sub>lum, s</sub>* and *T<sub>chrom, s</sub>* are the corresponding values from the (scaled) standard tables.

Repeating this for all quality levels results in 100 *SSE* values. The quality level with the smallest *SSE* value is then our best estimate for the quality of the image. An *SSE* value of exactly 0 means the image uses the standard JPEG quantization tables. In that case we can be confident that our estimate is the exact quality level at which the image was compressed. Larger values indicate the use of non-standard quantization tables, in which case the quality estimate may be less accurate.

## Characterizing similarity to standard tables

For the latter case, it would be useful to have some measure of *how much* the quantization tables deviate from the best matching standard table. By itself, *SSE* is not a good measure of this. The first reason for this is, that is influenced by the number of quantization tables in the image. We could remove this influence by transforming the *SSE* value to a root mean squared error:

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow>
    <mi>RMSE</mi>
    <mo>=</mo>
    <msqrt>
      <mfrac>
        <mrow>
          <mi>SSE</mi>
        </mrow>
        <mrow>
          <mi>tables</mi>
          <mo>&middot;</mo>
          <mn>64</mn>
        </mrow>
      </mfrac>
    </msqrt>
  </mrow>
</math>

Here, *tables* is the number of quantization tables (which is either 1 or 2). However, the interpretation of these *RMSE* values is complicated by the fact that the values in the quantization tables are significantly larger for lower quality levels. In practice this has the effect that the *RMSE* values are generally much larger at low quality levels relative to higher quality levels, even for images that have a similar overall "fit" to a standard quantization table.

One "goodness of fit" measure that does not have these drawbacks is the [Nash–Sutcliffe efficiency coefficient (NSE)](https://en.wikipedia.org/wiki/Nash%E2%80%93Sutcliffe_model_efficiency_coefficient). This measure is mostly used in the field of hydrology to characterize how well a hydrological simulation model reproduces observed river discharges. Rewritten for our quantization tables it can be calculated as: 

<math xmlns="http://www.w3.org/1998/Math/MathML">
  <mrow>
    <mi>NSE</mi>
    <mo>=</mo>
    <mn>1</mn>
    <mo>-</mo>
    <mfrac>
    <mrow>
    <munderover>
      <mo>&sum;</mo>
      <mrow>
        <mi>i</mi>
        <mo>=</mo>
        <mn>1</mn>
      </mrow>
      <mi>N</mi>
    </munderover>
    <msup>
      <mrow>
        <mo>(</mo>
          <mi>T</mi>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>-</mo>
        <msub>
          <mi>T</mi>
            <mi>s</mi>
        </msub>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>)</mo>
      </mrow>
      <mn>2</mn>
    </msup>
    </mrow>
    <mrow>
    <munderover>
      <mo>&sum;</mo>
      <mrow>
        <mi>i</mi>
        <mo>=</mo>
        <mn>1</mn>
      </mrow>
      <mi>N</mi>
    </munderover>
    <msup>
      <mrow>
        <mo>(</mo>
          <mi>T</mi>
        <mo>[</mo>
        <mi>i</mi>
        <mo>]</mo>
        <mo>-</mo>
        <msub>
          <mi>T</mi>
          <mi>avg</mi>
        </msub>
        <mo>)</mo>
      </mrow>
      <mn>2</mn>
    </msup>
    </mrow>
    </mfrac>
  </mrow>
</math>

Here *T[i]* represents the *i*th value from the image's quantization tables, and *T<sub>s</sub>* is the corresponding value from the (scaled) standard table. *N* is the total number of values in the quantization tables. Note that, unlike the  *SSE* equation, this includes both the luminance and chrominance values. Finally, *T<sub>avg</sub>* is the mean of all values in the image's quantization tables. The interpretation of *NSE* is quite straightforward:

- A value of 1 indicates a perfect agreement between the image quantization tables and the corresponding standard tables.
- For a value of 0, the standard tables are as good (or rather, bad) an approximation of the image's quantization tables as *T<sub>avg</sub>*.
- Negative values indicate an extremely poor agreement.

As an example, the below scatterplot shows the quantization values (*T*) from [one of out dbnl master images](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/dbnl/mul-master.jpg), plotted against the corresponding values (*Ts*) from the best matching standard table. It also shows the line of perfect agreement (red, dashed), the quality estimate, the root mean squared error, and the Nash-Sutcliffe Efficiency:

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2024/10/mul-master-scatter.png" alt="Scatterplot of T against Ts for file mul-master.jpg, with Quality = 84%, RMSE = 1.057 and NSE = 0.997.">
</figure>

The plot shows that, aside from one outlier, the values from the standard quantization table closely approximate the image's quantization table. This is reflected by the *NSE* value, which is close to 1. Now compare this with the plot I made for [the corresponding access image](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/dbnl/mul-access.jpg):

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2024/10/mul-access-scatter.png" alt="Scatterplot of T against Ts for file mul-access.jpg, with Quality = 18%, RMSE = 6.244 and NSE = 0.999.">
</figure>

Visually, the agreement between the standard quantization table values and those of the image looks very similar to the previous plot. But note how the *RMSE* value is much larger here, which is caused by the much larger overall values in the quantization tables. However, this doesn't have any effect on *NSE*, which is even marginally closer to 1 than for the master image.

By contrast, things are very different for [this image](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/image-177.jpg):

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2024/10/image-177-scatter.png" alt="Scatterplot of T against Ts for file image-177-scatter.png, with Quality = 81%, RMSE = 20.509 and NSE = 0.732.">
</figure>

The plot shows that the standard JPEG tables are a relatively poor approximation here, and this is refected by the low *NSE* value. A similar case is [this image](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/sample-jpg-files-sample-4.jpg): 

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2024/10/sample-jpg-files-sample-4-scatter.png" alt="Scatterplot of T against Ts for file sample-jpg-files-sample-4-scatter.png, with Quality = 89%, RMSE = 12.294 and NSE = 0.734.">
</figure>

Despite the different quality estimate and *RMSE* value, this visually looks like it's in the same ballpark as the previous image, and the similar *NSE* value confirms this.

## Python implementation

I created [a test script with a Python implementation](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/test-jpegquality-tablematch.py) of the above procedure. 


## Comparison of different methods

|File|Q<br>(im)|Q<br>(im, mod)|Exact?<br>(im, mod)|Q<br>table)|RMSE<br>(table)|NSE<br>(table)|
|:--|:--|:--|:--|:--|:--|:--|
|[psgradient.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/psgradient.jpg)|92|92|False|93|4.438|0.747|
|[hopper_16bit_qtables.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/hopper_16bit_qtables.jpg)|-1|1|False|13|0.795|1.0|
|[image-177.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/image-177.jpg)|63|63|True|81|20.509|0.732|
|[image-98.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/image-98.jpg)|63|63|True|81|20.509|0.732|
|[jpeg420exif.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/jpeg420exif.jpg)|90|90|False|89|0.424|0.999|
|[jpeg422jfif.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/jpeg422jfif.jpg)|96|96|False|97|0.0|1.0|
|[jpeg444.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/jpeg444.jpg)|60|60|False|75|0.0|1.0|
|[sample-birch-400x300.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/sample-birch-400x300.jpg)|95|95|False|94|3.188|0.838|
|[sample-jpg-files-sample-4.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/sample-jpg-files-sample-4.jpg)|78|78|True|89|12.294|0.734|
|[tapedeck1.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/tapedeck1.jpg)|90|90|False|93|0.753|0.992|
|[tapedeck2.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/tapedeck2.jpg)|92|92|False|95|0.421|0.996|

Note the result for [jpeg444.jpg](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/misc/jpeg444.jpg): both the original and modified ImageMagick heuristics estimate the quality at 60%, with no "exact" match. By contrast, my table-based method came up with a quality of 75%, with a perfect match with the standard JPEG quantization tables. This is comfirmed by plotting the values from the quantization tables against the standard values:

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2024/10/jpeg444-scatter.png" alt="Scatterplot of T against Ts for file jpeg444-scatter.png, with Quality = 75%, RMSE = 0.0 and NSE = 1.0.">
</figure>

I even triple-checked the result by uploading the image to FotoForensics, which [also comes up with 75% quality, and an exact match with the standard tables](https://fotoforensics.com/analysis.php?id=2e8c6fc55fefbdf2e3b96e9c531d3d24b7b5ea16.5667).


## Tests

As a first test, I ran my test script on the [Pillow and ImageMagick JPEGs](https://github.com/KBNLresearch/jpeg-quality-demo/tree/main/images/im_pil). For each image this successfully returned the encoding quality, with an *RMSE* value of exactly 0.

For the problematic JPEGs from my access PDF ([example here](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/dbnl/mul-access.jpg)), the script reported a quality of 18 (which is identical to the modified ImageMagick heuristic), and an *RMSE* of 6.244. JPEGs from the corresponding master PDF ([example](https://github.com/KBNLresearch/jpeg-quality-demo/blob/main/images/dbnl/mul-master.jpg)) resulted (again) in an estimated quality of 84, with and *RMSE* of 1.057. This illustrates the usefulness of the *RMSE* value. The value for the corresponding master JPEG is much smaller than the access one, which means its quantization tables are more similar to the "standard" JPEG tables. Therefore, the corresponding quality estimate is most likely more accurate than the estimate for the access JPEG.


[this file](https://yavuzceliker.github.io/sample-images/image-98.jpg) results in:

```
quality: 81, RMS Error: 20.509
```

ImageMagick:

```
identify -format '%Q\n' image-98.jpg
```

Result:

```
63
```

ExifTool:

```
exiftool -JPEGQualityEstimate image-98.jpg
```

Result:

```
JPEG Quality Estimate           : 64
```

[Fotoforensics result](https://fotoforensics.com/analysis.php?id=3f271d3383ea2984b461620f2d54075dc5ec26da.37769):

```
JPEG last saved at 81% quality (estimated) 
```

[jpeg444](https://www.w3.org/MarkUp/Test/xhtml-print/20050519/tests/jpeg444.jpg)



[Image with 16 bit quantization tables](https://raw.githubusercontent.com/python-pillow/Pillow/refs/heads/main/Tests/images/hopper_16bit_qtables.jpg).

IM: -1

IM mod: 1, exact = False

Table match: 13, RMSE = 0.795

Fotoforensics result:

<https://fotoforensics.com/analysis.php?id=079bf03e3187859bf2bdf0b28c341d3b75ad8442.2044>

```
JPEG last saved at 13% quality (estimated) 
```

Sample files in:

<https://github.com/KBNLresearch/jpeg-quality-demo/tree/main/images/misc>

## Performance

One potential drawback of the direct table match method is that it is computationally less efficient than ImageMagick's heuristic: for each image, the analysis typically involves 200 comparisons of 64-element tables. Out of interest I did a little performance test using [this collection of 700 JPEGs](https://github.com/yavuzceliker/sample-images). I analyzed these files with my scripts for the original and modified ImageMagick heuristics, and the table match method.

For each script, I first ran this command to empty my cache memory:

```
sudo sysctl vm.drop_caches=3
```

I then ran the script like this:

```
(time python3 ./jpeg-quality-demo/test-jpegquality-tablematch.py ./sample-images/images/*.jpg > sample-images.txt) 2> time-tablematch.txt
```

The "time" command results in 3 performance metrics, The most important of which are:

- "real" - the actual amount of time passed between starting the script and its termination.
- "user" - actual CPU time used in executing the process[^8].

Below table shows these metrics for the three scripts[^5]:

|Method|time (real)|time (user)|
|:--|:--|:--|
|im_original|0m7,624s|0m3,321s|
|im_modified|0m7,215s|0m2,968s|
|tablematch|0m13,134s|0m8,673s|

This shows the table match method is almost 3 times slower than the ImageMagick heuristics in terms of "user" time, and about 2 times slower in terms of "real" time. This translates to an average processing time of 0.01 to 0.2 s per file. Therefore, the reduced performance shouldn't be any problem in practical terms.


I hope this post will provoke some reactions from people who are more familiar with the relevant technical details.

Mention JPEGs with 16 bit quantization tables!

## Let me know your feedback!

For all of the reasons mentioned above, any feedback on the modified ImageMagick heuristic would be highly appreciated! I've created a [Github repo](https://github.com/KBNLresearch/jpeg-quality-demo) with all the scripts and data I used for my analyses. Hopefully this will encourage others to have another look at this, or even come up with something better!

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

This method is outlined on [Neil Krawetz’s Fotoforensics site](https://fotoforensics.com/tutorial.php?tt=estq), which also links to a [an inplementation in C](https://www.hackerfactor.com/src/jpegquality.c). A more detailed explanation can be found in Section 3.3.3 of his [Digital Image Analysis and Forensics whitepaper](https://blackhat.com/presentations/bh-dc-08/Krawetz/Whitepaper/bh-dc-08-krawetz-WP.pdf). As Krawetz explains, this method can become unreliable at quality values below 50%, so I didn't consider it suitable for my use case. 

### Cogranne method

A [2018 paper by Rémi Cogranne](https://arxiv.org/abs/1802.00992) describes an alternative method for estimating JPEG quality. It claims to overcome some of the limitations of established methods, such as the ImageMagick heuristic. However, as it is only valid for a quality factor greater than 49, this also isn't ideally suited to my use case.

## Scripts and test data

All scripts and test data that were used in this analysis are available from this Git repository:

<https://github.com/KBNLresearch/jpeg-quality-demo>

## Revision history

- 24 October 2024: re-arranged introductory section, and added an explanation on the difference between quality level and image quality.

[^1]: I used [Poppler](https://poppler.freedesktop.org/)'s *pdfimages* tool to extract the images from the PDF, using the `-all` switch which ensures images are kept in their original format.

[^2]: I used ImageMagick 6.9.10-23.

[^3]: From its description I think this corresponds to the "Approximate Quantization Tables" method that is mentioned on (and used by) [Neil Krawetz's Fotoforensics site](https://fotoforensics.com/tutorial.php?tt=estq) (but the site doesn't provide any details about the implementation).

[^4]: Note that the quality level does not necessarily reflect the *image quality*! As an example, if an image was first compressed at 20% quality and subsequently re-saved at 90%, the image quality will be very low (relative to the source image), despite the high quality level at the last save operation. So the quality level only says something about the *compression process* that was used on the last save.

[^5]: I ran the test on a pretty low spec machine with an Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz with 4 cores.

[^6]: He mentions this in the context of the "Approximate Ratios" quality estimation method (which indeed becomes unreliable for low qualities). It's not clear to me if other methods such as the one used by ImageMagick are also affected by this.

[^7]: More precisely, each value in the `sums` lists represents the sum of all "standard" quantization values for a particular quality level. Similarly, each value in the `hashes` lists represents a value of `qvalue` for a particular quality level in the "standard" tables.

[^8]: For a more in-depth explanation of these metrics see: <https://stackoverflow.com/a/556411/1209004>