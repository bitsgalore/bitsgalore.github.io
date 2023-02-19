---
layout: post
title: Running archived Android apps on a PC&#58; first impressions
tags: [Android,emulation,virtualization]
comment_id: 41
---

Earlier this week I had a discussion with some colleagues about the archiving of mobile phone and tablet apps (iPhone/Android), and, equally important, ways to provide long-term access. The immediate incentive for this was an announcement by a Dutch publisher, who recently published a children's book that is accompanied by its own app. Also, there are already several examples of Ebooks that are published exclusively as mobile apps. So, even though we're not receiving any apps in our collections yet, we'll have to address this at some point, and it's useful to have an initial idea of the challenges that may lie ahead.

<!-- more -->

The scope of this blog is *not* to provide any in-depth coverage of the long-term preservation of mobile apps. Instead, I was just curious about two specific aspects: 

* Is it possible to run a phone app on a regular PC, and, if yes, how? 
* How can you use this to run an archived copy of an app?

I spent a few afternoons running some preliminary tests. Since I'm pretty sure other institutions must be looking into this as well, I thought I might as well share the results, as well as some useful resources I came across along the way. For now I limited myself to the [Android](http://www.android.com/) platform ([iOS](http://en.wikipedia.org/wiki/IOS) presents additional challenges because of its restrictive license).

## The Android app format

First of all it is helpful to know a bit more about the Android app format. Basically, an app (*.apk*) file is just a ZIP archive with a specific file and directory structure. It is based on Java's [Jar](http://fileformats.archiveteam.org/wiki/Jar) format. For more information see this entry on Archive Team's file format wiki:

<http://fileformats.archiveteam.org/wiki/APK>

Here you can also find how to download local copies of Android app files (e.g. to a PC), which is something that is not possible directly from the  [Google Play store](https://play.google.com/store/apps?hl=en).

## Running Android on a PC

If you want to run Android on a regular (Linux or Windows) PC, several options exist. [This article](http://www.extremetech.com/computing/83812-run-android-apps-on-your-windows-pc-2) gives a good general overview. The "best" solution according to its author is a [third-party developed app player](http://www.bluestacks.com/app-player.html). However, that player only works under Windows, it is proprietary, and non-free: to use it, you either pay a monthly fee, or put up with "sponsored apps". Google's Android SDK also includes an [emulator](http://developer.android.com/tools/help/emulator.html), which is mainly targeted at app developers. I didn't look into that now, mainly because of its alleged poor performance. Instead, I went for a third option.

## Android on VirtualBox

The [Android-x86 Project](http://www.android-x86.org/) has created a port of the [Android Open Source Project](http://source.android.com/) that runs on [X86-based](http://en.wikipedia.org/wiki/X86) architectures. This opens op the possibility to run Android on an ordinary PC, either as the main operating system, or in a virtual machine. So, I decided to take the latter route and installed Android on a virtual machine using [VirtualBox](https://www.virtualbox.org/). This is relatively straightforward, and several excellent step-by-step descriptions on how to do this exist, for instance:

* [How to Install Android in VirtualBox](http://www.howtogeek.com/164570/how-to-install-android-in-virtualbox/) ([archived link](http://web.archive.org/web/20141023111241/http://www.howtogeek.com/164570/how-to-install-android-in-virtualbox/)); and

* [How to install Android 4.4 KitKat in Windows using VirtualBox](http://www.fixedbyvonnie.com/2014/02/install-android-4-4-kitkat-windows-using-virtualbox/) ([archived link](http://web.archive.org/web/20141023111321/http://www.fixedbyvonnie.com/2014/02/install-android-4-4-kitkat-windows-using-virtualbox/)).  
 
The latest ISO images from Android-x86 can be found here (for this test I used version 4.4):

<http://sourceforge.net/projects/android-x86/files/>

## Running and waking up Android

Setting up the virtual machine was easy enough, and Android appeared to work well straight away. One thing that can be confusing for first-time users is the way VirtualBox deals with mouse input: once you click your mouse in the screen area that is occupied by the virtual machine, VirtualBox shows a dialog asking whether it should "capture" the mouse. Once you click *Capture*, the mouse can only be used inside the virtual machine (and not for any other applications that are running on the host machine). You can  *uncapture* the mouse at any time by pressing the right-hand *Ctrl* key. Another thing that initially puzzled me, is that Android enters sleep mode after several minutes of inactivity, resulting in a black screen. Once in sleep mode, it is not very obvious how to wake it up again (even rebooting the VM didn't do the trick). After some searching I found that [the solution](http://www.sysads.co.uk/2014/01/install-android-4-3-virtualbox-screenshots) here is to press the [Menu key](http://en.wikipedia.org/wiki/Menu_key) on the keyboard (located next to the right-hand *Ctrl* key on most keyboards), which instantly brings the machine back to life.

## Moving an app to the virtual machine 

To install an app in Android, you would normally go to the [Google Play store](https://play.google.com/store/apps?hl=en). In an archival setting it is more likely that you already have an archived copy stored somewhere, so what we need here is the ability to install from a local [APK file](http://fileformats.archiveteam.org/wiki/APK). This is also known as "side loading", and [this article](http://www.cnet.com/how-to/how-to-install-apps-outside-of-google-play/) gives general instructions on how to do this with a physical device. Since we're running Android on a virtual machine here, things are a bit different, and ideally we should  be able to share a folder between the host machine and the (virtual) guest device. In theory this is all possible in VirtualBox, but as it turns out [it doesn't work](http://superuser.com/questions/665696/shared-folder-in-virtualbox-with-android-not-working) because [Android-86 doesn't support VirtualBox Guest Additions](http://stackoverflow.com/questions/8235165/getting-vbox-guest-addtions-for-android-x86). As a workaround, I ended up uploading my the APK to DropBox, and then opened DropBox in Android's web browser to download the file. 

## Installing the app

The downloaded APK is now located in the *Download* folder, which is accessible using Android's file browser. After clicking on it, the following security warning popped up:

![]({{ BASE_PATH }}/images/2014/10/installBlocked.png)

This is because, by default, apps from unknown sources (i.e. other than the [Google Play store](https://play.google.com/store/apps?hl=en)) are blocked by Android. The solution here is to click on *Settings*, which opens up the security settings dialog: 

![]({{ BASE_PATH }}/images/2014/10/tickUnknownSources.png)

Here, check the *Unknown sources* option [^1]. Then go back to the *Downloads* folder and click on the APK file again. It will now install the app.

## Final thoughts

In this blog I provided some basic information about Android's APK format, how to run Android in VirtualBox, and how to install an archived app. I tested this myself with a handful of apps. One thing I noticed was that some apps didn't quite work as expected on my virtual Android machine, but as I didn't have access to a 'real' (physical) device it's impossible to tell whether this  was due to the virtualisation or just a shortcoming of those apps. This would obviously need more work. Nevertheless, considering that I only spent a few  odd afternoons on this, this approach looks quite promising.

[^1]: Note that this will also enable you to install apps that are possibly harmful; use with care!

<hr>
Originally published at the [Open Preservation Foundation blog](https://openpreservation.org/blog/2014/10/23/running-archived-android-apps-pc-first-impressions/)
