---
layout: post
title: On The Significant Properties of Spreadsheets
tags: [spreadsheets, TIFF, PDF, preservation-risks, significant-properties]
comment_id: 77
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2021/09/clippy-800.png" alt="Clippy saying It looks like you're migrating a spreadsheet to ... TIFF?!">
</figure>

Earlier this month saw the publication of [The Significant Properties of Spreadsheets](https://zenodo.org/record/5468116). This is the final report of a six-year research effort by the Open Preservation Foundation's Archives Interest Group (AIG), which is composed of participants from the National Archives of the Netherlands (NANETH), the National Archives of Estonia (NAE), the Danish National Archives (DNA), and Preservica. The report caught my attention for two reasons. First, there's the subject matter of spreadsheets, on which I've written a few posts in the past[^1]. Second, it marks a surprising (at least to me!) return of "significant properties", a concept that was omnipresent in the digital preservation world between, roughly, 2005 and 2010, but which has largely fallen into disuse since then. In this post I'm sharing some of my thoughts on the report.

<!-- more -->

## Overall aim

The authors describe the rationale behind their work as follows: 

> Preserving files in spreadsheet formats is a priority for every member. We need to answer questions such as ‘should we migrate?’ and ‘how do we measure the success or quality of the migration?’. For the latter, we need to know what aspects of the file are important (significant), which led us to the decision to investigate the significant properties of spreadsheets.

Various definitions of "significant properties" exist. The authors followed the 2007 [InSPECT report](https://significantproperties.kdl.kcl.ac.uk/wp22_significant_properties.pdf) here, which defines them as:

> the characteristics of digital objects that must be preserved over time in order to ensure the continued accessibility, usability, and meaning of the objects, and their capacity to be accepted as evidence of what they purport to record.

The authors describe the role of "significant properties" in the preservation process as follows:

> When the digital objects (e.g. files in a format) or the technology to use them (e.g. viewers) are at risk of becoming obsolete, preservation actions may be required (e.g. file format migration or viewer software emulation). Ensuring that the significant properties are reasonably preserved as a result of these preservation actions is then a means of validating these actions.

So, by this view, the importance of "significant properties" lies in their utility to validate preservation actions.

## Simple versus complex spreadsheets

Throughout the report, the authors differentiate between what they call "simple" (or "static") and "complex" (or "dynamic") spreadsheets. They define "simple" spreadsheets as:

> spreadsheets that are mainly used for (human) visualisation and contain static data values organised into tabular format on one or more worksheets.

Irrespective of whether you agree with the "simple" versus "complex" distinction, the above definition is problematic because it mixes up actual spreadsheet characteristics ("contains static data") with a subjective statement of how such spreadsheets are supposed to be used ("mainly used for human visualization"). Moreover, it's not at all clear what the "human visualization" assessment is based on (some written intention statement by the original creators, an institutional policy, or something else?).

By contrast, "complex" spreadsheets are defined as:  

> spreadsheets that contain formulae, notes, macros, dates, links to external data sources or other functions or dynamic behaviour.

## Migrating spreadsheets to TIFF and PDF/A

The authors explain that this distinction was motivated by a practical format migration use case. In particular, on page 10 they write that (emphasis added by me):

> simple spreadsheets would likely render more or less the same in most spreadsheet-rendering applications at every moment of time. **One would lose no information when migrating to other, primarily rendering-oriented file formats, like the Tagged Image File Format (TIFF) currently accepted by DNA.**

On page 11, they continue:

> In case of ‘simple/static’ spreadsheets with formatting (fonts, colours, styles, cell width, etc.), no significant loss of information would occur if the spreadsheets were migrated, to e.g. TIFF or PDF/A.

And:

> In short, ‘simple/static’ spreadsheets can be migrated to non-spreadsheet specific file formats or formats that are not meant to preserve dynamic behaviour.

These are truly mind-boggling statements, and I hardly know where to even start here. First, migrating a spreadsheet to an image format like TIFF would result in the loss of *most* of the information of the source document, including *all* coded text and numbers. This would break machine-readability, and would also make the data values inaccessible to visually impaired users. Most of the functionality of the source file would be lost as well, such as the ability to navigate and access individual cells, rows and columns, search and filter cell values, and copy content, to name but a few. These objections apply to PDF/A as well (albeit to a somewhat lesser degree). I already addressed this [in my 2016 post on PDF/A as a preferred, sustainable format for spreadsheets]({{ BASE_PATH }}/2016/12/09/pdfa-as-a-preferred-sustainable-format-for-spreadsheets) (which, incidentally, is even cited in the report). Statements like these show a fundamental lack of understanding of digital information and file formats, and I'm surprised (and frankly a bit shocked) to find them in a contemporary report by digital preservation professionals[^8].

On "complex" spreadsheets, the authors note that (page 11):

> Migrating to non-spreadsheet formats could cause severe information loss.

While this is of course true, this largely applies to "simple" spreadsheets as well. Also, migrating to other spreadsheet formats could result in information loss as well.

## Identification of significant properties

The main outcome of the work is a categorised list of "significant properties" of spreadsheets. The authors arrived at this by adapting the [methodology from the InSPECT project](https://significantproperties.kdl.kcl.ac.uk/inspect-finalreport.pdf). I found the description of the InSPECT methodology and its application to spreadsheets in the AIG report quite hard to follow in places, so in this section I'll attempt to provide a brief (and somewhat simplified) summary.

The InSPECT methodology involves two main components. The first one is the "object analysis". The overall objective here is the compilation of an extensive list of spreadsheet properties, based on sample files, characterisation tools and technical specifications. These were then categorised into "property groups". Subsequently, the "property groups" were linked to expected "user behaviours" (e.g. "View data in cells"), which were classified into more generic "functions". The associations between functions, behaviours and property groups are visualised in what the authors call the "spaghetti diagram", which can be viewed [here](https://zenodo.org/record/5468116/files/FBS%20diagram%20%28final%20report%29.png).

A particularly interesting strand of the object analysis work, was the development of a [Spreadsheet Complexity Analyser](https://github.com/RvanVeenendaal/Spreadsheet-Complexity-Analyser). This is a tool that classifies Microsoft Excel spreadsheets (both XLS and XLSX) as "simple" or "complex", based on extracted document- and cell-level properties (which can be reported as well).

The second component is a "stakeholder analysis". Here, the authors presented the properties and property groups from the object analysis to various types of stakeholders (e.g. archivists, users), and asked them which properties they deemed "significant".

The authors combined the results from the "object analysis" and the "stakeholder analysis", which resulted in [this list](https://zenodo.org/record/5468116/files/Combined%20%28relevant%20and%20significant%20properties%29.xlsx?download=1). Out of the 334 properties in the list, 105 were deemed "significant" by the stakeholders at the individual property level. At the property group level, only 15 out of the 38 property groups were deemed "significant", which corresponds to 140 properties (i.e. all properties that are part of these 15 groups).

In the following sections I will comment on some things that caught my attention.

## Specificity of properties

One thing that struck me while browsing the [list of properties](https://zenodo.org/record/5468116/files/Combined%20%28relevant%20and%20significant%20properties%29.xlsx?download=1), is that some of the identified properties are very general and lack specificity. As an example, properties such as "Database Functions", "Engineering Functions" and "Date and Time Functions" only describe broad *categories* of spreadsheet functions, but not the actual functions themselves. I don't really understand the value of these categories within the context of validating preservation actions (which, after all, is ultimately the larger aim of this work). As an example, imagine we migrate an XLSX spreadsheet to ODS, and for some weird reason the [*WEEKDAY*](https://support.microsoft.com/en-us/office/weekday-function-60e44483-2ed1-439f-8bd0-e404c190949a) function is changed to [*YEAR*](https://support.microsoft.com/en-us/office/year-function-c64f017a-1354-490d-981f-578e8ec8d3b9). Both are "Date and Time Functions", but the result of such a migration would still be nonsense. What we'd really like to know in this case, is whether *the exact functions* are preserved. Moreover, according to [the Document Foundation Wiki](https://wiki.documentfoundation.org/Feature_Comparison:_LibreOffice_-_Microsoft_Office#Desktop_Spreadsheet_applications:_LibreOffice_Calc_vs._Microsoft_Excel) both Excel and LibreOffice Calc and have some functions that are unique to each application (29 and 22, respectively). This is a potential source for information loss when migrating between their respective formats. A significant properties approach would only be able to account for such information losses if the properties are defined at the level of individual spreadsheet functions[^6].Needless to say, this would increase the number of properties to take into account considerably, and in the absence of any way to automate this it would also make things even more time consuming.

The properties related to macros[^7] provide another example. Microsoft Excel uses [Visual Basic for Applications](https://en.wikipedia.org/wiki/Visual_Basic_for_Applications) (VBA) as the macro language for its XLS and XLSX formats. However, the Open Document Format does not dictate any specific macro or scripting language, and simply declares this as [implementation dependent](https://docs.oasis-open.org/office/OpenDocument/v1.3/os/part3-schema/OpenDocument-v1.3-os-part3-schema.html#attribute-script_language). By default, LibreOffice uses its own Basic implementation, which is [largely incompatible with Microsoft's VBA language](https://help.libreoffice.org/latest/en-US/text/shared/guide/ms_user.html). In addition, it allows the use of other languages such as [Python](https://wiki.documentfoundation.org/Macros/Python_Guide/Introduction) and [JavaScript](https://webodf.org/blog/2012-04-13.html). This variety of (mostly incompatible) macro languages introduces various preservation challenges. In spite of this, "macro language" or "scripting language" is not included in the "macros" property group. This raises some questions on how useful these properties actually are for validating preservation actions.

## How to measure the properties

Related to this, the authors provide no explanation *how* any of the identified properties can be measured, and using what software. My best guess is that they used some spreadsheet application[^4]. If this is indeed the case, how would one even start to do this at scale, with collections of thousands of spreadsheets? Again, the authors don't even mention this. The Spreadsheet Complexity Analyser could be really useful here, and in fact, DNA tried this approach as part of the object analysis work (p. 19):

> The Danish National Archives ran the SCA against about 16,000 Microsoft Excel spreadsheets (both binary formats and OOXML) to investigate the possible information loss when converting Excel spreadsheets to ODS. (...) The test showed that the conversion from XLS and XLSX to ODS and back to XLS and XLSX resulted in minimal data loss. Yet, data loss for significant structures such as cell typographies, fonts and hyperlinks were encountered.

However, the Spreadsheet Complexity Analyser is currently only able to extract 13 spreadsheet-specific properties[^5], which is only a fraction of all the properties that were deemed "significant". So while this is an admirable start, it doesn't even come close to what would be needed for any real-world application.

## Where are the examples, case studies?

The introductory chapter of the AIG report explicitly mentions how the work is aimed at validating preservation actions. However, it is remarkably vague on how the outcomes of the inSPECT methodology could help in solving real-world spreadsheet preservation challenges. The authors do mention application areas such as preservation actions (e.g. format migration and emulation) and choosing preferred formats. However, they don't provide any examples or case studies that demonstrate how the identified properties, property groups and user behaviours would help here in practice. The closest thing to an actual example is given in the concluding chapter, where they write (p. 39):

> One example is that the Danish National Archives used the gained knowledge of spreadsheet properties in the decision to revise their accepted formats and adopt a spreadsheet-specific format, which probably will be the Open Document Spreadsheet format.

This refers to DNA's current practice of accepting TIFF(!) as a preservation format for spreadsheets. The authors then write:

 > \[O\]ur work provided the lists, tools and insights DNA required to revise their accepted format policy and adopt a spreadsheet-specific format in their Preservation Policy.

It's not my intention to mock DNA's odd choice of TIFF here[^3], but that TIFF is a terrible target format for spreadsheets should be blindingly obvious to anyone with even the slightest understanding of spreadsheets or digital formats in general. You really don't need to spend six years researching hundreds of "significant properties" to arrive at this conclusion[^2]!

Because of the absence of any further examples or case studies, it remains unclear how the authors envisage the use of their work in any real-world format migration or emulation scenario. For a start, the sheer number of properties would make any manual, non-automated analysis extremely cumbersome and time consuming. Interestingly, this is confirmed by the following quote from the DNA stakeholder analysis (p. 18):

> Our experiences from the stakeholder interviews were that it can be extremely time and competency consuming to analyse every single property and behaviour for a complex content information type such as spreadsheets. In fact, for these kinds of analyses it can be counterproductive to conducting the interview if we do not try to stray away from the InSPECT approach and instead focus on facilitating a meaningful conversation with people and from this conversation try to deduce the behaviours necessary to preserve for future reuse of the data. The questions you instead can ask people are what they deem important to be able to do with the data and what data and associated functionality do they find important to preserve, if they were to reuse it in the future.

I'm not surprised by this observation, as it is consistent with earlier work by Euan Cochrane. In his 2012 [Rendering Matters report](https://web.archive.org/web/20130218111126/http://archives.govt.nz/rendering-matters-report-results-research-digital-object-rendering), he showed that for a migration-based approach, at least 13.5 hours were needed to test only 100 Office files (a mixture of word-processing, spreadsheet, database and presentation formats) comprehensively. This doesn't inspire much confidence in using such analyses at scale for any real-world applications.

## Significance in context

Some of the problems that I outlined in the previous sections could be remedied to some extent by introducing even more (and more detailed) properties, or by adding support for more properties to the Spreadsheet Complexity Analyser. However, this would make the analyses even more complex and time consuming. This shouldn't come as a surprise, as Webb, Pearson & Koerbin already pointed out some of these difficulties in their [2013 D-Lib paper](http://www.dlib.org/dlib/january13/webb/01webb.html):

> We have come to a tentative conclusion that recognising and taking action to maintain significant properties will be critical, but that the concept can be more of a stumbling block than a starting block, at least in the context of our own institution. We believe reference to significant properties in preservation planning requires some prior consideration of both the purposes for which digital content has been collected and the purposes of providing preservation attention. In effect, we are asking how can we know what attributes of digital materials we need to preserve if we haven't articulated why we are preserving them?

Specifically, they ran into two problems. The first one was the complexity of defining the measurable levels of properties that were required to fulfill a collection's preservation intention. The second one was the realisation that potential changes to digital objects as a result of some preservation action can best be evaluated around the time the preservation action takes place, using the tools that are then available. This doesn't mean they discarded the concept of significant properties altogether, but rather they decided that the definition of significant properties is informed by practical experience with (the tools used to perform) preservation actions.

Owens is also skeptical about the significant properties approach in his 2018 book [The Theory and Craft of Digital Preservation](https://osf.io/preprints/lissa/5cpjt):

> At one point, many in the digital preservation field worked to try and identify context agnostic notions of the significant properties of various kinds of digital files and formats. While there is some merit to this approach, it has been largely abandoned in the face of the realization that significance is largely a matter of context. That is, significance isn’t an inherent property of forms of content, but an emergent feature of the context in which that content was created and the purpose it serves in an institutions’ collection.

The over-arching problem of much of the work presented in the AIG report, is the almost total lack of any such context. The result is, that valuable time and effort are spent on the analysis of a myriad of atomic properties, many of which will only matter in certain specific contexts, and not at all in others. So, before embarking on any preservation action, I would like (at the absolute minimum) answers to questions such as: 

- Who are the (present and future) users of the spreadsheet collections?
- What accessibility levels are required to meet these users needs (e.g. viewing in original environment, navigating, editing, accessibility for visually impaired users, machine-readability)?
- What preservation actions are needed to enable these accessibility requirements (e.g. emulation, migration to one or more access formats)?

The answers to such questions largely fit into the "preservation intent" concept, which was (if I'm not mistaken) first introduced by Webb, Pearson & Koerbin:

> The Library's preservation intent methodology is simply to engage collection curators in making explicit statements about which collection materials, and which copies of collection materials, need to remain accessible for an extended period, and which ones can be discarded when no longer in use or when access to them becomes troublesome. Curators are also asked to make broad statements clarifying what 'accessible' means by stating the priority elements that need to be re-presented in any future access for each kind of digital object type in their collections.

Owens considers preservation intent to be one of the foundations of any digital preservation work:

> In this model, preservation intent and collection development function are the foundation of your digital preservation work. All of the work one does to enable long-term access to digital information should be grounded in a stated purpose and intention that directly informs what is selected and what is retained.

One important implication is that there is no "one size fits all" solution here. For example, the properties that are "significant" for an authentic rendering in an emulator will be quite different from those for a machine-readable access copy, and such nuances are not apparent from the AIG report, which seems to take the view that digital preservation is merely a matter of preserving the (contextless) "significant properties" (p. 41):

> \[I\]f you look closely at your digital preservation strategy, it boils down to finding the best way to preserve the significant properties of information.

By contrast, taking the preservation intent as a starting point, a practical solution for the above case might look something like this: 

- Always keep the original spreadsheets (in their original formats)
- Use emulation to ensure faithful rendering of these files in their original environment
- If needed, use migration to derive access copies in more convenient formats to suit specific use cases (e.g. text and data mining)[^10].

This would not require any complex and time-consuming analysis of significant properties, because the original data are preserved in their original formats. The Spreadsheet Complexity Analyser could possibly help validate the migration to the access formats (at least to some degree).

## Representativity of stakeholder group

In the inSPECT methodology, the "significance" that is attributed to a property depends to a large degree on the background and knowledge level of the stakeholders. To their credit, the authors readily acknowledge this in the report, and remark that:

> A stakeholder that never makes use of more advanced features, such as formulas and macros, will not deem these significant.

A [2009 report by Dappert and Farquhar](https://planets-project.eu/docs/papers/Dappert_Significant_Characteristics_ECDL2009.pdf) already stressed how "significance" is largely in the eye of the stakeholder. So, one would expect that the stakeholders would be somewhat representative of the (future) users of the spreadsheet collections. However, the authors restricted their stakeholder analysis population to "individuals that are employed in the public sector". They justify this by stating that: 

> This is due to the fact that the organisations for which this study is carried out, the National Archives of various countries, preserve information from public institutes.

I understand the logic behind limiting the size of the stakeholder group (if only for practical reasons), but the justification seems to imply that these National Archives see their depositing institutes as their only (or primary) users. This strikes me as an overly narrow view, as national archives also serve independent researchers, authors, (data) journalists, and members of the general public. It is not clear to what extent the actual (potential) users were represented in the current stakeholder analysis. For instance, a researcher or data journalist may attribute high importance to the ability to access spreadsheet collections through text and data-mining techniques. For a visually impaired user, accessibility through a screenreader application will be vitally important. 

## User behaviours

In addition, one component of the inSPECT methodology's stakeholder analysis is the collection of "actual behaviours", which are defined as "activities that a specific category of stakeholder will likely perform when using the object". These are then mapped against properties, which eventually determines to a large degree which properties are deemed "significant". However, the authors decided that this would be too daunting a task (p. 15):

> With spreadsheets, however, this is an immense task. Stakeholders use spreadsheets for a wide range of activities with no established set of functions that has to be used every time. Furthermore, we felt this would be difficult to accomplish thoroughly during interviews with stakeholders, considering the size of the task. Therefore, these last steps were not performed by us during this research.

Instead, they limited themselves to drafting a list of "expected behaviours", which inSPECT defines as "the different types of activities that a user – any type of user – may wish to perform". But even this turned out to be overly ambitious as well (p. 21, emphasis added by me):

> We soon realised that we would never establish an exhaustive list of all possible behaviours and **chose to use those behaviours that we found most important from our perspective as archives preserving spreadsheets**.

## Significant to (future) users?

The combined effects of the restricted stakeholder population, not taking into account "actual behaviours", and limiting the "expected behaviours" to an archivist's perspective raises some major concerns. Most importantly, it calls into question the degree to which the "significance" judgements truly reflect what is "significant" to actual and future users (which should be informed by the preservation intent). Ultimately, digital preservation is largely about preserving digital materials for future *use*. If accounting for actual user behaviours, needs and requirements is too gargantuan a task, shouldn't this lead to the inevitable conclusion that, at least in this situation, the inSPECT methodology is not suitable as a basis for validating preservation actions[^9]?

## Conclusion

The AIG report on the significant properties of spreadsheets leaves me with some very mixed thoughts. On the one hand, it is clear that a lot of time, effort and dedication have gone into this work. The detailed breakdown of properties in the ["blue sheet"](https://zenodo.org/record/5468116/files/List%20of%20properties%20and%20property%20groups%20%28blue%20sheet%29.ods?download=1) contains a wealth of information that, I expect, will be of interest to any digital preservationist working on spreadsheet preservation. I particularly like the [Spreadsheet Complexity Analyser](https://github.com/RvanVeenendaal/Spreadsheet-Complexity-Analyser), which looks like a genuinely useful tool. 

With that said, it's not clear to me how the results of the main body of this work would support its own stated aim of validating preservation actions, and the authors provide no examples of this. It almost seems that they got lost in a labyrinthine web of properties, and lost sight of this overall aim along the way.

I'm also concerned about the largely context-agnostic view of "significant properties" that the authors express throughout the report. On the one hand, this results in a scope that will be unnecessarily wide for most real-world preservation actions (meaning that the number of properties to consider becomes unmanageable quickly). Simultaneously, the authors made several decisions that, taken together, have the effect that the needs and requirements of (future) users of the spreadsheet collections are only minimally represented. I expect this will seriously limit the utility of this exercise as a basis for making preservation decisions.

## Link to report

[The Significant Properties of Spreadsheets (OPF AIG Final Report)](https://zenodo.org/record/5468116)

## Revision history

- 8 October 2021: added mention of Euan Cochrane's Rendering Matters report to examples, case studies section.

[^1]: See my posts on [Quattro Pro for DOS]({{ BASE_PATH }}/2014/10/29/quattro-pro-dos-obsolete-format-last) and [PDF/A as a preferred, sustainable format for spreadsheets?]({{ BASE_PATH }}/2016/12/09/pdfa-as-a-preferred-sustainable-format-for-spreadsheets).

[^2]: It's worth noting here that a footnote on page 31 mentions how DNA are working on revising their format policy because the TIFF format "will not support the migration of dynamic properties". It also mentions that "\[o\]ur research has made clear how significant these are and why they must be preserved". This suggests that despite spending six years of research on the significant properties of spreadsheets, DNA still see no problem with TIFF as a target format for "simple"/"static" spreadsheets.

[^3]: I suspect this might have it roots in some ill-advised decision in some distant past. I think most memory institutions will have examples of similarly unfortunate historically-evolved practices, and DNA should be praised here for being open about this.

[^4]: E.g. (some version of) Microsoft Excel, LibreOffice Calc, or perhaps something else?

[^5]: The total number of extracted properties is actually 18, but 5 of these are filesystem-level level attributes (file name, size and time stamps) that are not specific to spreadsheets.

[^6]: Also, I cannot even imagine how one would measure these function categories, as to the best of my knowledge they are not even explicitly coded in the spreadsheet.

[^7]: I should add here that macros were not considered "significant" by the stakeholders.

[^8]: The discussion of the Danish National Archives Stakeholder Analysis on page 29 provides a more realistic view, which is absolutely clear on why migrating to TIFF is a horrible idea for spreadsheets.

[^9]: Which, by the authors' own account, is the underlying aim of this work.

[^10]: Even migration to some image format might be useful as supporting evidence of how a spreadsheet should be rendered in its original environment (but please use PNG, not TIFF).
