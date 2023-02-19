---
layout: post
title: Dutch newspaper wipes out articles citing fabricated sources - Internet Archive to the rescue!
tags: [web-archiving]
comment_id: 37
---

Shortly before Christmas, Dutch daily newspaper *Trouw* [removed 126 articles](http://www.nrc.nl/nieuws/2014/12/20/trouw-trekt-126-artikelen-van-perdiep-ramesar-in/) from its website. These articles were all authored by Perdiep Ramesar, a former journalist of the newspaper. Ramesar had been fired by *Trouw* in November, after it turned out that many of the sources that are cited in his articles were [fabricated](http://static1.trouw.nl/static/asset/2014/Onderzoeksrapport_bronnengebruik_Trouw_19122014_7707.pdf). The most notorious example was a series of pieces about the so-called "Sharia Triangle", a neighbourhood in the city of The Hague, which Ramesar claimed was being ruled  by Sharia law. As it turned out, this story was largely based on fabricated sources. Nevertheless, it was taken at face value by most major Dutch news outlets at the time, and even prompted a [parliamentary debate](http://www.tweedekamer.nl/kamerstukken/detail?id=2013D34540&did=2013D34540).

*Trouw*'s decision to remove the 126  articles overnight was met with considerable criticism. For example, historian Jan Dirk Snel [noted](http://jandirksnel.wordpress.com/2014/12/24/geschiedvervalsing-het-echte-schandaal-bij-trouw-is-nu-pas-begonnen/) that the removal of these articles makes it impossible to check *what* was wrong with them in the first place. Various other critics accused *Trouw* of trying to [rewrite history](http://www.journalismlab.nl/2014/12/perdiep-gewist-gaan-trouw-en-ad-gaan-voor-geschiedvervalsing/).  

<!-- more -->

## Internet Archive to the rescue?

A quick check on a handful of Ramesar's articles revealed that quite a few were still accessible from the [Internet Archive's Wayback Machine](http://archive.org/web/). This got me curious how many out of the 126 deleted articles would still be available there. Answering this question isn't completely straightforward, because the Wayback Machine isn't easily searchable. In order to locate any of the deleted articles, one first needs to know its original URL (i.e. the one at *Trouw*'s website). A [list of all deleted articles](http://static3.trouw.nl/static/asset/2014/Artikelen_met_niet_verifieerbare_bronnen_Ramesar_2007_2014_7708.pdf) does exist, but this only provides each article's *title*, without listing the full URL.

However, by entering each title into a search engine (I used a combination of *Google* and *DuckDuckGo*[^1]), I was able to recover the original URL of every article in the list. In many cases the URLs were still present in the cache of the search engine. In other cases URLs could be recovered from linking pages on the *Trouw* website. I then wrote a simple [script](https://github.com/bitsgalore/trouwRamesarWayback/blob/master/scripts/checkLinksInWayback.py) to check the availability of each URL in Internet Archive's Wayback Machine. The script is just a wrapper around Wayback's [Availability JSON API](https://archive.org/help/wayback_api.php), which is insanely handy (and really easy to use as well!). This yielded a [list]({{ BASE_PATH }}/images/2015/01/ramesarTrouwURLSWayback.csv) with -for each article- its status in Wayback (i.e. has it been archived), and, if so, the URL to the most recent capture. 

## Result

The results of the above exercise are summarised in [this table]({{ BASE_PATH }}/images/2015/01/tabelRamesar.html). As it turned out, 53 out of the 126 deleted articles are still accessible from the Internet Archive. These are mostly pieces that were written from 2010 onward, and include the notorious "Sharia Triangle" ones. From the time period 2007-2009 very few articles could be found. 

## Possibly more?

It may be possible that more removed articles are hidden in the Internet Archive. This is because of the way the *Trouw* website handles news items. If I understand things correctly, articles in *Trouw* are often first published under a *news* URL; subsequently it is moved to the *archive* section of the website, where it is published under a different URL. By way of illustration, a  *DuckDuckGo* search of the article *Ik kan mezelf niet veranderen in een witte man* yielded the following URL: 

<http://www.trouw.nl/tr/nl/5009/Archief/archief/article/detail/3287592/2012/07/17/Ik-kan-mezelf-niet-veranderen-in-een-witte-man.dhtml>

This *archive* link (recognisable from the word *archief* in the URL) cannot be found anywhere in Internet Archive. By chance I encountered a different link to the same article on the website of historian Jan Dirk Snel:

<http://www.trouw.nl/tr/nl/4504/Economie/article/detail/3287689/2012/07/17/Ik-kan-mezelf-niet-veranderen-in-een-witte-man.dhtml>

This is the *news* link under which the article was first published, and a snapshot of it exists in the Internet Archive:

<http://web.archive.org/web/20141224185904/http://www.trouw.nl/tr/nl/4504/Economie/article/detail/3287689/2012/07/17/Ik-kan-mezelf-niet-veranderen-in-een-witte-man.dhtml>

Likewise, I expect that some articles may have slipped through the net in a similar way. Nevertheless, I think the above results are pretty good as they are!

**[Click here for the full list of removed articles, includes both original URLs and URLs in Internet Archive (if available)]({{ BASE_PATH }}/images/2015/01/tabelRamesar.html)**


## Links

* [Data as comma-separated text file (UTF-8)]({{ BASE_PATH }}/images/2015/01/ramesarTrouwURLSWayback.csv)
* [Github repo with scripts and raw data files](https://github.com/bitsgalore/trouwRamesarWayback)
* [Github repo (as single ZIP file)](https://github.com/bitsgalore/trouwRamesarWayback/archive/master.zip)

[^1]: In order to get unscrambled links from Google, I used the following FireFox add-on: <https://palant.de/2011/11/28/google-yandex-search-link-fix>
