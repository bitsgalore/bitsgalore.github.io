---
layout: post
title: How to preserve your personal Twitter archive
tags: [web-archiving, Twitter, digital-dark-age]
comment_id: 82
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2022/11/fail_whale_by_ka_92-d5ra7vf.jpg" alt="Painting of a whale, which is lifted above the beach by a swarm of red Twitter birds. In the foreground, an elderly couple watches the scene.">
  <figcaption><a href="https://web.archive.org/web/20130204212401/https://ka-92.deviantart.com/art/fail-whale-348157275">"Fail whale"</a> by <a href="https://web.archive.org/web/20130128112538/http://ka-92.deviantart.com/">Kuni (ka-92)</a> (license unknown), based on "Lifting a Dreamer" by <a href="http://www.yiyinglu.com/?portfolio=lifting-a-dreamer-aka-twitter-fail-whale">Yiying Lu</a>.</figcaption>
</figure>

As a total collapse of Twitter is becoming more likely every day, many Twitter users have started to archive their personal data from the platform while it still exists. Twitter allows you to request and download your personal archive. Even though this works well, and the quality of the archive is surpringly good, it does have some shortcomings. The result of these shortcomings will be that some information in the archive (e.g. on followed accounts and followers) will be lost once Twitter ceases to exist. Some other information (in particular full, unshortened URLs) *is* included in the archive, but it is not easily accessible from the main HTML interface. The good news is, that some excellent tools exist to fix these shortcomings.

In this post I outline the workflow I used to preserve my own Twitter archive, and while doing so I also provide some background information on the shortcomings of the Twitter archive. Since some of these steps may, at first sight, be a little daunting for less tech-savvy readers, I've tried to provide step-by-step instructions where possible.

<!-- more -->

## Disclaimer

First a little disclaimer: I don't want to suggest that what follows is the "best", "proper" or even a "good" way to do this. For most of the issues I'm addressing here, several alternatives exist, and some may be better than what I'm suggesting here. Also, an in-depth analysis of the Twitter archive, and a comparison of different tools and approaches are both beyond the scope of this post (besides I don't have the time for this). Ultimately, all I'm describing here is a workflow that, for now, looks "good enough" for my own purposes. I'm sharing it here because I *think* it will be "good enough" for many others as well, or at least provide a reasonable starting point. With that said, I can't give any guarantees here, so keep this in mind while reading what follows!

## Request and download Twitter archive

Most importantly, you need to request and download your Twitter data[^4]. The basic steps are:

