---
layout: post
title: Response to report on JPEG 2000 expert round table 
tags: [jpeg-2000,JP2]
comment_id: 32
---

Today my attention was caught by [this report of an "Expert round table" on JPEG2000 and Digitisation](https://www.townswebarchiving.com/2015/10/jpeg2000-and-digitisation-expert-round-table/), which was published on the  TownsWeb Archiving blog. Although the report as a whole is quite balanced, it's unfortunate that it provides fuel to some long-running myths about JPEG 2000 not supporting fully lossless compression. Since I wasn't able to leave a comment on the Townweb blog itself, I turned my response into this small blog post.

<!-- more -->

## Visually lossless vs mathematically lossless

For a start, **Dave Thompson** says about JPEG 2000: 

> Its wavelet based technology means that it can be used in a compressed format which is visually lossless. 

While this statement is true, it may confuse some people, since  it doesn't mention that *mathematically* lossless compression is supported as well (in which case decoding the image returns the *exact* pixel values as they were prior to compression). Also, "visually lossless" compression is just a lossy compression that results in compression errors that are not detectable to the eye (see also the definition [here](http://www.digitizationguidelines.gov/term.php?term=compressionvisuallylossless)). This is not unique to JPEG 2000, and there's nothing that stops you from implementing visually lossless compression with "ordinary" JPEG, even though this would be pretty inefficient when compared to JPEG 2000.

## Suitability for preservation

More seriously, **Paul Sugden** questions JPEG 2000's suitability as a preservation format. The reason behind his concerns is a conference talk he attended:

> I attended a conference last year where an expert seemed to demonstrate that it was not possible to convert a JPEG2000 image precisely back to the original lossless TIFF file from which it was created. His example showed that after the retro conversion the file size of the newly created TIFF was different to the original TIFF as captured, 
and there were also visible differences between the images (albeit minor differences). 

From this he concludes:

> In my opinion, a file format that does not offer accurate retro-conversion back to precisely the image that was originally captured certainly cannot be seen as a reliable preservation format.

This is quite a bold statement, especially given that it is based on only *one* report (of which Sugden doesn't provide any specific information). Having done some pretty extensive testing with different encoders and decoders, I suspect the *real* problem here is just some bug in a specific encoder. I've encountered similar problems myself (e.g. see [here](https://github.com/bitsgalore/jpegToLosslessJP2)), but those are just software bugs, which say *nothing about the format itself*, nor about its  general suitability as a preservation format. 

So, without any supporting evidence I don't see much that justifies the sweeping generalisations that are made in the blog post (but I'm open to be proven wrong!).

## Verifying lossless image migrations

To be clear: JPEG 2000 fully supports completely lossless compression. The "losslessness" is also easy to verify using pixel-wise comparisons between source and destinaton images (e.g. using ImageMagick's "compare" tool, some examples [here](https://github.com/bitsgalore/jpegToLosslessJP2)). Not all encoders handle things like embedded metadata and ICC profiles equally well. Also, JPEG 2000's baseline JP2 format has some restrictions on the embedding of ICC profiles, which can be a problem in very specific cases (e.g. when the ICC profile  of the source TIFF you want to convert is not supported by JP2). 
