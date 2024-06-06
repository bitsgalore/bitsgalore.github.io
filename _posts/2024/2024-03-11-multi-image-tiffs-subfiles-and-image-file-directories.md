---
layout: post
title: Multi-image TIFFs, subfiles and image file directories
image: "/images/2024/03/confused-muddled-illogical-957696-1024.jpg"
description: "This post discusses the impact of having multiple images indide a TIFF on preservation workflows, and also provides some suggestions on how to identify such files."
tags: [TIFF, ImageMagick, JHOVE, ExifTool, preservation-risks]
comment_id: 90
---

<figure class="image">
  <img src="{{ BASE_PATH }}{{ page.image }}" alt="Photograph that shows a hammer that is used to smash a screw into a piece of wood. On the left is a nail that is partially pushed into the same piece of wood, with an adjustable wrench immediately next to it.">
  <figcaption><a href="https://jenikirbyhistory.getarchive.net/media/confused-muddled-illogical-957696">"Confused, muddled, illogical"</a>. Used under <a href="https://web.archive.org/web/20170727004823/https://pixabay.com/en/service/license/">Pixabay License</a>.</figcaption>
</figure>

The KB has been using JP2 (JPEG 2000 Part 1) as the primary file format for its mass-digitisation activities for over 15 years now. Nevertheless, we still use uncompressed TIFF for a few collections. At the moment there's an ongoing discussion about whether we should migrate those to JP2 as well at some point to save storage costs. Last week I ran a small test on a selection of TIFFs from those collections. I first converted them to JP2, and then verified whether no information got lost during the conversion. This resulted in some unexpected surprises, which turned out to be caused by the presence of thumbnail images in some of the source TIFFs. This post discusses the impact of having multiple images indide a TIFF on preservation workflows, and also provides some suggestions on how to identify such files.

<!-- more -->

## TIFF to JP2 workflow

For my tests, I took a selection of 20 test targets in uncompressed TIFF format from five different digitisation batches. For each of these targets, I then:

