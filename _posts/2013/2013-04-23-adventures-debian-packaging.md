---
layout: post
title: Adventures in Debian packaging
tags: [packaging,Debian,jpylyzer]
comment_id: 54
---

About a year ago, work started on [packaging SCAPE tools](http://www.openplanetsfoundation.org/blogs/2012-02-20-summary-outputs-and-roadmap-feb-2012). *Jpylyzer* was the first SCAPE tool that was [turned into a Debian package](http://www.openplanetsfoundation.org/blogs/2012-02-15-sustainability-and-adoption-preservation-tools). Some time later, the OPF set up a couple of machine images at Amazon Web Services, which can be used to [create packages repeatedly using a virtual machine](http://www.openplanetsfoundation.org/blogs/2012-03-08-turning-github-code-debian-packages-opf-way). Even though I've used the Amazon service a couple of times myself, I really know next to nothing about Debian packages, and it's safe to say that the underlying build process has been more or less a complete mystery to me.

To get a better understanding of the process for building Debian packages, I had a try at packaging *jpylyzer* on my local machine (which runs on [Linux Mint 14](http://blog.linuxmint.com/?p=2216)). Some time ago Dave Tarrant and Rui Castro wrote [a nice step-by-step guide on building Debian packages](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) on the OPF Wiki, so I tried to follow the instructions there. While working on this, I made some notes, mainly to remind myself of what I was doing. Then I realised that some of this might be useful to others as well, so I decided to turn it into a blog post.

<!-- more -->

## Objectives

The objectives of this exercise were:

+ to get more more familiar with the packaging process myself;
+ to provide some input on how useful the [guide](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) on the OPF Wiki is from the perspective of someone who is largely ignorant of the packaging procedure;
+ to identify any problems in *jpylyzer*'s packaging procedure.

I did two experiments: first, I did a *very* limited test where I tried to create a template directory structure using *debhelper*, which would be the first step when starting from scratch. Since for *jpylyzer* all the files in the *debian* directory already exist, I then moved on to building *jpylyzer* using the existing files.

## Test 1: creating the directory structure from scratch

For this, I first installed all the required packages listed in the *Pre-Requisites* section of the [guide](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) using:

```bash
sudo apt-get install build-essential dh-make devscripts debhelper lintian
```

Subsequently I followed the instructions in the *Getting Started* section. For this I simply created an empty directory:

```bash
mkdir debtest_1.0.0
```

And then:

```bash
cd debtest_1.0.0
```

Then I ran *dh\_make*:

```bash
dh_make
```

This resulted in an error message, telling me that the package name and its version number should be separated by a dash ('-') instead of an underscore ('_'), or, alternatively, that the -p flag should be used. So I changed the directory name:

```bash
mv debtest_1.0.0 debtest-1.0.0
```

Re-running *dh\_make*, it now accepted the directory name, but it complained about a missing tarball (which I purposefully didn't make in this test). However, as *dh\_make* offered the suggestion to use the `--createorig` option (which creates a tarball) I tried this:

```bash
dh_make --createorig
```

This resulted in the creation of a *debian* directory with file templates, and an (empty) tarball *debtest\_1.0.0.orig.tar.gz* which was created in the parent (*debtest*) directory.

So, apart from the dash/underscore mix-up this is all pretty straightforward.  
  
## Test 2: building *jpylyzer*

In this second test I tried to build *jpylyzer* using the [already existing files in the *debian* folder of  *jpylyzer*'s Git repository](https://github.com/openplanets/jpylyzer/tree/master/debian). First I cloned the repository to my local machine:

```bash
git clone https://github.com/openplanets/jpylyzer.git
```

Then I went into the *jpylyzer* directory:

```bash
cd jpylyzer
```

From there I tried to build *jpylyzer* directly, using the command given in the [guide's](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) *Building your package* section:

```bash
dpkg-buildpackage -tc
```

## Missing changelog

The above command resulted in an error message about a missing *changelog* file in the *debian* folder. The *changelog* section in the [OPF guide](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) does mention an OPF-hosted GitHub 2 Changelog service, which is supposed to be callable from the rules file. But I don't see any reference to it in jpylyzer's [*rules*](https://github.com/openplanets/jpylyzer/blob/master/debian/rules) file, so I don't really know how this is supposed to work! To to keep going I simply grabbed the default changelog that was created by *debhelper* in an earlier experiment. After this I ran the command again.

## Unknown commands in makefile

This time, *dpkg-buildpackage* exited with the following errors:

```data
pymakespec --onefile jpylyzer.py
make[1]: pymakespec: Command not found
make[1]: *** [build] Error 127
make[1]: Leaving directory `/home/johan/debtest/jpylyzer'
make: *** [build] Error 2
dpkg-buildpackage: error: debian/rules build gave error exit status
```

These errors arise from the following lines in *jpylyzer*'s [makefile](https://github.com/openplanets/jpylyzer/blob/master/Makefile):

```data
build:
    pymakespec --onefile jpylyzer.py
    pyinstaller jpylyzer.spec
    @echo "Built in dist/jpylyzer"
```

The *pymakespec* and *pyinstaller* commands above are most likely shell scripts that launch the *Makespec.py* and *pyinstaller.py* scripts that are both part of [*PyInstaller*](http://www.pyinstaller.org/) (these are used for building an executable from the source code). However, neither the shell scripts nor any references to them are included in *jpylyzer*'s repository (my best guess is that they exist only on a specific machine instance - perhaps the Amazon virtual machines?), so the makefile simply won't work.

I was able to fix this by changing the references to the shell scripts to this (using *PyInstaller 1.5*):

```bash
python /home/johan/pyinstall1.5/Makespec.py --onefile jpylyzer.py
python home/johan/pyinstall1.5/pyinstaller.py jpylyzer.spec
```

For *PyInstaller 2* these two lines should be substituted by:

```bash
python /home/johan/pyinstall/pyinstaller.py --onefile jpylyzer.py	
```

Note here that *PyInstaller* has no default installation location, and the file paths will vary from machine to machine!

After making these changes I was able to run *dpkg-buildpackage* without any problems:

```bash
dpkg-buildpackage -tc
```

Result: the following files were created in the repo's parent directory:

+ jpylyzer\_1.9.0\_amd64.changes
+ jpylyzer\_1.9.0\_amd64.deb
+ jpylyzer\_1.9.0.dsc
+ jpylyzer\_1.9.0.tar.gz

## Tarball schmarball

One thing that confused me at first: the *Getting Started* section in the [OPF guide](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) mentions the need for building a *native* package before starting the Debian packaging:

> If you have got here and you don't have any already packaged code (a tar ball with makefile etc) then you will need to build a native package.

So, I initially thought I would need to create a tarball of my repo first. As it turns out this is not the case: the tarball is created automatically once you run *dpkg-buildpackage*. So this is one thing less to worry about!

## Verifying the package with *lintian*

As a final step I used *lintian* to verify my package:

```bash
lintian jpylyzer_1.9.0_amd64.deb
```

This resulted in the following output (using *PyInstaller* 1.5):

```data
E: jpylyzer: unstripped-binary-or-object usr/bin/jpylyzer
W: jpylyzer: hardening-no-fortify-functions usr/bin/jpylyzer
W: jpylyzer: wrong-bug-number-in-closes l3:#nnnn
E: jpylyzer: debian-changelog-file-contains-invalid-email-address johan@unknown
E: jpylyzer: helper-templates-in-copyright
```

With *PyInstaller 2* I got this additional warning:  

```data
W: jpylyzer: hardening-no-relro usr/bin/jpylyzer
```

I still need to give these errors and warnings an in-depth look. At least one error is related to the bogus *changelog* file I used. Some others (e.g. *unstripped-binary-or-object*) appear to be related to the build process of the binaries.

## Conclusions

Using the [Building Your Debian Package](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) guide on the OPF Wiki I was able to create a rudimentary skeleton structure for Debian packaging. I was also able to build a Debian package for *jpylyzer*. The exercise revealed some problems with the Debian setup for *jpylyzer*. The most important ones are:

+ It's unclear how *jpylyzer*'s *changelog* file is supposed to be generated. Perhaps there's a dependency on some external service (the OPF Github 2 Changelog service?), but I cannot find any documentation on how to make this work!
+ The makefile calls *PyInstaller* in a non-standard an undocumented way. This is easy to fix locally if you are familiar with *PyInstaller*, but not so otherwise. Also, the interfaces of versions 1.5 and 2 of *PyInstaller* are different, and depending of what version you are running this may require additional changes to the makefile.
+ Even though I was able to build a Debian package for *jpylyzer*, it still ended up with some *lintian* errors.

I also came across a few minor errors in the OPF guide. I left a short comment on this [here](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package) (scroll to bottom). Overall, I found the guide really helpful, and it provides an accessible and relatively painless introduction to the packaging process.
  
## Reference

[Building Your Debian Package (OPF Wiki)](http://wiki.opf-labs.org/display/SP/Building+Your+Debian+Package)

## Post scriptum

Proof again that it's always a bad idea to come up with a clever title for a blog post without [Googling](https://www.google.com/#hl=en&gs_rn=11&gs_ri=psy-ab&cp=2&gs_id=zz&xhr=t&q=%22adventures+in+Debian+Packaging%22&es_nrs=true&pf=p&sclient=psy-ab&oq=%22adventures+in+Debian+Packaging%22&gs_l=&pbx=1&bav=on.2,or.r_qf.&bvm=bv.45580626,d.d2k&fp=6b211d71a5d752ed&biw=1140&bih=553) it first: after writing this post I found out that the [Mid Hudson Valley Linux and Open Source Users Group](http://mhvlug.org/) will be organising a meeting called [*Adventures in Debian Packaging*](http://mhvlug.org/meetings/2013/adventures-in-debian-packaging) later this year in Poughkeepsie, NY. Completely unrelated to this blog, of course, but it's only fair to give it a mention. Well, there you go.

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/04/23/adventures-debian-packaging/)
