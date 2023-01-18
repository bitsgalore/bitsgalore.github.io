---
layout: post
title: Image Portable Media Like a Boss!
tags: [disk-imaging, floppy-disks]
comment_id: 83
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2022/11/fail_whale_by_ka_92-d5ra7vf.jpg" alt="Painting of a whale, which is lifted above the beach by a swarm of red Twitter birds. In the foreground, an elderly couple watches the scene.">
  <figcaption><a href="https://web.archive.org/web/20130204212401/https://ka-92.deviantart.com/art/fail-whale-348157275">"Fail whale"</a> by <a href="https://web.archive.org/web/20130128112538/http://ka-92.deviantart.com/">Kuni (ka-92)</a> (license unknown), based on "Lifting a Dreamer" by <a href="http://www.yiyinglu.com/?portfolio=lifting-a-dreamer-aka-twitter-fail-whale">Yiying Lu</a>.</figcaption>
</figure>

In 2017 I wrote [a blog post]({{ BASE_PATH }}/2017/06/19/image-and-rip-optical-media-like-a-boss) on [Iromlab](https://github.com/KBNLresearch/iromlab) (an acronym for "Image and Rip Optical Media Like A Boss"), a custom-built software tool that streamlines imaging and ripping of optical media, using an Acronova Nimbie disc robot. The KB has been using Iromlab since 2019 as part of an [ongoing effort](https://www.kb.nl/over-ons/projecten/digitalisering-optische-dragers) to preserve the information contained in its vast collection of legacy optical media. This project is expected to reach its completion later this year, but as demonstrated [by this earlier inventory]({{ BASE_PATH }}/2020/02/20/offline-digital-carriers-kb-deposit-collection), our deposit collection also contains various other types of legacy media that are under threat of becoming inaccessible. As shown by the inventory, 3.5 inch floppy disks are the most common data carriers (after optical media), so it made sense to focus on these as a next step. 

Using the existing Iromlab-based workflow as a starting point, I created a preliminary workflow tool that could be used for imaging our 3.5" floppies (and various other types portable media). In this post I'll explain how this tool came about, and highlight some of the challenges I encountered during its development.

<!-- more -->

## The plan

Since we already have a workflow in place for optical media, it seemed to make sense to use Iromlab as a starting point. This would allow us to re-use most of the software components of the existing optical media workflow with only minor modifications. It also implied that Windows (10) would be the target platform. So far for the theory, but as we'll see below, things got slightly more complicated and messy along the way!

## Iromlab to Ipmlab

As a first step I forked the existing Iromlab code. Unlike optical media, the possibilities for automating the load and unload process are limited for floppies (although some hackers have successfully [repurposed vintage floppy duplicators into DIY autoloaders](https://hackaday.com/2012/03/31/floppy-autoloader-takes-the-pain-out-of-archiving-5000-amiga-disks/)). So, I started by removing all dependencies on the Nimbie disc robot. Next I removed everything that is specific to optical media, and made some small changes to the IsoBuster call, in order to better accommodate 3.5 inch floppies. As the software also needed a name, I settled on "Ipmlab" (Image Portable Media Like A Boss).

## Enter Aaru

Although the first tests with this early prototype went well, I wasn't entirely happy with IsoBuster's behaviour in case of floppies with damaged sectors. Depending on the configuration settings, IsoBuster either require manual intervention  in this case, or alternatively abort the imaging process altogether (filling in the missing sectors with placeholder bytes). On Twitter, Robin François suggested the open-source, cross-platform [Aaru](https://www.aaru.app/) software as a possible alternative imaging solution. This looked worthy of further exploration. Although I hadn't used Aaru before, the results of some quick first tests looked promising enough, so I added a simple Aaru wrapper module to Ipmlab.

## Write blocker woes

This all seemed to work fine at first, but once I connected any of my (external USB) floppy drives through a write blocker (Tableau T8u forensic USB Bridge), Aaru would sometimes throw device exceptions. 

Now it's relevant here to mention that for testing I mostly used some old, DOS-formatted 3.5 inch floppies from my personal collection. These include both ["high density"](https://obsoletemedia.org/3-5-inch-microfloppy-high-density/) disks with a capacity of 1.44 MB, as well as some older ["double density"](https://obsoletemedia.org/3-5-inch-microfloppy/) disks, which can only hold 720 KB worth of data. The main pattern in the Aaru crashes was, that they typically happened whenever I tried to process a "high density" disk after having processed one of more "double density" ones, or vice versa. The crashes didn't happen when the floppy drives were connected directly to my machine.

## Ddrescue tests

To get a better idea what was going on, I ran more tests using an alternative imaging application ([ddrescue](https://www.gnu.org/software/ddrescue/)), and repeated these tests on another machine with a different operating system (Linux Mint).

I started by imaging one "double density" (720 KB) disk. Any subsequent floppies were typically imaged without problems, provided that these were also "double density" disks. These each resulted in a 737 KB disk image, that I could mount normally on my Linux machine. On the other hand, following up a "double density" disk with a "high density" disk resulted in the following output from ddrescue:

```bash
Press Ctrl-C to interrupt
     ipos:   720896 B
     opos:   720896 B
non-tried:        0 B
  rescued:   737280 B
pct rescued:  100.00%, read errors:        0
```

This may look normal at first, as it does not report any errors. On closer inspection, we see that only 737 KB of data were extracted from the disk. This is unexpected, because a "high density" disk should really result in a 1.5 MB disk image. But the 737 KB value *does* correspond exactly to the expected size for a "double density" disk! So in spite of the lack of any error messages from ddrescue, about half of the data on the disks are missing from the image files! 

As an additional test I switched off the write blocker, switched it on again, and then repeated the above experiment, but now starting with some "high density" disks. These were all imaged correctly, each resulting in 1.5 MB image files. Following this up with any of my "double density" disks invariably resulted in ddrescue reporting read errors like this:

```
pct rescued:   48.88%, read errors:     7372
```

So apparently ddrescue expected 1.44 MB of data (the size of the "high density" disk I inserted first), instead of the 720 KB of a "double density" disk.

Running ddrescue on an *empty* floppy drive resulted in an image with only null bytes (all bad blocks), with an image size of either 737 KB or 1.5 MB, depending on whether a "double" or "high" density floppy had been inserted first.

Repeating any of the above tests with Aaru mostly resulted in Aaru device exceptions. At first sight, this all suggests that the medium size that is exposed by the write blocker to the operating system somehow remains "stuck" to the size of the first floppy that was loaded after switching it on, and isn't updated afterwards.

## Tableau response

I contacted Tableau (the manufacturer of the write blocker) about this, who pointed at an inability of the write blocker to differentiate properly between the actual floppy and the USB adapter device as the most likely cause. They don't envisage a short-term solution for this because of the complexities involved, and advised me to look for some alternative solution that doesn't use this write blocker.

Why use write blocker at all? "System Volume Information" folders! Show screenshot!

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/01/svi-win10.png" alt="Screenshot of file manager showing the contents of a floppy disk. In addition to the original files, it also shows a System Volunme Information directory that was added by Windows 10 after inserting the floppy into the machine." >
  <figcaption>Contents of a floppy. The System Volume Information directory was not originally part of the floppy, but was automatically added by Windows 10 after insertion into the floppy drive.</figcaption>
</figure>


[floppy autoloader](https://hackaday.com/2012/03/31/floppy-autoloader-takes-the-pain-out-of-archiving-5000-amiga-disks/)

## Additional links and resources

- [Ipmlab](https://github.com/KBNLresearch/ipmlab)
- [Offline digital data carriers in the KB deposit collection]({{ BASE_PATH }}/2020/02/20/offline-digital-carriers-kb-deposit-collection)
- [A simple disk imaging workflow tool]({{ BASE_PATH }}/2019/04/10/a-simple-disk-imaging-workflow-tool)
- [Safeguarding optical information carriers (in Dutch)](https://www.kb.nl/over-ons/projecten/digitalisering-optische-dragers)
- [Image and Rip Optical Media Like A Boss!]({{ BASE_PATH }}/2017/06/19/image-and-rip-optical-media-like-a-boss)
- [Iromlab](https://github.com/KBNLresearch/iromlab)
