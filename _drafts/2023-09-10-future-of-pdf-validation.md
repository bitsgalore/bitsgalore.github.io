---
layout: post
title: The future of PDF validation
tags: [PDF, format-validation, preservation-risks, VeraPDF, JHOVE]
comment_id: 90
---

Before the summer holidays I wrote two blog posts on PDF processing tools. The [first one of these]({{ BASE_PATH }}/2023/05/25/identification-of-pdf-preservation-risks-with-verapdf-and-jhove) looked into feature extraction (and more specifically the identification of preservation risks) with JHOVE and VeraPDF. The [second one]({{ BASE_PATH }}/2023/06/29/verapdf-parse-status-as-a-proxy-for-rendering) offered a first glimpse into associations between VeraPDF parse errors and rendering behaviour. The goal here was to see if such parse errors could be used as a rough "validity proxy" to identify malformed PDFs.

This work raised (at least for me!) some questions about the place of tools like JHOVE and VeraPDF in modern digital preservation workflows. I briefly addressed some of these in my earlier posts, but I think this deserves a dedicated post.




<!-- more -->

## Purpose of tools

Why are qwe using dp tools, what purposes do they serve, what's the point?

Good 2016 discussion of rationales behind format identification, validation and property extraction by Ross Spencer:

[What is the point? The motivation for adopting different tools inside the digital preservation workflow](https://openpreservation.org/blogs/what-is-the-point-the-motivation-for-adopting-different-tools-inside-the-digitalpreservation-workflow/)

On validation:

> The surmise is that readers of a file format (the software) will have been created alongside a specification, and that correcting out-of-band features will lead to a greater number of readers per format being able to handle it. 

On property extraction:

> We extract properties so that file formats can be well understood to promote preservation?

In general:

> If we have tools in our digital preservation system doing the work of format identification, validation, and extraction, they’re there for a reason. 

## Risk model

Thread:

<https://digipres.club/@bitsgalore/110457949965653023>

> One of my main gripes with all the work being done on interpreting JHOVE PDF validation errors, is the lack (as far as I can see) of a clearly defined risk model: what are the actual risks we want to mitigate, and where's the evidence JHOVE's output actually helps with this?

> For long-term access there are other risks, and those risks don't necessarily follow from validation errors. So why this fixation on validation errors? And why this preoccupation with JHOVE, and putting all this time and effort in fixing and documenting a partially outdated tool that is plagued by 20 years of technical debt?

> having one tool that handles many formats may be convenient, but if the functionality doesn't match requirements (which should follow from the risk model + policies) it all becomes a pretty pointless box-ticking exercise.

> It's a bit like using a hammer for assembling a piece of furniture that has scam screw connectors, because we ONLY have a hammer, and it's a very nice hammer (slightly over the top analogy, but you get the idea).

## A valediction for validation?

In 2018, Paul Wheatley wrote ["A valediction for validation?"](https://www.dpconline.org/blog/a-valediction-for-validation), which reflects on file format validation, with a focus on PDF. He observes that "there is a significant disconnect between validation and the digital preservation answers we need". He mentions several factors that are contributing to this, including:

- Flaws of the validation tools themselves.
- Tool reports with microscopic level minutiae that are often difficult to interpret.
- The fact that even if the format validation is complete and thorough, other issues that are important from a digital preservation point of view may go unnoticed.

Some good examples of the last point are the presence of encrypted content or external dependencies. Even though both may be allowed by the format specification, these features are nevertheless obstacles against successful rendering.

## Preservation risk assessment

He also argues that validation is often treated as an end in itself, without much consideration for the purpose it is supposed to fulfil. As a way out of this situation, he first stresses the importance of clearly stating the purpose. In Paul's view, validation is part of a broader risk assessment of digital content[^9]. The overall goal of this risk assessment is to answer the following questions: 

> Does this digital object render without error, accurately and usefully for the user, and is it likely to render in the future? If not, should we do something about that?

The risk assessment should then ensure that a file is not encrypted, doesn't have any external dependencies, isn't badly broken, and has the expected file format[^8]. Even though this won't 100% guarantee that a file will render, at least it covers the most likely obstacles against a file successfully rendering.

## Format validation vs rendering tools

Within this risk assessment, "format validation" largely serves as an automated proxy for rendering a file in a viewer (which would be too laborious to do manually). The main purpose of the validation tool here is to "highlight the (hopefully) small number of our files that warrant further investigation via manual assessment". Ultimately, Paul suggests that this particular task could be done even better by dropping validation tools altogether, and that it might be more worthwhile to invest in adapting rendering tools to do this job instead.

## Links

- [A valediction for validation?](https://www.dpconline.org/blog/a-valediction-for-validation) - blog post by Paul Wheatley


[^8]: As per the result of the file format identification tool that was used.

[^9]: What makes things slightly confusing is that throughout his post, Paul appears to use the word "validation" to indicate two different things: first, as a process that evaluates a file against a format specification (narrow definition), and second as a process that includes the full risk assessment, including checking for encryption and external dependencies (broad definition). In my post I'm sticking to the narrow definition, but in doing so I had to re-state my summary of Paul's suggestions in those terms.


## Further resources

[post on identifying PDF preservation risks]({{ BASE_PATH }}/2023/05/25/identification-of-pdf-preservation-risks-with-verapdf-and-jhove)

[VeraPDF parse status as a proxy for PDF rendering]({{ BASE_PATH }}/2023/06/29/verapdf-parse-status-as-a-proxy-for-rendering)