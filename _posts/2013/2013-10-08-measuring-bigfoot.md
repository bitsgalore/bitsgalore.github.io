---
layout: post
title: Measuring Bigfoot
tags: [preservation-risks]
comment_id: 48
---

My previous blog [*Assessing file format risks: searching for Bigfoot?*]({{ BASE_PATH }}/2013/09/30/assessing-file-format-risks-searching-bigfoot) resulted in some interesting feedback from a number of people. There was a particularly elaborate response from Ross Spencer, and I originally wanted to reply to that directly using the comment fields. However, my reply turned out to be a bit more lengthy than I meant to, so I decided to turn it into a separate blog entry.

<!-- more -->

## Numbers first?

Ross overall point is that [we need the numbers first](http://www.openplanetsfoundation.org/comment/511#comment-511); he makes a plea for collecting more format-related data, and adding numbers to these. Although these data do not directly translate into risks, Ross argues that it might be possible to use them to address format risks at a later stage. This may look like a sensible approach at first glance, but on closer inspection there's a pretty fundamental problem, which I'll try to explain below. To avoid any confusion here, I will be speaking of "format risk" here in the sense used by [Graf & Gordea](http://purl.pt/24107/1/iPres2013_PDF/A%20Risk%20Analysis%20of%20File%20Formats%20for%20Preservation%20Planning.pdf), which follows from the idea of "institutional obsolescence" (which is probably worth a blog post by itself, but I won't go into this here).

## The risk model

Graf & Gordea define institutional obsolescence in terms of "the additional effort required to render a file beyond the capability of a regular PC setup in particular institution". Let's call this effort *E*. Now the aim is to arrive at an index that has some predictive power of *E*. Let's call this index *R<sub>E</sub>*. For the sake of the argument it doesn't matter how *R<sub>E</sub>* is defined precisely, but it's reasonable to assume it will be proportional to *E* (i.e. as the effort to render a file increases, so does the risk):

*R<sub>E</sub>* &prop; *E*

The next step is to find a way to estimate *R<sub>E</sub>* (the dependent variable) as a function of a set of potential predictor variables:

*R<sub>E</sub>* = *f*(*S*, *P*, *C*, ... )

where *S* = software count, *P* = popularity, *C* = complexity, and so on. To establish the predictor function we have two possibilities:

1. use a statistical approach (e.g. multiple regression or something more sophisticated);
2. use a conceptual model that is based on prior knowledge of how the predictor variables affect *R<sub>E</sub>*.

The first case (statistical approach) is only feasible if we have actual data on *E*. For the second case we also need observations on *E*, if only to be able to say anything about the model's ability to predict *R<sub>E</sub>* (verification).

## No observed data on *E*!

Either way, the problem here is that there's an almost complete lack of any data on *E*. Although we may have a handful of isolated 'war stories', these don't even come close to the amount of data that would be needed to support any risk model, no matter whether it is purely statistical or based on an underlying conceptual model[^1]. So how are we going to model a quantity for which we do not have any observed data in the first place? Or am I overlooking something here?

Looking at Ross's suggestions for collecting more data, all of the examples he provides fall into the *potential* (!) predictor variables category. For instance, prompted by my observation on compression in *PDF*, Ross suggests to start analysing large collections of *PDF*s to establish patterns on the occurrence of various types of compression (and other features), and attach numbers to them. Ross acknowledges that such numbers by themselves don't tell you if *PDF* is "riskier" than another format, but he argues that:

> once we've got them  [the numbers], subject matter experts and maybe some of those mathematical types with far greater statistics capability than my own might be able to work with us to do something just a little bit clever with them.

Aside from the fact that it's debatable whether, in practical terms, the use of compression is really a risk (is there any evidence to back up this claim?), there's a more fundamental issue here. Bearing in mind that, ultimately, the thing we're *really* interested in here is *E*, how could collecting more data on potential predictor variables of *E* ever help here *in the near absence of any actual data* on *E*? No amount of clever maths or statistics  can compensate for that! Meanwhile, ongoing work on the prediction of *E* mainly seems to be focused on the collection, aggregation and analysis of potential predictor variables (which is also illustrated by Ross's suggestions), even though the purpose of these efforts remains largely unclear.

Within this context I was quite intrigued by the grant proposal mentioned by [Andrea Goethals](http://www.openplanetsfoundation.org/comment/513#comment-513) which, from the description, looks like an actual (and quite possibly the first) attempt at the systematic collection of data on *E* (although like [Andy Jackson said here](http://www.openplanetsfoundation.org/comment/513#comment-513) I'm also wondering whether this may be too ambitious). 

## Obsolescence-related risks versus format instance risks

On a final note, Ross makes the following remark about the role of tools:

> [W]ith tools such as Jpylyzer we have such powerful ways of measuring formats - and more and more should appear over time.

This is true to some extent, but a tool like *jpylyzer* only provides information on format *instances* (i.e. features of *individual files*); it doesn't say anything about preservation risks of the JP2 format *in general*. The same applies to tools that are are able to [detect features in individual PDF files]({{ BASE_PATH }}/2013/07/25/identification-pdf-preservation-risks-sequel) that are risky from a long-term preservation point of view. Such risks affect file instances of *current* formats, and this is an area that is covered by the [OPF File Format Risk Registry](http://wiki.opf-labs.org/display/TR/OPF+File+Format+Risk+Registry) that is being developed within SCAPE (it only covers a limited number of formats). They are largely unrelated to (institutional) format obsolescence, which is the domain that is being addressed by [*FFMA*](http://ffma.ait.ac.at:8080/preservation-riskmanagement/). This distinction is important, because both types of risks need to be tackled in fundamentally different ways, using different tools, methods and data. Also, by not being clear about which risks are being addressed, we may end up not using our data in the best possible way. For example, Ross's suggestion on compression in *PDF* entails (if I'm understanding him correctly) the analysis of large volumes of *PDF*s in order to gather statistics on the use of different  compression types. Since such statistics say little about individual file instances, a more practically useful approach might be to profile individual files instances for 'risky' features.

[^1]: On a side note even conceptual models often need to be fine-tuned against observed data, which can make them pretty similar to statistically-derived models.
<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2013/10/08/measuring-bigfoot/)
