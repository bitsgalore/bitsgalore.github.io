---
layout: post
title: Why can't we have digital preservation tools that just work?
tags: [DROID,JHOVE,JHOVE2,FITS,Fido,rant]
comment_id: 46
---

One of my first blogs here covered an [evaluation of a number of format identification tools]({{ BASE_PATH }}/2011/09/21/evaluation-identification-tools-first-results-scape). One of the more surprising results of that work was that out of the five tools that were tested, no less than four of them (*FITS*, *DROID*, *Fido* and *JHOVE2*) failed to even *run* when executed with their associated launcher script. In many cases the *Windows* launcher scripts (batch files) only worked when executed from the installation folder. Apart from making things unnecessarily difficult for the user, this also completely flies in the face of all existing conventions on command-line interface design. Around the time of this work (summer 2011) I had been in contact with the developers of all the evaluated tools, and until last week I thought those issues were a thing of the past. Well, was I wrong!

<!-- more -->

## [FITS 0.8](http://projects.iq.harvard.edu/files/fits/files/fits-0.8.0.zip)

Fast-forward 2.5 years: this week I saw the announcement of the latest [FITS](http://projects.iq.harvard.edu/fits) release. This got me curious, also because of the recent work on this tool as part of the [FITS Blitz](http://www.openplanetsfoundation.org/blogs/2013-11-06-fits-blitz). So I downloaded [FITS 0.8](http://projects.iq.harvard.edu/files/fits/files/fits-0.8.0.zip), installed it in a directory called *c:\fits\\*on my *Windows* PC, and then typed (while being in directory *f:\myData\\*):

```bash
f:\myData>c:\fits\fits
```
Instead of the expected helper message I ended up with this:

```data
The system cannot find the path specified.
Error: Could not find or load main class edu.harvard.hul.ois.fits.Fits
```

Hang on, I've seen this before ... don't tell me this is the same bug that I already reported 2.5 years ago ? Well, turns out [it is](https://github.com/harvard-lts/fits/issues/10) after all!

This got me curious about the status of the other tools that had similar problems in 2011, so I started downloading the latest versions of [*DROID*](http://www.nationalarchives.gov.uk/information-management/our-services/dc-file-profiling-tool.htm), [*JHOVE2*](https://bitbucket.org/jhove2/main/wiki/Home) and [*Fido*](https://github.com/openplanets/fido). As I was on a roll anyway, I gave [JHOVE](http://jhove.sourceforge.net/) a try as well (even though it was not part of the 2011 evaluation). The objective of the test was simply to *run* each tool and get some screen output (e.g. a help message), nothing more. I did these tests on a PC running *Windows* 7 with *Java* version 1.7.0_25. Here are the results.  

## [DROID 6.1.3](http://www.nationalarchives.gov.uk/documents/information-management/droid-binary-6.1.3-bin.zip)

First I installed *DROID* in a directory *C:\droid\\*. Then I executed it using:

```bash
f:\myData>c:\droid\droid
```

This started up a *Java Virtual Machine Launcher* that showed this message box:

![]({{ BASE_PATH }}/images/2014/01/droidError.png)

The *Running DROID* text document that comes with *DROID* says:

> To run DROID on Windows, use the "droid.bat" file.  You can either double-click on this file, or run it from the command-line console, by typing "droid" **when you are in the droid installation folder**.

So, no progress on this for *DROID* either, then. I *was* able to get *DROID* running by circumventing the launcher script like this:

```bash
java -jar c:\droid\droid-command-line-6.1.3.jar
```

This resulted in the following output:

```data
No command line options specified
```

This isn't particularly helpful. There *is* a helper message, for which you have to give the *-h* flag on the command line. But you don't get to see this until you give the *-h* flag on the command line. Catch 22 anyone?

## [JHOVE2-2.1.0](http://bitbucket.org/jhove2/main/downloads/jhove2-2.1.0.zip)

After installing *JHOVE2* in *c:\jhove2\\*, I typed:

```bash
f:\myData>c:\jhove2\jhove2
```

This gave me **1393** (yes, you read that right: 1393!) *Java* deprecation warnings, each along the lines of:

```data
16:51:02,702 [main] WARN  TypeConverterDelegate : PropertyEditor [com.sun.beans.editors.EnumEditor]
found through deprecated global PropertyEditorManager fallback - consider using a more isolated
form of registration, e.g. on the BeanWrapper/BeanFactory!
```

This was eventually followed by the (expected) *JHOVE2* help message, and a quick test on some actual files confirmed that *JHOVE2* *does* actually work. Nevertheless, by the time the tsunami of warning messages is over, many first-time users will have started running for the bunkers!

## [Fido 1.3.1](https://github.com/openplanets/fido/releases/tag/1.3.1-70)

*Fido* doesn't make use of any launcher scripts any more, and the default way to run it is to use the *Python* script directly. After installing in *c:\fido\\* I typed:

```bash
f:\myData>c:\fido\fido.py
```

Which resulted in ..... (drum roll) ... a nicely formatted *Fido* help message, which is exactly what I was hoping for. Beautiful!

## [JHOVE 1.11](http://sourceforge.net/projects/jhove/files/latest/download)

I installed *JHOVE* in *c:\jhove\\* and then typed:

```bash
f:\myData>c:\jhove\jhove 
```

Which resulted in this:

```data
Exception in thread "main" java.lang.NoClassDefFoundError: edu/harvard/hul/ois/j
hove/viewer/ConfigWindow
        at edu.harvard.hul.ois.jhove.DefaultConfigurationBuilder.writeDefaultCon
figFile(Unknown Source)
        at edu.harvard.hul.ois.jhove.JhoveBase.init(Unknown Source)
        at Jhove.main(Unknown Source)
Caused by: java.lang.ClassNotFoundException: edu.harvard.hul.ois.jhove.viewer.Co
nfigWindow
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        ... 3 more
```

Ouch!

## Final remarks

I limited my tests to a *Windows* environment only, and results may well be better under *Linux* for some of these tools. Nevertheless, I find it nothing less than astounding that so many of these (often widely cited) preservation tools fail to even *execute* on today's [most widespread operating system](http://en.wikipedia.org/wiki/Usage_share_of_operating_systems). Granted, in some cases there are workarounds, such as tweaking the launcher scripts, or circumventing them altogether. However, this is not an option for less tech-savvy users, who will simply conclude "*Hey, this tool doesn't work*", give up, and move on to other things. Moreover, this means that much of the (often huge) amounts of development effort that went into these tools will simply fail to reach its potential audience, and I think this is a tremendous waste. I'm also wondering why there's been so little progress on this over the past 2.5 years. Is it really that difficult to develop preservation tools with command-line interfaces that follow basic design conventions that have been ubiquitous elsewhere for more than 30 years? Tools that *just work*?

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2014/01/31/why-cant-we-have-digital-preservation-tools-just-work/)
