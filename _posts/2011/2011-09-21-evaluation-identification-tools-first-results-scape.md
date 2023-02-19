---
layout: post
title: Evaluation of identification tools&#58; first results from SCAPE
tags: [format-identification,unix-file,DROID,Fido,FITS,JHOVE2]
comment_id: 64
---

As I already briefly mentioned in a [previous blog
post]({{ BASE_PATH }}/2011/07/11/improved-identification-xml-python-experiment),
one of the objectives of the
[SCAPE](http://www.scape-project.eu) project is to develop an
architecture that will enable large scale characterisation of digital
file objects. As a first step, we are evaluating existing
characterisation tools. The overall aim of this work is twofold. First,
we want to establish which tools are suitable candidates for inclusion
in the SCAPE architecture. As the enhancement of existing tools is
another goal of SCAPE, the evaluation is also aimed at getting a better
idea of the specific strengths and weaknesses of each individual tool.
The outcome of this will be helpful for deciding what modifications and
improvements are needed. Also, many of these tools are widely used
outside of the SCAPE project, which means that the results will most
likely be relevant to a wider audience (including the original tool
developers).

<!-- more -->

## Evaluation of identification tools

Over the last months, work on this has focused on format identification
tools. This has resulted in a
[report](https://zenodo.org/record/840345)
which is attached with this blog post. We have evaluated the following
tools:

- [DROID](http://sourceforge.net/apps/mediawiki/droid/index.php?title=Main_Page)
  6.0
- [FIDO](https://github.com/openplanets/fido) 0.9
- [Unix File Utility](http://www.darwinsys.com/file/)
- [FITS](http://code.google.com/p/fits/) 0.5
- [JHOVE2](https://bitbucket.org/jhove2/main/wiki/Home)

All tools were evaluated against a set of 22 criteria. Extensive testing
using real data has been a key part of the work. One area which, I
think, we haven’t been able to tackle sufficiently so far is the
accuracy of the tools. This is problematic, since it would require a
test corpus where the format of each file object is known *a priori*. In
most large data sets this information will be derived from the very same
tools that we are trying to test, so we need to see if we can say
anything meaningful about this in a follow-up.

## Involvement of tool developers

Over the previous months we’ve been sending out earlier drafts of this
document to the developers of DROID, FIDO, FITS and JHOVE2, and we have
received a lot of feedback to this. In the case of FIDO, a new version
is underway, and this should correct most (if not all) of the problems
that are mentioned in the report. For the other tools we have also
received confirmation that some of the found issues will be fixed in
upcoming releases.

## Status of the report and future work {#status-of-the-report-and-future-work}

The attached report should be seen as a living document. There will
probably be one or more updates at some later point, and we may decide
to include more tests using additional data. Meanwhile, as always, we
appreciate any of your feedback on this!

## Link to report

[Evaluation of characterisation tools – Part 1:
Identification](https://zenodo.org/record/840345)

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2011/09/21/evaluation-identification-tools-first-results-scape/)