1. While logged in to Twitter, go to [your account's settings](https://twitter.com/settings/account).
1. Click on "Download an archive of your data", and then simply follow the instructions.
1. Twitter will issue a notification when the archive is ready:
    ![]({{ BASE_PATH }}/images/2022/11/archive-notification.png)
   This may take a while. When I did this two weeks ago, I had to wait over 24 hours; various people have told me that currently it take several days. Once ready, you can download the archive as one large ZIP file.
1. Once downloaded, unzip the file to a folder.

## Shortcomings of the Twitter archive

As several people have pointed out before, the archive data provided by Twitter have a number of shortcomings. Below a (probably incomplete) overview of the most obvious ones. 

### Shortened t.co links

Perhaps most importantly, if you access your Twitter archive from your web browser by opening the "Your archive.html" file, any clickable hyperlinks you see are shortened t.co links that use [Twitter's link shortener service](https://web.archive.org/web/20221116161843/https://help.twitter.com/en/using-twitter/url-shortener):

![]({{ BASE_PATH }}/images/2022/11/tweet-link.png)

These links will stop working once Twitter goes down. The original, unshortened links are actually stored in the underlying JSON data that are part of the archive (file "tweets.js" in the "data" folder). For example, [here's the full data from the Tweet in that screenshot](https://gist.github.com/bitsgalore/cfdff3ce67f1ffa85f67e87c778a9e75). Let's zoom in on its "urls" attribute:


```json
 "urls" : [
          {
            "url" : "https://t.co/y2gpEVvjAd",
            "expanded_url" : "https://youtu.be/C47ZCosJPAw",
            "display_url" : "youtu.be/C47ZCosJPAw",
            "indices" : [
              "246",
              "269"
            ]
          }
        ]
```

As you can see, the "urls" attribute contains both a shortened t.co link ("url"), the unshortened link ("expanded_url"), and a display link ("display_url"). So all the data are there, but the unshortened links just aren't accessible from the archive's web interface.

### Full-size images

The Twitter archive only contains downscaled versions of posted images. Clicking on an image to expand it takes you to the live Twitter website. Again, this is something that will stop working once Twitter is gone.

### Twitter network data

Although the Twitter archive does contain files with the accounts that you follow and the accounts that follow you, these are given as numerical identifiers that will most likely be meaningless if Twitter disappears. Here's an example:

```json
  {
    "following" : {
      "accountId" : "216697909",
      "userLink" : "https://twitter.com/intent/user?user_id=216697909"
    }
  }
```
Here, the user link with account ID 216697909 (<https://twitter.com/intent/user?user_id=216697909>) resolves to [the account of the Open Preservation Foundation](https://twitter.com/openpreserve). Once Twitter is gone, this user link will stop resolving, which will make it very hard to figure out which actual person or organization was associated with it. 

Fortunately, some excellent solutions exist that can fix most of the above shortcomings, and they are pretty easy to use as well. Let's start with addressing the network data issue, as this is something you can do immediately, without having to wait for your Twitter archive.

## Preserve your Twitter network with FediFinder 

[FediFinder](https://fedifinder.glitch.me/) is an online tool written by Luca Hammer. Its main purpose is to find Fediverse accounts that correspond to your Twitter connections. However, it also works great for tracking your entire Twitter network, including your followers, accounts that you follow, and list members.

Just follow these steps:

1. While you're logged into your Twitter account, go to <https://fedifinder.glitch.me/>

1. Click on the "Authorize to extract handles" button (this will give FediFinder read permissions to your Twitter account):

    ![]({{ BASE_PATH }}/images/2022/11/ff-authorize.png)

1. In the page that appears, click on "Scan followings", "Scan followers" and "Load lists". For each list, click on "Scan members". If all goes well you'll see something like this:

    ![]({{ BASE_PATH }}/images/2022/11/ff-scan-finished.png)

1. Scroll past the list of search results to the bottom of the page, and click on the small "accounts.csv" link[^1]:

    ![]({{ BASE_PATH }}/images/2022/11/ff-accounts.png)

Once downloaded, you can open the file in any spreadsheet software. For each account, it contains the Twitter user name, the real name, any lists of which the account is a member, associated Fediverse handles, the account's location, and its profile description.

Optionally, afterwards you may want to remove FediFinder from your Twitter account's connected app list using [this link](https://twitter.com/settings/connected_apps/):

![]({{ BASE_PATH }}/images/2022/11/ff-revoke.png)

## Improve archive with Twitter archive parser

Tim Hutton has written [twitter-archive-parser](https://github.com/timhutton/twitter-archive-parser), which is a Python tool that fixes most of the remaining issues, and some other issues as well[^5]. Most importantly, it creates both HTML and [Markdown](https://en.wikipedia.org/wiki/Markdown) versions of the archive, with all shortened t.co URLs replaced with their original versions. Optionally, it can also be instructed to download full-size versions of images[^3].

To use it, follow these steps:

1. Install Python 3 on your system if you don't have it already. If you're a Windows user and you're not sure how to do this, check out [Alberto Pettarin's easy to follow instructions](https://github.com/pettarin/python-on-windows).

1. Then download the twitter-archive-parser script. For this, just right-click [this link](https://raw.githubusercontent.com/timhutton/twitter-archive-parser/main/parser.py), select "Save link as", and save the file into the folder where you extracted the archive[^2].

1. Start a command-prompt or terminal, and change the working directory to the folder where you extracted the archive. Windows users may want to have another look at [Alberto Pettarin's explainer](https://github.com/pettarin/python-on-windows), in particular the "Using The Command Prompt" and "Changing The Working Directory" sections.

1. Run the twitter-archive-parser script from the command-prompt or terminal, using the following command:

    ```bash
    python parser.py
    ```

   Depending on your operating system, you may need to replace "python" with "python3":

    ```bash
    python3 parser.py
    ```

   The script will most likely ask you to confirm the installation of one or two Python modules ("requests" and "imagesize"); if this happens, just type "y" and press the Enter key.

1. Finally, the script will ask if you want to download original-size images. Type "y" if you want this, or "n" if not. Note that downloading the images can take quite a bit of time (depending on the size of your Twitter archive, and the number of images and multimedia it contains).

   As an aside, I noticed that archive parser was unable to download some images. I don't particularly care about this myself, but if images are crucially important to you, the "media" folder contains a download log (file "download_log.txt") with full details of the download status of each image.

And that's all there is to it!

## Alternative approaches and variations

As I already mentioned in the introduction, I'm not making any claims that the above steps are the "proper" way to do this, and various alternative approaches exist. In this section I'll highlight a few of these.

After I published the first draft of this post, I found [this earlier post by Jeroen Wiert Pluimers](https://wiert.me/2022/11/12/exporting-your-twitter-content-converting-to-markdown-and-getting-the-image-alt-texts-thanks-isotopp-hbeckpdx-for-the-info-and-kcgreenn-dreamjar-for-the-comic/). This describes an overall workflow that is similar to the one described here (it also uses archive parser), but adds the extraction of  alt-text image descriptions, exporting of bookmarks, and archiving of t.co URL shortener links to the the [Wayback Machine](https://web.archive.org/).

Ed Summers has written an [alternative t.co unshortener](https://github.com/docnow/twitter-archive-unshorten), which is explained [in this blog post](https://inkdroid.org/2022/11/20/t-dot-co/). Ed has also [written this post on archiving Twitter bookmarks](https://inkdroid.org/2022/11/16/bookmarks/), which are not included in the archive data. And [here's a Ruby script by Ryan Baumann](https://gist.github.com/ryanfb/53f167feebde61ad262c4f09d879733e) that exports your Twitter Bookmarks to JSON (note that the scripts deletes the original bookmarks to get around API limits). As I never use bookmarks, I'm not really interested in this myself, but this might be important to some users.

Mike Hucka's [Taupe tool](https://github.com/mhucka/taupe) extracts URLs from tweets, retweets, replies, quote tweets, and "likes" from a personal Twitter archive, and writes these to a comma-delimited text file. This is especially useful if you want to preserve linked resources (e.g. by sending them to a web archive).

[Tweetback](https://github.com/tweetback/tweetback/) is an open-source software project by Zach Leatherman that creates a static website from your Twitter archive. See [this blog post](https://www.zachleat.com/web/tweetback/) for more information about it, as well as some links to static websites that were created with it. This looks like a really interesting option for those who want to publish their Twitter archive.

[Twitter-nest](https://github.com/sqiouyilu/twitter-nest) is a set of tools by S. Qiouyi Lu that allow you to create a decentralized Twitter clone on WordPress.

Internet Archive has also made it possible to upload your Twitter archive to its [Wayback Machine](https://archive.org/web/). See [this article for instructions](https://help.archive.org/help/how-to-archive-your-tweets-with-the-wayback-machine/).

Finally [this TechCrunch feature](https://techcrunch.com/2022/11/21/quit-twitter-better-with-these-free-tools-that-make-archiving-a-breeze/) mentions some more tools that might be worth perusing.

## Final thoughts

It's good to keep in mind that the development of tools like archive parser currently [moves at a pretty fast pace](https://digipres.club/web/@timhutton@mathstodon.xyz/109377490421206529). Just as an example, when I ran archive parser only yesterday (19th of November), it wasn't able to report Twitter followers and followings, whereas this functionality is included in the latest (20th of November) release. So I expect these tools will become even better over time (but don't wait for it, as there's a real chance that Twitter may be gone by then!).

Please feel free to use the comment section to post links to alternative tools or methods, or if you spot any glaring errors in this post. 

## Additional links and resources

- [Twitter archive parser](https://github.com/timhutton/twitter-archive-parser) ("related tools" section lists some more more tools that might be useful)

- [FediFinder](https://fedifinder.glitch.me/)

- [A step-by-step guide on installing Python and using the Command Prompt for Windows](https://github.com/pettarin/python-on-windows)

- [Ed Summers on archiving Twitter bookmarks](https://inkdroid.org/2022/11/16/bookmarks/)

- [Ryan Baumann - Exporting As Many of Your Twitter Bookmarks As Possible](https://ryanfb.github.io/etc/2022/11/21/exporting_as_many_of_your_twitter_bookmarks_as_possible.html)

- [Ruby script by Ryan Baumann that exports Twitter bookmarks to JSON](https://gist.github.com/ryanfb/53f167feebde61ad262c4f09d879733e) (this also deletes the original bookmarks to get around API limits, so use with caution!)

- [Alternative t.co unshorten approach by Ed Summers](https://inkdroid.org/2022/11/20/t-dot-co/)

- [twitter-archive-unshorten tool](https://github.com/docnow/twitter-archive-unshorten)

- [Jeroen Wiert Pluimers on exporting your Twitter content, converting to Markdown and getting the image alt-texts](https://wiert.me/2022/11/12/exporting-your-twitter-content-converting-to-markdown-and-getting-the-image-alt-texts-thanks-isotopp-hbeckpdx-for-the-info-and-kcgreenn-dreamjar-for-the-comic/)

- [Taupe tool - extracts URLs from Twitter archive](https://github.com/mhucka/taupe)

- [Quit Twitter better with these free tools that make archiving a breeze (TechCrunch)](https://techcrunch.com/2022/11/21/quit-twitter-better-with-these-free-tools-that-make-archiving-a-breeze/)

- [Tweetback by Zach Leatherman](https://github.com/tweetback/tweetback/)

- [Blog post on Tweetback](https://www.zachleat.com/web/tweetback/)

- [Twitter-nest by S. Qiouyi Lu](https://github.com/sqiouyilu/twitter-nest)

- [How to archive your Tweets with the Wayback Machine](https://help.archive.org/help/how-to-archive-your-tweets-with-the-wayback-machine/)

## Revision history

- 21 November 2022: updated info about Twitter's notification when the archive download is ready.

- 21 November 2022: added references to Ryan Baumann's bookmarks export script.

- 21 November 2022: added references to earlier blog post by Jeroen Wiert Pluimers.

- 22 November 2022: added references to Taupe tool by Mike Hucka.

- 23 November 2022: added references to TechCrunch feature.

- 29 November 2022: added references to Tweetback.

- 1 December 2022: added references to Twitter-nest.

- 15 December 2022: added references to Wayback archiving; renamed and re-strucured final sections.

[^1]: The large "Export fedifinder_accounts.csv" link will give you a file that only includes Fediverse accounts. This can be useful for automating your follows on Mastodon, but if you want detailed information on *all* Twitter accounts you (also) need to use the small link!

[^2]: The name of this folder looks something like this:<br>
      "twitter-2022-11-07-2366bc80316...4e7b77".

[^3]: The version of Archive parser I used wasn't able to produce lists of followers and followings. More recent versions (20 November 2022 an onward) do have this functionality, but the output is less detailed than FediFinder. Also, it doesn't distinguish between direct follows and follows through a list. So it's probably a good idea to use FediFinder for this.

[^4]: More details can be found [here](https://help.twitter.com/en/managing-your-account/accessing-your-twitter-data), although some of the info looks slightly out of date.

[^5]: Among other things, it also fixes some issues with Direct Messages, which by default don't include user handles.