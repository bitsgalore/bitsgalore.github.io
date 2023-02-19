---
layout: post
title: Ensuring the suitability of JPEG 2000 for preservation
tags: [jpeg-2000,JP2]
comment_id: 68
---

In my [presentation](http://www.dpconline.org/component/docman/doc_download/526-jp2knov2010vanderkniff)
 during the [Wellcome Trust's JPEG 2000 seminar](http://blog.wellcomelibrary.org/2010/11/wellcome-trust-hosts-jpeg-2000-seminar/) I discussed the suitability of JPEG 2000
(and more specifically its JP2 format) for long-term preservation. I
highlighted the erroneous restriction in the JP2 (and JPX) format
specification that only allows ICC profiles of the 'input' class to be
used. This effectively prohibits the use of all working colour spaces
such as Adobe RGB, which are defined using 'display device' profiles. I
also showed how different software vendors interpret the format
specification in subtly different ways, and how such issues can create
problems in the long term, such as the loss of colour space and
resolution information after some future migration.

<!-- more -->

This leads us to the question to which extent we can predict a specific
file format's suitability for long-term preservation. The answer is not
that straightforward. The Library of Congress assesses file formats
against 7 ['sustainability factors'](http://www.digitalpreservation.gov/formats/sustain/sustain.shtml),
whereas the National Archives have formulated [a list of 12 criteria](http://www.nationalarchives.gov.uk/documents/selecting-file-formats.pdf).
It is beyond the scope of this blog post to present a detailed analysis
of the extent to which JP2 lives up to either set of criteria. However,
it is interesting to have a look at whether these criteria could have
been helpful in identifying the issues covered by my presentation.

## Format specifications

First, both the LoC's 'sustainability factors' and the TNA criteria
acknowledge the importance of having published specifications of a file
format. The LoC uses a 'Disclosure' factor, which refers to “the
existence of complete documentation, preferably subject to external
expert evaluation”. TNA take this one step further by also defining a
'Documentation Quality' criterion, which expresses the degree to which
documentation is comprehensive, accurate and comprehensible. This last
criterion largely covers the JPEG 2000 ICC issue, although it's
questionable how useful this would have been to identify it *a priori*.
A problem with errors and ambiguities in format specifications is that
they can be incredibly easy to overlook, and you may only become aware
of them after discovering that different software products interpret the
specifications in slightly different ways.

## Adoption

Formats that are widely used are typically well supported by an array of
software tools, and such formats are unlikely to disappear into
obsolescence. TNA expresses this through a 'Ubiquity' criterion, which
essentially reflects a file format's overall popularity. The definition
of the LoC's 'Adoption' factor includes a list of criteria that can be
used as “evidence of adoption”. The first set of criteria here includes
“bundling of tools with personal computers, native support in Web
browsers or market-leading content creation tools, and the existence of
many competing products for creation, manipulation, or rendering of
digital objects in the format”. Note that JP2 isn't doing particularly
well when measured against any of these criteria. However, the LoC list
adds that “a format that has been reviewed by other archival
institutions and accepted as a preferred or supported archival format
also provides evidence of adoption”. This certainly seems to be the case
for JP2. But how relevant is this, really? Going back to the ICC
profiles issue: the JP2 file format has been around for about 10 years
now, and its acceptance by the archival community has been growing
steadily over the last 5 years or so. Yet, this whole issue seems to
have gone unnoticed in the archival community for all those years, and I
think this is slightly worrying.

Now let's imagine for a moment that JP2 would have been picked up by the
digital photography and graphic design communities. For such uses the
ability to do proper colour management is a basic prerequisite, and
limiting the support of ICC profiles to the 'input' class would have
made the format virtually useless to these user communities. My guess is
that in this -entirely fictional- scenario, the format specification
would have either improved quickly (based on feedback from the user
community), or the respective user communities would have simply stopped
using the format altogether. The problem here seems to be that very few
people in the archiving community are even aware of such things as
colour spaces and colour management, let alone their importance within
the context of preservation. With more established formats such as TIFF
this may not be as much of a problem, if only because TIFF has been
'road tested' for decades by the photography and graphic design
communities. As an archiving community we cannot fall back to any
similar 'road testing' in the case of JP2. And this brings me to my next
point.

## Importance of hands-on experience

Preservation criteria such as those of the LoC or TNA are invaluable for
assessing the suitability of a format for preservation, but I believe it
is equally important to have actual hands-on experience with the tools
that are used for creating, modifying, and reading the format. For
instance, the TNA criteria use the *number* of software tools that
support a given format as an indicator for the extent of current
software support of that format. But knowing the *number* of tools says
nothing about how good or useful these tools actually are! In the case
of JP2, quite a large number of (mostly free or open-source) tools exist
that, under the hood, are using the open [JasPer](http://www.ece.uvic.ca/~mdadams/jasper/)
library. JasPer is known to have performance and stability issues that
make it unsuitable for most professional applications (for which, I
should emphasise, it was never developed in the first place!). These
issues affect *all* software tools that are using JasPer. So, only
counting the number of available tools may be simply missing the point
without incorporating any additional quality criteria. But how would you
define these?

Part of the answer, I think, is that assessing a format's suitability
for long-term preservation is not a purely top-down process. Most of the
software-related issues that I showed in my presentation were found by
simply experimenting with actual files, encoders and characterisation
tools: convert a TIFF to JP2; convert it back to TIFF; use existing
metadata-extraction and characterisation tools such as [ExifTool](http://www.sno.phy.queensu.ca/~phil/exiftool/)
and [JHOVE](http://hul.harvard.edu/jhove/) to analyse the
in- and output files; try to understand the output of these tools;
compare the output before and after the conversion, and so on. Such
experiments are extremely useful for getting a feel for the strengths
and weaknesses of specific software tools, and they can reveal problems
that are not readily captured by pre-defined criteria. In some cases,
their results may be used to refine existing criteria, or even add new
ones.

## Final notes on preservation criteria

Although I wouldn’t downplay the importance of preservation criteria
such as those used by the LoC or TNA, I think it’s important to realise
that such criteria are largely based on theoretical considerations. In
most cases they are not based on any empirical data, and as a result
their predictive value is largely unknown. For example, [an interesting
blog post by David Rosenthal](http://blog.dshr.org/2009/01/are-format-specifications-important-for.html)
argues that preserving the specifications of a file
format doesn’t contribute anything to practical digital preservation.
According to Rosenthal, the availability of working open-source
rendering software is much more important, and he explains how “formats
with open source renderers are, for all practical purposes, immune from
format obsolescence”.

This takes us directly to the lack of JPEG 2000-related activity in the
open source community, which I also referred to in my presentation.
Perhaps the best way to ensure sustainability of JPEG 2000 and the JP2
format would be to invest in a truly open JP2 software library, and
release this under a free software license. This could either take the
form of the development of a completely new library, or investing in the
improvement and further development of an existing one, such as [OpenJPEG](http://www.openjpeg.org/).
This would require an investment from the archival community, but the
payoff may be well worth it.

## Acknowledgements

This blog entry was largely inspired by an e-mail discussion that was
started by Richard Clark, and in particular by a contribution to this
discussion by William Kilbride.

<hr>
Originally published at the [Wellcome Library Blog](http://blog.wellcomelibrary.org/2010/12/guest-post-ensuring-the-suitability-of-jpeg-2000-for-preservation/)