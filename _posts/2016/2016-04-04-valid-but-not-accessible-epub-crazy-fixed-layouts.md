---
layout: post
title: Valid, but not accessible&#58; crazy fixed EPUB layouts
tags: [EPUB]
comment_id: 28
---

[*EpubCheck*](https://github.com/IDPF/epubcheck) is an invaluable tool for assessing the quality of *EPUB* files. Still, it is possible that *EPUB*s that are valid according to the format specification (and thus *EpubCheck*) are nevertheless inaccessible to some users. Some weeks ago a colleague sent me an *EPUB* 2 file that produced some really strange behaviour across a number of viewer applications. For a start, the text wouldn't reflow properly after re-sizing the viewer window, and increasing the font size resulted in garbled text. Running the file through *EpubCheck* did return some validation errors, but none of these were related to the behaviour I was getting. Closer inspection revealed some very peculiar stylesheet and *HTML* use.

<!-- more -->

## Crazy Fixed Layout

As I cannot share the original file for rights reasons, I fired up the [*Sigil*](https://sigil-ebook.com/) e-book editor and made a handcrafted *EPUB* that reproduces its behaviour. You can [download the file here](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_crazy_fixed_layout.epub?raw=true). If you open it in an e-book viewer, it will probably look perfectly normal at first sight. For example, here's a screenshot I made using the [*Calibre*](https://calibre-ebook.com/) viewer:

![]({{ BASE_PATH }}/images/2016/04/calibre_normal.png)

Next I reduced the width of the viewer window. One would expect the text to re-flow to the new width. Instead this happened:  

![]({{ BASE_PATH }}/images/2016/04/calibre_resized_screen.png)

After increasing the font size, I ended up with this:   

![]({{ BASE_PATH }}/images/2016/04/calibre_largefont.png)

I got similar results in [*Chome's Readium* extension](https://chrome.google.com/webstore/detail/readium/fepbnnnkkadjhjahcafoaglimekefifl). On my e-Ink reader, a Sony PRS-T2, the book rendered as follows:

![]({{ BASE_PATH }}/images/2016/04/sony_fixedlayout.png)

However, I wasn't able to change the font size.

## Analysis

The file [passes validation in *EpubCheck* 4.0.1](https://github.com/KBNLresearch/epubPolicyTests/blob/master/epubcheckout/4.0.1/epub20_crazy_fixed_layout.xml) without errors. However, the output does contain a series of warnings about the use of absolute positions in a stylesheet:

```data
CSS-017, WARN, [CSS selector specifies absolute position.], OEBPS/Styles/styles.css (6-2)
CSS-017, WARN, [CSS selector specifies absolute position.], OEBPS/Styles/styles.css (24-1)
CSS-017, WARN, [CSS selector specifies absolute position.], OEBPS/Styles/styles.css (43-1)
::
```

To really understand what causes the problem, we need to look inside the file's *HTML* and *CSS* resources. Here's some of the [*HTML* that underlies the text](https://github.com/KBNLresearch/epubPolicyTests/blob/master/content/epub20_crazy_fixed_layout/OEBPS/Text/Section0001.xhtml):

```html
<p id="p01" class="para">This is an <em>EPUB</em> 2 file that uses a fixed layout.</p>
<p id="p02" class="para">This is achieved by placing each line inside a</p>
<p id="p03" class="para"><em>paragraph</em> element. Each <em>paragraph</em> element</p>
<p id="p04" class="para">is placed at a fixed position on the page. Even</p>
<p id="p05" class="para">though this file is valid <em>EPUB</em>, this is a pretty</p>
<p id="p06" class="para"> terrible idea, because in most readers the text</p>
<p id="p07" class="para">will not reflow after resizing the viewer window.</p>
```

So, every line is wrapped inside a *paragraph* element, each of which has a unique *id* selector. These refer to style definitions in the [*EPUB*'s stylesheet](https://github.com/KBNLresearch/epubPolicyTests/blob/master/content/epub20_crazy_fixed_layout/OEBPS/Styles/styles.css). Here are the definitions for the first two lines:

```css
#p01
{
position:absolute;
left:40px;
top:80px;
letter-spacing:0.42px;
word-spacing:0.1em;
}
#p02
{
position:absolute;
left:40px;
top:120px;
letter-spacing:0.42px;
word-spacing:0.1em;
```

Each style definition specifies a line's position on the canvas (*left*, *top*); moreover, these co-ordinates are defined as *absolute* positions. This means that each line is placed at a fixed position, regardless of whether this makes any sense given the actual dimensions of the viewer window (or device), or the user's preferred font size. It seems that the intention of the producer of the original *EPUB* (from which I derived my example) was to create some sort of ["fixed layout"](http://www.idpf.org/epub/fxl/) document. However, this doesn't make much sense for books with simple, text-only layouts (as in this case). Worse, depending on the viewing device and the user the file may be effectively inaccessible. For example, someone with a visual impairment may only be able to read an *EPUB* using very large font sizes, which in this case results in garbled text. 

## Crazy Columns

Things can even get worse. I once came across an *EPUB* that used similar tricks to achieve a two-column layout. Again I'm not able to share the original file, so I created [another *EPUB* that mimicks its behavour](https://github.com/KBNLresearch/epubPolicyTests/blob/master/build/epub20_crazy_columns.epub?raw=true). In the *Calibre* viewer it looks like this:

![]({{ BASE_PATH }}/images/2016/04/calibre_columns.png)

As with the first example, the text doesn't reflow after resizing the viewer window, and increasing the font resulted in this:

![]({{ BASE_PATH }}/images/2016/04/calibre_columns_largefont.png)

This is what I got when I opened the file in my Sony e-Ink reader:

![]({{ BASE_PATH }}/images/2016/04/sony_crazycolumns1.png)

After I increased the font size this happened:

![]({{ BASE_PATH }}/images/2016/04/sony_crazycolumns2.png)

Similarly, when I tried to copy the text in the file to the clipboard, and then pasted it in a text editor, I ended up with this:

> This is an EPUB filepage. Even though thisthat uses a two-columnfile is valid EPUB, there'slayout. For each column,no way to establish theevery line is placed atlogical reading order ofa fixed position on thethe text.

Ouch!

## Analysis

Again, throwing this file at *EpubCheck* 4 [doesn't result in any validation errors](https://github.com/KBNLresearch/epubPolicyTests/blob/master/epubcheckout/4.0.1/epub20_crazy_columns.xml), although just like the previous file there are some warnings about the use of absolute positions in the stylesheet:

```data
CSS-017, WARN, [CSS selector specifies absolute position.], OEBPS/Styles/styles.css (13-1)
```
A peek [inside the *HTML*](https://github.com/KBNLresearch/epubPolicyTests/blob/master/content/epub20_crazy_columns/OEBPS/Text/Section0001.xhtml) reveals the true horrors of this *EPUB*. This is how the text is encoded:

```html
<div class="pos" style="left: 40px; top: 100px;">This is an <em>EPUB</em> file<div>
<div class="pos" style="left: 260px; top: 100px;">page. Even though this</div>
<div class="pos" style="left: 40px; top: 140px;">that uses a two-column</div>
<div class="pos" style="left: 260px; top: 140px;">file is valid <em>EPUB</em>, there's</div>
```

So, every line of each column is wrapped in a division element that has a fixed position. The class *pos* in the [stylesheet](https://github.com/KBNLresearch/epubPolicyTests/blob/master/content/epub20_crazy_columns/OEBPS/Styles/styles.css) defines the general layout of each division element. In this case, it specifies that all positions are (again) absolute:

```css
.pos {position:absolute;
}
```
    
Technically this is pretty similar to the first example. Note that the above *HTML* doesn't contain any semantic information on the fact that there are two separate columns. Worse, the order of the text in the HTML doesn't even follow the actual reading order! This also explains the results after copying and pasting. [Screen reader](https://en.wikipedia.org/wiki/Screen_reader) applications will not be able to handle this either, which makes books like these inaccessible to many visually impaired users. All of this could have been avoided if the book's producer had followed the [W3C multi-column layout specification](https://www.w3.org/TR/css3-multicol/).

## Conclusion

I don't know how common (or rare) *EPUB*s like the above are. They may just be weird edge cases. Nevertheless, their existence indicates that checking for validity alone may not be sufficient to ensure accessibility for all users (in particular those with a visual impairment). In any case, files like these can be identified relatively easily by checking *EpubCheck*'s output for the presence of a *CSS-017* warning ("CSS selector specifies absolute position")[^1]. These examples also underline the importance of guidelines and best practices. Several good resources for making accessible *EPUB* are available from the [*EPUB* 3 Accessibility Guidelines](http://www.idpf.org/accessibility/guidelines/), including a useful [Accessibility QA Checklist](http://www.idpf.org/accessibility/guidelines/content/qa/qa-checklist.php). I would also be interested in hearing other people's experiences with "weird" *EPUB*s like these.

## Postscript

Alberto Pettarin pointed me to his blog post [*(Current) Fixed Layout eBooks Considered Harmful*](http://www.albertopettarin.it/blog/2015/02/21/current-fixed-layout-ebooks-considered-harmful.html). Written in 2015, it addresses the problems with current implementations of fixed layouts in *EPUB*, and if you found this blog post interesting, I would suggest to check out Alberto's blog as well. 
 
Alberto's [Twitter feed](https://twitter.com/acutebit/status/718031931221360640) also drew my attention to an interesting *EPUB* with the program of the recent *EPUB* Summit in Bordeaux. You can [download it here](http://edrlab.org/edrlab/wp-content/uploads/2016/04/EDRLabprogram_EN_HD_final.epub_.zip) (you need to unzip it first!). The file is interesting because:

1. It does not pass validation by *EpubCheck* (the mimetype file entry is not the first file resource in the archive)
2. It uses a fixed, multi-column layout that doesn't scale in either *Readium* or *Calibre*'s viewer (changing the font size has no effect), and I'm wondering if it is usable at all on any handheld devices!

There's some irony in that this file was published by [*EDRLab*](http://edrlab.org/edrlab/), an organisation that describes itself as "the European headquarter for IDPF and Readium Foundation", and which mentions "support for people who have print disabilities" as a "key part"of its mission. Oh well ...

## Link to dataset

The *EPUB*s used for this blog post are part of the [EPUB KB policy testing repository](https://github.com/KBNLresearch/epubPolicyTests). This is an annotated set of openly licensed *EPUB* files that were specifically created for testing purposes.

[^1]: Note that *EpubCheck* 3 (now outdated) [does not report this warning](https://github.com/KBNLresearch/epubPolicyTests/blob/master/epubcheckout/3.0.1/epub20_crazy_columns.xml), so always use *EpubCheck* 4.
<hr>
Originally published at the [KB Research blog](http://blog.kbresearch.nl/2016/04/04/valid-but-not-accessible-epub-crazy-fixed-layouts/)
