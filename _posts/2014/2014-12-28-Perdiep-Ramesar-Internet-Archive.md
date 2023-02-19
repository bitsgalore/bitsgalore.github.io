---
layout: post
title: Perdiep Ramesar in het Internet Archive
tags: [web-archiving]
comment_id: 38
---
Eerder deze week [verwijderde dagblad *Trouw* 126](http://www.nrc.nl/nieuws/2014/12/20/trouw-trekt-126-artikelen-van-perdiep-ramesar-in/) artikelen van haar website die geschreven waren door ontslagen journalist Perdiep Ramesar. Aanleiding hiervoor was het [onderzoek](http://static1.trouw.nl/static/asset/2014/Onderzoeksrapport_bronnengebruik_Trouw_19122014_7707.pdf) naar door Ramesar opgevoerde "niet traceerbare" bronnen. De beslissing van *Trouw* om de onbetrouwbare artikelen van de site af te halen stuitte op nogal wat kritiek. Sommigen noemden het [geschiedvervalsing](http://www.journalismlab.nl/2014/12/perdiep-gewist-gaan-trouw-en-ad-gaan-voor-geschiedvervalsing/). Historicus Jan Dirk Snel [merkte terecht op](http://jandirksnel.wordpress.com/2014/12/24/geschiedvervalsing-het-echte-schandaal-bij-trouw-is-nu-pas-begonnen/) dat nu de stukken zijn verwijderd, niemand meer kan controleren wat er eventueel wel of niet aan deugt.

<!-- more -->

## Vindbaarheid in Internet Archive

Uit nieuwsgierigheid heb ik van een aantal van de verwijderde artikelen gekeken of ze nog te vinden waren in het [Internet Archive](https://archive.org/). Voor sommige stukken bleek dit inderdaad het geval. Vervolgens werd ik benieuwd hoeveel van de 126 verwijderde artikelen nog vindbaar zouden zijn. Het probleem hierbij is alleen dat het Internet Archive niet echt makkelijk doorzoekbaar is. Om een artikel op het spoor te komen, heb je eigenlijk de originele URL (dus op de *Trouw* website) nodig. *Trouw* heeft wel een [lijst met de verwijderde artikelen](http://static3.trouw.nl/static/asset/2014/Artikelen_met_niet_verifieerbare_bronnen_Ramesar_2007_2014_7708.pdf) gepubliceerd, maar hierin wordt van elk artikel alleen de *titel* vermeld, en niet de volledige link.

Omdat de artikelen nog maar recent zijn verwijderd, zitten de URLs nog wel in de cache van de meeste zoekmachines. Door de titels uit de lijst met verwijderde artikelen in te voeren in *Google* en *DuckDuckGo*, lukte het me om van alle 126 artikelen de originele URLs te achterhalen[^1]. Met behulp van een zelfgeschreven [scriptje](https://github.com/bitsgalore/trouwRamesarWayback/blob/master/scripts/checkLinksInWayback.py) heb ik vervolgens elke URL opgezocht in het Internet Archive. Dit leverde me een [lijst]({{ BASE_PATH }}/images/2014/12/ramesarTrouwURLSWayback.csv) op met -voor elk artikel- de status in Internet Archive (is het gearchiveerd of niet), en, indien aanwezig, de meest recent gearchiveerde versie.

## Resultaat

Het resultaat van de hierboven beschreven analyse heb ik samengevat in [deze tabel]({{ BASE_PATH }}/images/2014/12/tabelRamesar.html). Van de 126 verwijderde artikelen zijn er 53 nog opvraagbaar in het Internet Archive. Het gaat hierbij vooral om artikelen uit 2010 en later; uit de periode 2007-2009 is nog maar weinig te vinden.

## Nog meer te vinden?

Overigens verwacht ik dat met goed zoeken nog wel meer te vinden valt: voor zover ik het het goed begrijp, wordt een artikel op de *Trouw* website eerst onder een *nieuwslink* gepubliceerd; vervolgens verhuist het naar het archief, waarna het onder een *archieflink* beschikbaar is. Een voorbeeld is het artikel *Ik kan mezelf niet veranderen in een witte man*. Een zoekactie met *DuckDuckGo* leverde me hiervan de volgende link op:

<http://www.trouw.nl/tr/nl/5009/Archief/archief/article/detail/3287592/2012/07/17/Ik-kan-mezelf-niet-veranderen-in-een-witte-man.dhtml>

Deze *archieflink* is niet te vinden in Internet Archive. Op de site van Jan Dirk Snel kwam ik van hetzelfde artikel de *nieuwslink* tegen:

<http://www.trouw.nl/tr/nl/4504/Economie/article/detail/3287689/2012/07/17/Ik-kan-mezelf-niet-veranderen-in-een-witte-man.dhtml>

En die zit wel in Internet Archive:

<http://web.archive.org/web/20141224185904/http://www.trouw.nl/tr/nl/4504/Economie/article/detail/3287689/2012/07/17/Ik-kan-mezelf-niet-veranderen-in-een-witte-man.dhtml>

Er zullen dus nog wel meer artikelen op vergelijkbare wijze door het net zijn geglipt. Als lezers nog aanvullingen of correcties hebben dan hoor ik dat natuurlijk graag!

## AD

Het *AD* is nog veel verder gegaan dan *Trouw*, en heeft gelijk *alle* artikelen waarvan Ramesar auteur is verwijderd. Van veel van deze stukken [zijn de originele URLs nog te achterhalen via de *Google* cache](https://www.google.nl/?q=%22Perdiep+Ramesar%22+site:ad.nl#q=%22Perdiep+Ramesar%22+site:ad.nl). Maar niet lang meer! Omdat het hier om honderden artikelen gaat, is het geen doen om de URLs allemaal handmatig op te vragen. [*Google* biedt een *Search API*](https://developers.google.com/custom-search/json-api/v1/overview) aan, en daarmee zou het mogelijk moeten zijn om dit grotendeels te automatiseren. Die URLs kun je vervolgens weer proberen terug te zoeken in Internet Archive, net zoals ik dat voor de *Trouw* artikelen heb gedaan. Ik ga daar zelf nu geen tijd in steken, maar misschien heeft iemand anders zin om hiermee aan de slag te gaan. Enige haast is hierbij wel geboden, want binnen een paar weken zullen de links uit *Google*'s cache verdwenen zijn, en het is maar de vraag of je het dan nog ooit terug kunt vinden.

**[Klik hier voor de volledige lijst artikelen, inclusief originele URLs en URLs in Internet Archive (voor zover beschikbaar)]({{ BASE_PATH }}/images/2014/12/tabelRamesar.html)**


## Links

* [Data als kommagescheiden tekstbestand (UTF-8)]({{ BASE_PATH }}/images/2014/12/ramesarTrouwURLSWayback.csv)
* [Github repository met gebruikte scripts en databestanden](https://github.com/bitsgalore/trouwRamesarWayback)
* [ZIP bestand met alle scripts en databestanden](https://github.com/bitsgalore/trouwRamesarWayback/archive/master.zip)

[^1]: Google heeft hierbij de vervelende gewoonte om niet de directe links naar de zoekresultaten te geven. Om dit te omzeilen heb ik de volgende FireFox add-on gebruikt: <https://palant.de/2011/11/28/google-yandex-search-link-fix>