1. converted the source TIFF to lossless JP2 with Kakadu;
2. converted the JP2 from step 1 back to uncompressed TIFF;
3. compared the pixel values of the TIFF from step 2 against those of the source TIFF[^1] with [ImageMagick's *compare* tool](https://imagemagick.org/script/compare.php).

For the pixel comparison in step 3, I used the following general command (using ImageMagick version 6.9.10-23):

```bash
compare -quiet -metric AE source.tif fromjp2.tif null:
```

This command computes the absolute error count (i.e. the number of different pixels) between both images[^2], which is printed to standard error ([stderr](https://en.wikipedia.org/wiki/Standard_streams)). The `-quiet` switch is included to suppress warning messages (these are also printed to stderr, and can muddle up the error count output).

## ImageMagick reports changed pixel values

Since we used lossless compression in our JP2 conversion step, the expected outcome here is that all pixel values are unchanged, and ImageMagick's AE output metric is exactly zero for all images. For 12 images this was indeed the case. However, for 8 images the AE metric indicated that nearly all pixels had changed. An additional check showed that the computed [peak signal-to-noise ratio](https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio) (PSNR) values for these images were also extremely poor, ranging between 3 and 13. This was definitely unexpected!

A subsequent visual inspection of the affected images in [Gimp](https://www.gimp.org/) did not show any obvious degradation. As an additional check, I also loaded one pair of images in Gimp as separate layers, and then [subtracted one layer from the other one](https://www.reddit.com/r/GIMP/comments/32dgfq/comment/cqa95dl/). This resulted in an image where all RGB values were exactly 0, which is only possible if both images are identical. So what's going on here?

## Subfiles and image file directories

My first idea was to analyze all source TIFFs with [ExifTool](https://exiftool.org/). For each source TIFF, I used the following general command (using ExifTool version 12.60):

```bash
exiftool -X source.tif > source.xml
```

An inspection of the resulting output files showed that all problematic TIFFs contain two separate "image file directories". Within TIFF, it is possible to bundle multiple images in one single file. This is most commonly done by defining each image as a "subfile", whose properties are described by a corresponding "image file directory" (*IFD*). The individual images can be pages in a multi-page document, or different representations of the same image (e.g. a full size image and a low resolution thumbnail). In our particular case, we have this:

``` xml
 <IFD1:SubfileType>Reduced-resolution image</IFD1:SubfileType>
 <IFD1:ImageWidth>160</IFD1:ImageWidth>
 <IFD1:ImageHeight>126</IFD1:ImageHeight>
 <IFD1:BitsPerSample>8 8 8</IFD1:BitsPerSample>
 <IFD1:Compression>Uncompressed</IFD1:Compression>
 <IFD1:PhotometricInterpretation>RGB</IFD1:PhotometricInterpretation>
 <IFD1:StripOffsets>6510</IFD1:StripOffsets>
 <IFD1:SamplesPerPixel>3</IFD1:SamplesPerPixel>
 <IFD1:RowsPerStrip>126</IFD1:RowsPerStrip>
 <IFD1:StripByteCounts>60480</IFD1:StripByteCounts>
 <IFD1:PlanarConfiguration>Chunky</IFD1:PlanarConfiguration>
 <IFD1:ThumbnailTIFF>(Binary data 60696 bytes, use -b option to extract)</IFD1:ThumbnailTIFF>
```

Here, all "IFD1" tags describe a subfile that is a reduced resolution thumbnail image (this is indicated by the *SubfileType* tag and its value). The full-resolution image is described by "IFD0", which is the first image file directory:

```xml
 <IFD0:ImageWidth>9458</IFD0:ImageWidth>
 <IFD0:ImageHeight>7429</IFD0:ImageHeight>
 <IFD0:BitsPerSample>8 8 8</IFD0:BitsPerSample>
 <IFD0:Compression>Uncompressed</IFD0:Compression>
 <IFD0:PhotometricInterpretation>RGB</IFD0:PhotometricInterpretation>
 <IFD0:Make>Leaf</IFD0:Make>
 <IFD0:Model>Leaf Aptus-II 12R(LI201033   )/Other</IFD0:Model>
 <IFD0:StripOffsets>(Binary data 72 bytes, use -b option to extract)</IFD0:StripOffsets>
 <IFD0:Orientation>Horizontal (normal)</IFD0:Orientation>
 <IFD0:SamplesPerPixel>3</IFD0:SamplesPerPixel>
 <IFD0:RowsPerStrip>1024</IFD0:RowsPerStrip>
 <IFD0:StripByteCounts>(Binary data 70 bytes, use -b option to extract)</IFD0:StripByteCounts>
 <IFD0:XResolution>300</IFD0:XResolution>
 <IFD0:YResolution>300</IFD0:YResolution>
 <IFD0:PlanarConfiguration>Chunky</IFD0:PlanarConfiguration>
 <IFD0:ResolutionUnit>inches</IFD0:ResolutionUnit>
 <IFD0:Software>Capture One 8 Macintosh</IFD0:Software>
```

## SubIFDs

For completeness, I should mention here that an alternative way to store multiple images in a TIFF, is by combining them in one single subfile. The corresponding image file directory then contains *SubIFD* child tags, each of which describes an individual image. The *SubIFD* tag is defined in the [Adobe PageMaker 6.0 TIFF Technical Notes](https://www.awaresystems.be/imaging/tiff/specification/TIFFPM6.pdf). According to the documentation of the [LibTIFF](https://libtiff.gitlab.io/libtiff/) library, ["SubIFD chains are rarely supported"](https://libtiff.gitlab.io/libtiff/multi_page.html). I also couldn't find any example files that use them.

## Remove thumbnail with ExifTool

To test whether the presence of the thumbnail is indeed the cause of my weird pixel check results, I removed it from (a copy of) one of the problematic source TIFFs with the following ExifTool command:

```bash
exiftool -ifd1:all= -m source.tif
```

Note that this removes *IFD1* (which is the second *IFD*) and its associated data.

When I re-ran ImageMagick's *compare* tool with the modified file, the output was as expected: the absolute error count was 0, and the PSNR value "inf"[^3].

## Pixel compare with unmodified source TIFFs

Even though removing the thumbnail gets the job done, it adds another processing step to our workflow. Fortunately, a [little digging](https://github.com/ImageMagick/ImageMagick/discussions/3279#discussioncomment-387226) revealed that it's possible to explicitly address individual images in a multi-image file in ImageMagick. If we (only) want to use the first image in both our input images, we can call the compare tool like this: 

```bash
compare -quiet -metric AE source.tif[0] fromjp2.tif[0] null:
```

Note that the number between square brackets sets the index of the subfile that is used for the comparison. Applying this to our unmodified source TIFFs, this indeed results in an absolute error count of 0 for all image pairs. I implemented the above command in a simple [test script](https://github.com/KBNLresearch/jp2totiff/blob/master/pixelCheck-im.sh) that compares two directory trees with TIFF images, and reports the results to a comma-delimited file.

## ImageMagick reads last subfile by default

As a final test, I deliberately instructed ImageMagick to use the second subfile (i.e. the thumbnail) of the source TIFF as a basis for the comparison:

```bash
compare -quiet -metric AE source.tif[1] fromjp2.tif[0] null:
```

The resulting (large) AE value is identical to what is reported if the subfiles are not set by the user at all. So, it seems that by default ImageMagick uses the *last* subfile of each input image, and that this is the root cause of the unexpected behaviour!

## Are multi-image TIFFs a preservation risk?

ImageMagick's behaviour made me wonder to what extent the presence of multiple images poses a preservation risk. In our specific example, the second IFD only represents a thumbnail, and losing this in a format migration isn't such a big deal.

Section 7 of the [TIFF 6.0 specification](https://web.archive.org/web/20180810205359/https://www.adobe.io/content/udp/en/open/standards/TIFF/_jcr_content/contentbody/download/file.res/TIFF6.pdf) describes how baseline TIFF readers should handle files with multiple images:

> TIFF readers must be prepared for multiple images (subfiles) per TIFF file,
> although they are not required to do anything with images after the first one. 

For the thumbnail case, I can think of two potential problems:

1. If the writer application mistakenly wrote the thumbnail as the first subfile (and the full resolution images as the last one), a migration tool that is based on a conforming reader would then migrate the thumbnail, and ignore the full resolution image data.
2. If a migration tool mistakenly reads the last subfile instead of the first one, this would also result in the loss of data.

I expect the above problems are largely hypothetical, especially since TIFF has been around for such a long time. But I wouldn't completely rule them out either. Importantly, such errors could pass unnoticed by image comparison tools like ImageMagick's *compare* if they use the wrong subfile as a comparison basis. Fortunately, the extremely small size of the resulting image files would be a pretty obvious clue that something is amiss.

TIFFs with multiple images that represent pages in a multi-page document could pose a larger risk. I don't really expect any of these to show up in our own collections[^4], but the situation might be different for other collections.

Finally, TIFFs with subfiles that in turn contain multiple images through the use of *SubIFD* tags add another layer of complexity.

So, depending on your specific situation, it might be prudent to check your TIFFs for the presence of multiple images before doing any preservation actions on them, like a migration to some other format.

## Detection of multi-image TIFFs

In order to detect the presence of multiple images, we have a couple of options.

### ExifTool

With [ExifTool](https://exiftool.org/), we can use the aforementioned command:

```bash
exiftool -X source.tif > source.xml
```

As an example, here's some output for a 6-page TIFF ([file available here](https://www.leadtools.com/support/forum/resource.ashx?a=544&b=1)):

```xml
 <IFD0:SubfileType>Full-resolution image</IFD0:SubfileType>
 <IFD0:ImageWidth>363</IFD0:ImageWidth>
 <IFD0:ImageHeight>7429</IFD0:ImageHeight>
  ::
 <IFD1:SubfileType>Full-resolution image</IFD1:SubfileType>
 <IFD1:ImageWidth>363</IFD1:ImageWidth>
 <IFD1:ImageHeight>382</IFD1:ImageHeight>
  ::
  ::
 <IFD5:SubfileType>Full-resolution image</IFD5:SubfileType>
 <IFD5:ImageWidth>363</IFD5:ImageWidth>
 <IFD5:ImageHeight>382</IFD5:ImageHeight>
  ::
```

If the output contains one or more `<IFD#:SubfileType>` tags (where `#` represents an integer number), this indicates the file contains subfiles. It's worth noting that page 41 of the [TIFF 6.0 specification](https://web.archive.org/web/20180810205359/https://www.adobe.io/content/udp/en/open/standards/TIFF/_jcr_content/contentbody/download/file.res/TIFF6.pdf) mentions that the *SubfileType* tag is deprecated, and the *NewSubfileType* tag should be used instead. So why does ExifTool report *SubFileType*? A quick look at [ExifTool's documentation](https://exiftool.org/TagNames/EXIF.html) reveals that ExifTool's *SubfileType* output actually corresponds to the TIFF spec's *NewSubfileType* tag. If a TIFF contains the deprecated tag (*SubfileType* in the TIFF 6.0 spec), ExifTool reports this as *OldSubfileType* (yes, this is a bit confusing!).

I also ran Exiftool on [this 11-page TIFF](https://www.leadtools.com/support/forum/resource.ashx?a=545&b=1), which uses a different compression type for each page. The compression types can be inferred from the *Compression* fields in ExifTool's  output: 

```xml
 <IFD0:SubfileType>Single page of multi-page image</IFD0:SubfileType>
    ::
 <IFD0:Compression>Uncompressed</IFD0:Compression>
    ::
 <IFD1:SubfileType>Single page of multi-page image</IFD1:SubfileType>
    ::
 <IFD2:SubfileType>Single page of multi-page image</IFD2:SubfileType>
    ::
 <IFD2:Compression>JPEG 2000</IFD2:Compression>
    ::
 <IFD3:SubfileType>Single page of multi-page image</IFD3:SubfileType>
    ::
 <IFD3:Compression>JPEG</IFD3:Compression>
    ::
    ::
```

Also, as explained above, a single subfile can in turn contain multiple images, each of which are described by a *SubIFD* tag. I couldn't find any example files that use *SubIFDs*, but ExifTool reports them according to [the documentation](https://exiftool.org/TagNames/EXIF.html).

So, if you just want to cover all of the aforementioned cases, you might want to check your TIFFs for the presence of:

1. *IFD* with *SubfileType* tag (`<IFD#:SubfileType>`)
2. *IFD* with *OldSubfileType* tag (`<IFD#:OldSubfileType>`)
3. *SubIFD* tag.

### ImageMagick identify

You can also identify the presence of multiple images with ImageMagick's [*identify*](https://imagemagick.org/script/identify.php) tool (via [Tyler Thorsted](https://preservation.tylerthorsted.com/2023/09/15/tiff/)):

```bash
identify -quiet source.tif
```

Result:

```
source.tif[0] TIFF 9458x7429 9458x7429+0+0 8-bit sRGB 201.089MiB 0.000u 0:00.010
source.tif[1] TIFF 160x126 160x126+0+0 8-bit sRGB 0.000u 0:00.010
```

However, running this on the 11-page multi-format TIFF gave me the following error (with no meaningful information on the number of images):

```
identify-im6.q16: compression not supported `MultipleFormats.tif' @ error/tiff.c/ReadTIFFImage/1433.
```

It's also not clear to me how ImageMagick handles *SubIFD* tags.

### JHOVE

The output of [JHOVE](https://jhove.openpreservation.org/)'s TIFF module also provides information on subfiles. You can use the following command: 

```bash
jhove -m TIFF-hul -i source.tif -h xml -o source-jhove.xml
```

JHOVE (version 1.28.0) reports each *NewSubfileType* tag as:

```xml
<property>
<name>NewSubfileType</name>
<values arity="Scalar" type="Long">
  <value>0</value>
</values>
</property>
```

Unlike ExifTool, JHOVE also reports a *NewSubfileType* property for the main image, which means the output always contains at least one *NewSubfileType* property. So, TIFFs with subfiles can be singled out by the presence of more than one *NewSubfileType* property in JHOVE's output.

I noticed that JHOVE's *NewSubfileType* output doesn't always describe the role of each subfile. For example, for a TIFF with a subfile that is a thumbnail, JHOVE reports this (which is as expected):

```xml
<property>
<name>NewSubfileType</name>
<values arity="List" type="String">
  <value>reduced-resolution image of another image in this file</value>
</values>
</property>
```

But for [this 6-page TIFF](https://www.leadtools.com/support/forum/resource.ashx?a=544&b=1), the *NewSubfileType* tag of each TIFF is reported as:

```xml
<property>
<name>NewSubfileType</name>
<values arity="Scalar" type="Long">
  <value>0</value>
</values>
</property>
```

Unlike ExifTool, JHOVE's output doesn't provide any direct clue here that each subfile is a full-resolution image. Oddly, for [this 11-page multi-format TIFF](https://www.leadtools.com/support/forum/resource.ashx?a=545&b=1), JHOVE *does* report that each subfile is a single page:

```xml
<property>
<name>NewSubfileType</name>
<values arity="List" type="String">
  <value>single page of multi-page image</value>
</values>
</property>
```

Further perusal reveals that JHOVE's output on compression is also incomplete for this file: for 3 pages, it reports an "unknown" compression scheme:

```xml
<mix:Compression>
 <mix:compressionScheme>Unknown</mix:compressionScheme>
</mix:Compression>
```

This happens for pages with JBIG, JPEG and JPEG 2000 compression.

Again, if you're working with very old TIFFs, you might also want to check JHOVE's output for the presence of *SubfileType* properties, which represent the deprecated TIFF tag discussed in the ExifTool section.

JHOVE's [documentation](https://jhove.openpreservation.org/modules/tiff/) doesn't mention the *SubIFD* tag, so my best guess is this isn't supported.

## Conclusion

The presence of multiple images in TIFF files, either as thumbnails or pages in a multi-page document, can result in unexpected software behaviour. Because of this, it is important to identify them in any TIFF-based preservation workflows. Based on these tests with ExifTool, ImageMagick and JHOVE, it appears that ExifTool is the most reliable, robust and complete tool for identifying multi-image TIFFS. ExifTool was also the only tool that was able to correctly identify the compression scheme of each image in a particularly challenging 11-page TIFF with multiple compression schemes.

## Further resources

- [Blog post on TIFF by Tyler Thorsted](https://preservation.tylerthorsted.com/2023/09/15/tiff/) (discusses TIFF preservation risks, including multi-image files)
- [Multipage TIFF sample files from Leadtools](https://www.leadtools.com/support/forum/posts/t10960-)
- [TIFF 6.0 specification](https://web.archive.org/web/20180810205359/https://www.adobe.io/content/udp/en/open/standards/TIFF/_jcr_content/contentbody/download/file.res/TIFF6.pdf)
- [Adobe PageMaker 6.0 TIFF Technical Notes](https://www.awaresystems.be/imaging/tiff/specification/TIFFPM6.pdf) (defines *SubIFDs*)
- [Test script (bash)](https://github.com/KBNLresearch/jp2totiff/blob/master/pixelCheck-im.sh) that performs a pixel comparison on two directory trees with TIFF images.

[^1]: You might wonder why I didn't do the comparison directly between the source TIFF and the JP2, which would eliminate step 2. The reason for this is, that the ImageMagick builds don't include JPEG 2000 support by default. The only way to make this work is to build ImageMagick from its source. This requires that all the necessary development libraries are installed, and ImageMagick's build configuration is set up to include JPEG 2000 support. Even though I've successfully managed to make this work in the past, the procedure is quite time consuming, and I didn't want to go through it right now. So, for this test I used the round-trip conversion to TIFF as a workaround.

[^2]: An overview of all available `-metric` values can be found [here](https://imagemagick.org/script/command-line-options.php#metric).

[^3]: PSNR approaches infinity if both inputs are identical, which is exactly what you would expect for lossless compression.

[^4]: This is mainly because these were created according to very specific guidelines.