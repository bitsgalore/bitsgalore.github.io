---
layout: post
title: Six ways to decode a lossy JP2
tags: [jpeg-2000,JP2]
comment_id: 42
---

Some time ago Will Palmer, Peter May and Peter Cliff of the British Library published a really interesting [paper that investigated three different JPEG 2000 codecs](http://www.scape-project.eu/publication/palmer-ipres2013), and their effects on image quality in response to lossy compression. Most remarkably, their analysis revealed differences not only in the way these codecs *encode* (compress) an image, but also in the *decoding* phase. In other words: reading the same lossy JP2 produced different results depending on which implementation was used to decode it.
 
A limitation of the paper's methodology is that it obscures the individual effects of the encoding and decoding components, since both are essentially lumped in the analysis. Thus, it's not clear how much of the observed degradation in image quality is caused by the compression, and how much by the decoding. This made me wonder how similar the *decode* results of different codecs really are.

<!-- more -->

## An experiment

To find out, I ran a simple experiment:

1. Encode a TIFF image to JP2.
2. Decode the JP2 back to TIFF using different decoders.
3. Compare the decode results using some similarity measure.

## Codecs used

I used the following codecs:

* [Kakadu](http://www.kakadusoftware.com/) v7.2.2 (kakadu)
* [OpenJPEG](http://www.openjpeg.org/) 2.0 (opj20)
* [ImageMagick](http://www.imagemagick.org/) 6.8.9-8 (im)
* [GraphicsMagick](http://www.graphicsmagick.org/) 1.3.18 (gm)
* [IrfanView](http://www.irfanview.com/) 4.35 with JPEG2000 plugin 4.33 (irfan)

Note that GraphicsMagick still uses the [JasPer](http://www.ece.uvic.ca/~frodo/jasper/) library for JPEG 2000. ImageMagick now uses OpenJPEG (older versions used JasPer). IrfanViews's JPEG 2000 plugin is made by [LuraTech](http://www.luratech.com/en/products/luratech-jp2-irfanview-plug-in/).

## Creating the JP2

First I compressed my source TIFF (a grayscale newspaper page) to a lossy JP2 with a compression ratio about about 4:1. For this example I used OpenJPEG, with the following command line:

```bash
opj_compress -i krant.tif -o krant_oj_4.jp2 -r 4 -I -p RPCL -n 7 -c [256,256],[256,256],[256,256],[256,256],[256,256],[256,256],[256,256] -b 64,64
```

## Decoding the JP2

Next I decoded this image back to TIFF using the aforementioned codecs. I used the following command lines:

|Codec|Command line|
|:--|:--|
|**opj20**|`opj_decompress -i krant_oj_4.jp2 -o krant_oj_4_oj.tif`|
|**kakadu**|`kdu_expand -i krant_oj_4.jp2 -o krant_oj_4_kdu.tif`|
|**kakadu-precise**|`kdu_expand -i krant_oj_4.jp2 -o krant_oj_4_kdu_precise.tif -precise`|
|**irfan**|Used GUI|
|**im**|`convert krant_oj_4.jp2 krant_oj_4_im.tif`|
|**gm**|`gm convert krant_oj_4.jp2 krant_oj_4_gm.tif`|

This resulted in 6 images. Note that I ran Kakadu twice: once using the default settings, and also with the *-precise* switch, which "forces the use of 32-bit representations".

## Overall image quality 

As a first analysis step I computed the overall peak signal to noise ratio (PSNR) for each decoded image, relative to the source TIFF:

|Decoder|PSNR|
|:--|:--|
|**opj20**|48.08|
|**kakadu**|48.01|
|**kakadu-precise**|48.08|
|**irfan**|48.08|
|**im**|48.08|
|**gm**|48.07|

So *relative to the source image* these results are only marginally different. 

## Similarity of decoded images

But let's have a closer look at how similar the different decoded images are. I did this by computing PSNR values of all possible decoder pairs. This produced the following matrix:

|Decoder|opj20|kakadu|kakadu-precise|irfan|im|gm|
|:--|:--|:--|:--|:--|
|**opj20**|-|57.52|78.53|79.17|96.35|64.43|
|**kakadu**|57.52|-|57.51|57.52|57.52|57.23|
|**kakadu-precise**|78.53|57.51|-|79.00|78.53|64.52|
|**irfan**|79.17|57.52|79.00|-|79.18|64.44|
|**im**|96.35|57.52|78.53|79.18|-|64.43|
|**gm**|64.43|57.23|64.52|64.44|64.43|-|

Note that, unlike the table in the previous section, these PSNR values are only a measure of the *similarity* between the different decoder results. They don't directly say anything about *quality* (since we're not comparing against the source image). Interestingly, the PSNR values in the matrix show two clear groups:

* **Group A**: all combinations of OpenJPEG, Irfanview, ImageMagick and Kakadu in *precise* mode, all with a PSNR of > 78 dB.
* **Group B**: all remaining decoder combinations, with a PSNR of < 64 dB.

What this means is that OpenJPEG, Irfanview, ImageMagick and Kakadu in *precise* mode all decode the image in a similar way, whereas Kakadu (default mode) and GraphicsMagick behave differently. Another way of looking at this is to count the pixels that have different values for each combination. This yields up to 2 % different pixels for all combinations in group *A*, and about 12 % in group *B*. Finally, we can look at the peak absolute error value (PAE) of each combination, which is the maximum value difference for any pixel in the image. This figure was 1 pixel level (0.4 % of the full range) in both groups.

I also repeated the above procedure for a small RGB image. In this case I used Kakadu as the encoder. The decoding results of that experiment showed the same overall pattern, although the differences between groups *A* and *B* were even more pronounced, with PAE values in group *B* reaching up to 3 pixel values (1.2 % of full range) for some decoder combinations.    

## What does this say about decoding quality?

It would be tempting to conclude from this that the codecs that make up group *A* provide better quality decoding than the others (GraphicsMagick, Kakadu in default mode). If this were true, one would expect that the overall PSNR values *relative to the source TIFF* (see previous table) would be higher for those codecs. But the values in the table are only marginally different. Also, in the test on the small RGB image, running Kakadu in *precise* mode *lowered* the overall PSNR value (although by a tiny amount). Such small effects could be due to chance, and for a conclusive answer one would need to repeat the experiment for a large number of images, and test the PSNR differences for statistical significance (as was done in the BL analysis).

I'm still somewhat surprised that even in group *A* the decoding results aren't *identical*, but I suspect this has something to do with small rounding errors that arise during the decode process (maybe someone with a better understanding of the mathematical intricacies of JPEG 2000 decoding can comment on this). Overall, these results suggest that the errors that are introduced by the decode step are very small when compared against the encode errors.

## Conclusions

OpenJPEG, (recent versions of) ImageMagick, IrfanView and Kakadu in *precise* mode all produce similar results when decoding lossily compressed JP2s, whereas Kakadu in default mode and GraphicsMagick (which uses the JasPer library) behave differently. These differences are very small when compared to the errors that are introduced by the encoding step, but for critical decode applications (migrate lossy JP2 to something else) they may still be significant. As both ImageMagick and GraphicsMagick are often used for calculating image (quality) statistics, the observed differences also affect the outcome of such analyses: calculating PSNR for a JP2 with ImageMagick and GraphicsMagick results in two different outcomes!

For *losslessy* compressed JP2s, the decode results for all tested codecs are 100% identical[^2].

This tentative analysis does not support any conclusions on which decoders are 'better'. That would need additional tests with more images. I don't have time for that myself, but I'd be happy to see others have a go at this!

## Link

[William Palmer, Peter May and Peter Cliff: An Analysis of Contemporary JPEG2000 Codecs for Image Format Migration (Proceedings, iPres 2013)](http://www.scape-project.eu/publication/palmer-ipres2013)

[^2]: Identical in terms of pixel values; for this analysis I didn't look at things such as embedded ICC profiles, [which not all encoders/decoders handle well](http://wiki.opf-labs.org/display/TR/Handling+of+ICC+profiles).

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2014/09/26/six-ways-decode-lossy-jp2/)
