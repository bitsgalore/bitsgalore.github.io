---
layout: post
title: Towards a preservation workflow for mobile apps
tags: [Android, iOS, APK, IPA, Apache-Tika, Siegfried, unix-file, format-identification]
comment_id: 75
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2021/02/stewardess-phone.jpg" alt="Satellite image of Wadden Sea">
  <figcaption>Production photo from "2001: A Space Odyssey". ©Stanley Kubrick Archives/TASCHEN.</figcaption>
</figure>

My [previous post]({{ BASE_PATH }}/2021/02/09/four-android-emulators-two-apps) addressed the emulation of mobile Android apps. In this follow-up, I'll explore some other aspects of mobile app preservation, with a focus on acquisition and ingest processes. The [2019 iPres paper on the Acquisition and Preservation of Mobile eBook Apps](https://zenodo.org/record/3460450) by Maureen Pennock, Peter May and Michael Day again was the departure point. In its concluding section, they recommend:

> In terms of target formats for acquisition, we reach the undeniable conclusion that acquisition of the app in its packaged form (either an IPA file or an APK file) is optimal for ensuring organisations at least acquire a complete published object for preservation.

And:

> \[T\]his form should at least also include sufficient metadata about inherent technical dependencies to understand what is needed to meet them.

In practical terms, this means that the workflows that are used for acquisition and (pre-)ingest must include components that are able to deal with the following aspects:

1. Acquisition of the app packages (either by direct deposit from the publisher, or using the app store).
2. Identification of the package format (APK for Android, IPA for iOS).
3. Identification of metadata about the app's technical dependencies.

The main objective of this post is to get an idea of what would be needed to implement these components. Is it possible to do all of this with existing tools? If not so, what are the gaps? The underlying assumption here is an emulation-based preservation strategy[^14].

<!-- more -->

## Outline of this post

As for the acquisition component, Pennock, May and Day recommend direct publisher deposit, as this may avoid some potential problems related to digital rights management and dependencies on content that is hosted remotely. Since we've only started exploring mobile app preservation at this stage, it's too early to make any assumptions about what acquisition route would work best in our case. Because of this, I started out by investigating to what extent it is possible to download APK and iOS packages from the Google Play Store and the Apple App Store, respectively.

I then tried to do automatic format identification on some sample APK and IPA packages, using recent versions of [Siegfried](https://www.itforarchivists.com/siegfried/), [Unix File](http://darwinsys.com/file/) and [Apache Tika](http://tika.apache.org/).

Next, I looked at how the APK and IPA formats store metadata about an app's technical dependencies, and how this information can be extracted.

The following sections discuss these components for both the Android and iOS platforms. For convenience I included a summary of the main findings at the end of this post, followed by some observations on the value of additional documentation such as video recordings that show mobile apps in action. 

## Downloading Android packages

Android apps are distributed as [Android Package (APK)](https://en.wikipedia.org/wiki/Android_application_package) installer files through the [Google Play Store](https://play.google.com/store/apps?hl=en). However, the Play Store doesn't allow you to download APK files on a non-Android device, which is a problem within a preservation workflow. Various third-party websites exist that offer the possibility to download APK installers, but it is often difficult to establish their trustworthiness. Some of these sites also re-package the original app data, which introduces various concerns related to security and authenticity. Because of this, I would strongly advise against using any of these services within a preservation workflow.

### Virtual machine method

A better (but somewhat cumbersome) solution would be to set up a virtual machine or emulator with Android[^1], and use that to install the app directly from the Play store. Once installed, it is possible to transfer the APK installer file from the virtual machine to the host machine using the [Android Debug Bridge (adb) tool](https://developer.android.com/studio/command-line/adb). This involves the following steps:


1. Get a list of all installed packages on the virtual machine (redirecting output to a text file):
```bash
adb shell pm list packages > packages.txt
```
2. Look up the package id of the installed app in this file. Taking the [ARize app](https://play.google.com/store/apps/details?id=com.Triplee.TripleeSocial) from my previous post as an example, the identifier is `com.Triplee.TripleeSocial`. We can then use the following command to find the full file path of the package on the VM:
```bash
adb shell pm path com.Triplee.TripleeSocial
```
  Result:
```
package:/data/app/com.Triplee.TripleeSocial-r8iVFUp1MOSAc6LmHA1MDQ==/base.apk
```
3. Use the above file path to download the package to the host machine:
```bash
adb pull /data/app/com.Triplee.TripleeSocial-r8iVFUp1MOSAc6LmHA1MDQ==/base.apk
```

In this case, this results in a file "base.apk".

### Gplaycli method

As the above method is a bit clumsy, I started looking for tools that allow downloading packages from the Play Store directly. Several such open-source tools exist, but many of these are abondoned projects that no longer work. After trying out a few of them, I utimately had success with [gplaycli](https://github.com/matlink/gplaycli), which is "a command line tool to search, install, update Android applications from the Google Play Store". It allows you to download an APK, using the App ID as an identifier. Taking the ARize app as an example again, we can download the APK with the following command[^2]:

```bash
gplaycli -d com.Triplee.TripleeSocial
```

This resulted in a file "com.Triplee.TripleeSocial.apk". I verified the file by doing a bitwise comparison against the APK obtained from the "virtual machine method" described in the previous section[^3]. This confirmed both files were identical. It's worth mentioning that gplaycli is also [reported to work for downloading paid apps](https://github.com/matlink/gplaycli/issues/8) (provided the proper login credentials are used), but I haven't tested this.

## Android package identification

As most archival ingest workflows include a format identification component, I tried to identify the ARize and Immer Android packages from my previous post with 3 widely used format identification tools. The table below shows the results:

|Tool|Version|ID|
|:--|:--|:--|
|[Siegfried](https://www.itforarchivists.com/siegfried/)|1.9.1; DROID Signature File V97[^7]|x-fmt/412 (Java Archive Format)<br>x-fmt/263 (ZIP Format)|
|[Unix File](http://darwinsys.com/file/)|5.32|application/zip|
|[Apache Tika](http://tika.apache.org/)|1.23|application/vnd.android.package-archive|

Apache Tika was the only tool that identified both files as Android packages. Siegfried (which uses the [PRONOM](https://www.nationalarchives.gov.uk/PRONOM/Default.aspx) format signatures) identified one file as a regular ZIP file, and the other one as a [Java Archive](https://en.wikipedia.org/wiki/JAR_(file_format)). Since the Android package format is based on the Java Archive format (which is in turn a subset of the ZIP format) this result is not necessarily wrong, but it lacks specificity. At the time of writing, the [PRONOM](https://www.nationalarchives.gov.uk/PRONOM/Default.aspx) technical registry does not have an entry for the Android package format[^6], so this result is not surprising. A look at [Tika's Mimetype definition file](https://github.com/apache/tika/blob/618345263ee41108e1a225dbcdbb8db16b2aae28/tika-core/src/main/resources/org/apache/tika/mime/tika-mimetypes.xml#L316) reveals that Tika only uses the file extension to differentiate between the Android packages and Java archives:

```xml
<mime-type type="application/vnd.android.package-archive">
  <sub-class-of type="application/java-archive"/>
  <glob pattern="*.apk"/>
</mime-type>
```

## Android package metadata

To ensure long-term access, it is vital that an archived app installer is accompanied by [preservation metadata](https://en.wikipedia.org/wiki/Preservation_metadata) about the technical environment that is needed to render it. At the very minimum this would include details about the required Android version(s), hardware features, and shared software libraries. This information (and much more) is stored in an Android Package's [App Manifest](https://developer.android.com/guide/topics/manifest/manifest-intro). The App Manifest is stored in a [binary XML](https://en.wikipedia.org/wiki/Binary_XML) format for which [no publicly available documentation exists](https://reverseengineering.stackexchange.com/questions/21806/where-is-android-binary-xml-format-documented). This makes reading it somewhat challenging, although [various software solutions for decoding the App Manifest exist](https://stackoverflow.com/q/4191762/1209004).

## Extraction of App Manifest

[Apkanalyzer](https://developer.android.com/studio/command-line/apkanalyzer.html), which is part of [Android Studio](https://developer.android.com/studio/), is the tool that is officially supported by Google. However, running apkanalyzer only resulted in a sequence of Java exceptions for me[^4]. Besides, it's not entirely clear if the [terms and conditions](https://developer.android.com/studio/terms.html) of Android Studio permit is use in an archival workflow[^5]. I eventually found [Androguard](https://github.com/androguard/androguard), which is a Python-based tool that is primarily aimed at reverse-engineering Android apps. Using the command below, it will extract and decode an APK's app manifest, resulting in a human-readable XML file:

```bash
androguard axml com.Triplee.TripleeSocial.apk -o arize-android.xml
```

## Interesting App Manifest elements 

The decoded app manifest from the above example can be found in full [here](https://github.com/KBNLresearch/mobile-apps/blob/main/sample-files/arize-androidManifest.xml). A detailed discussion of the app manifest is beyond the scope of this post, but it's worth highlighting a few elements that are particularly interesting:

- The [uses-sdk](https://developer.android.com/guide/topics/manifest/uses-sdk-element) element contains information about the app's compatibility with one or more Android versions:
  ```xml
  <uses-sdk android:minSdkVersion="24" android:targetSdkVersion="29"/>
  ```
  Here, the (confusingly named) `minSdkVersion` and `targetSdkVersion` attributes define the minimum and target API levels of the app, respectively. In [the table here](https://developer.android.com/guide/topics/manifest/uses-sdk-element#ApiLevels) we see that API level 24 (the minimum level) corresponds to Android 7.0, and level 29 (the target level) to Android 10.

- The [uses-feature](https://developer.android.com/guide/topics/manifest/uses-feature-element) element is used to declare hardware or software features that are used by the app:
  ```xml
  <uses-feature android:name="android.hardware.camera" android:required="true"/>
  ```
  In the above example, it informs us that the app needs a camera.

- The [uses-library](https://developer.android.com/guide/topics/manifest/uses-library-element) element tells us about any shared libraries that the app depends on. 

The above information largely defines the (emulated) technical environment that is required to run the app. Even though I've only skimmed the surface of the App Manifest here, its importance as a source for deriving technical and preservation metadata about an Android app should be clear.

## Downloading iOS packages

Apple iOS apps are distributed through the [Apple App Store](https://www.apple.com/app-store/). Installer packages are published in the [iOS App Store Package (IPA)](https://en.wikipedia.org/wiki/.ipa) format. A good overview of the format can be found [here](https://web.archive.org/web/20200714200020/https://blog.razb.me/pulling-apart-an-ios-app/). Like Google's Play Store, Apple doesn't allow you to download the packages on anything but an Apple device. Unlike the Android situation, there don't appear to be any tools that are able to get around this limitation, and this seriously limits the possibilities to incorporate downloading iOS packages as part of a preservation workflow. It might be possible to work around these limitations to some degree by installing the app on either a physical or virtual[^10] iOS device, and then transfer the app to another machine. The open-source [libimobiledevice](https://libimobiledevice.org/) library appears to be capable of file transfers between iOS and other platforms. However, according to various online sources iOS doesn't actually keep the original IPA files after installation[^8]. I'm unable to confirm this, as I don't currently have an iOS device available for further testing.

## iOS package identification

As I was unable to obtain any IPA installers from the Apple App Store, I downloaded some random IPA files from an [iOS jailbreaking](https://en.wikipedia.org/wiki/IOS_jailbreaking) website[^9], and ran them through Siegfried, Unix File and Apache Tika. The table below shows the results:

|Tool|Version|ID|
|:--|:--|:--|
|[Siegfried](https://www.itforarchivists.com/siegfried/)|1.9.1; DROID Signature File V97[^7]|x-fmt/263 (ZIP Format)|
|[Unix File](http://darwinsys.com/file/)|5.32|application/zip|
|[Apache Tika](http://tika.apache.org/)|1.23|application/x-itunes-ipa|

These results are similar to the situation for Android packages. Only Apache Tika was able to identify these files as IPA packages. Both Siegfried and File could only detect the container format. An inspection of PRONOM confirmed that it doesn't include the IPA format yet. Also, Tika's specific result for this format is again [only based on a file extension pattern](https://github.com/apache/tika/blob/master/tika-core/src/main/resources/org/apache/tika/mime/tika-mimetypes.xml#L3819):

```xml
<mime-type type="application/x-itunes-ipa">
  <sub-class-of type="application/zip"/>
  <_comment>Apple iOS IPA AppStore file</_comment>
  <glob pattern="*.ipa"/>
</mime-type>
```

## iOS package metadata

For iOS apps, the [information property list file](https://developer.apple.com/documentation/bundleresources/information_property_list) (Info.plist) in the root of the bundle directory[^11] contains various metadata, including information about the required technical environment. Confusingly, [Apple property lists](https://en.wikipedia.org/wiki/Property_list) can be implemented in both XML and binary formats. For both test files I analyzed, the format was XML, but I'm not entirely sure if this is always the case. The Apple developer's site provides [a brief explanation of the format](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/PropertyLists/UnderstandXMLPlist/UnderstandXMLPlist.html#//apple_ref/doc/uid/10000048i-CH6-SW1), and I've uploaded [an example file here](https://github.com/KBNLresearch/mobile-apps/blob/main/sample-files/Info.plist). Even though the format only defines simple key-value pairs, Apple's implementation is unusual to say the least. Instead of using the XML hierarchy, values are defined by their position relative to the "key" elements. The following fragment illustrates this:

```xml
<key>MinimumOSVersion</key>
<string>7.0</string>
<key>UIDeviceFamily</key>
<array>
  <integer>1</integer>
  <integer>2</integer>
</array>
```

In this example, the value of *MinimumOSVersion* is defined by the *string* element that directly follows the *key* element; likewise, the value of *UIDeviceFamily* is defined by the *array* element. This unusual layout means that simple parsing of these files with an XML library is not enough to interpret them in a meaningful way!

## Extraction of information property list

I was unable to find any tools that directly extract and process the information property list (similar to what [Androguard](https://github.com/androguard/androguard) does for Android packages)[^12]. However, Python has a built-in [plistlib](https://docs.python.org/3/library/plistlib.html) module that is able to read and write property lists in both binary and XML format. I did some tests with it, and at first sight it appears to work well: reading the XML property lists for each of my test apps resulted in a Python dictionary that accurately represented the key-value pairs[^13]. Using this module, it would be fairly straightforward to write a tool that extracts the property list items directly out of an IPA file, and transform them into a more manageable format. 

## Interesting property list elements

As with the Android App Manifest before, I won't go into a detailed discussion of all the items inside the information property list, but the following ones caught my attention:

- The [UIRequiredDeviceCapabilities](https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/DeviceCompatibilityMatrix/DeviceCompatibilityMatrix.html) key declares "the hardware or specific capabilities" that an app needs in order to run. This [includes](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW3), among other things, access to networking features, a camera or a microphone.
- The [MinimumOSVersion](https://developer.apple.com/documentation/bundleresources/information_property_list/minimumosversion) key defines the minimum operating system version required for the app to run.

Both are directly relevant for emulation purposes.

## Summary of test results

As this is quite a lengthy post, here's a brief summary of the main results of the above tests:

- Downloading Android APK packages from the Google Play store is possible, but it does require unofficial third-party tools like [gplaycli](https://github.com/matlink/gplaycli). However, such tools may stop functioning if Google applies changes to its Play Store API. This has already happened previously, leading to various unmaintained tools that no longer work. 
- Access to the Apple App Store appears to be completely restricted from non-Apple devices. Since installing an app on iOS reportedly gets rid of the IPA container, workarounds that use a native iOS device as an intermediate medium most likely won't be usable for preservation workflows. 
- Out of the three file format identification tools tested, only Apache Tika was able to correctly identify both APK and IPA files. However, Tika's specific results are solely based on file extension patterns. At the time of writing PRONOM doesn't cover these formats at all, and any tools that use its database (Siegfried, but also DROID and FIDO) only identify them at the higher container levels (ZIP, JAR). This could be easily remedied by developing PRONOM signatures for both formats[^16].
- Both the APK and IPA formats contain package-level metadata about an app's technical dependencies, such as the minimal OS version and required hardware.
- For the APK format a software tool that extracts this information is readily available. For the IPA format no such tool exists, but it could be developed with a limited amount of effort.
- Based on the cursory look presented here, these package-level metadata appear to be adequate for establishing the emulated environment needed to run an app, but they do not expose any dependencies on remotely hosted content[^15].

## Value of documentation

Pennock, May and Day propose the use of "alternative solutions such as recording or documentation" in case the end access solution (e.g. the emulator) does not provide a sufficiently 'authentic' experience. In addition to this, Trevor Owens makes an argument for storing screenshots and video recordings of software in his book "The theory and craft of digital preservation"[^18]:

> \[M\]oving to an approach to virtualize or emulate old systems on new hardware will inevitably be a complex process and having even a reference image of what it looked like at a particular moment in time would likely be valuable as a way to evaluate the extent to which it is being authentically rendered. Here a general principle emerges. Documentation (like the screenshot) can be useful as both a means to preserve significance and also as a means to create reference material to triangulate an object’s or work’s significance.

For both the ARize and Immer apps from my previous blog post, Youtube channels exist with videos that demonstrate how these apps work[^19]. I don't know how common this situation is for other apps, but such videos could serve as a reference for (future) emulation efforts, and it might be worthwhile to collect them as part of the acquisition process, and store them alongside the app packages.

## Final thoughts

This post is only a first (and at this stage incomplete) attempt at piecing together candidate components for acquisition and (pre-)ingest workflows for Android and iOS apps. Please feel free to use the comment section in case you have any corrections or additions.

## Further resources

- [Considerations on the Acquisition and Preservation of Mobile eBook Apps](https://zenodo.org/record/3460450)
- [gplaycli](https://github.com/matlink/gplaycli)
- [Androguard](https://github.com/androguard/androguard)
- [Pulling apart an iOS App](https://web.archive.org/web/20200714200020/https://blog.razb.me/pulling-apart-an-ios-app/)
- [Android App Manifest documentation](https://developer.android.com/guide/topics/manifest/manifest-intro)
- [Information Property List Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html)

[^1]: See my [previous post on Android emulation options]({{ BASE_PATH }}/2021/02/09/four-android-emulators-two-apps).

[^2]: Note that the current (3.29) version of the tool has a small bug that results in a warning, which is [documented here](https://github.com/matlink/gplaycli/issues/272). Other than that it does work as expected.

[^3]: I used the [cmp tool](https://linux.die.net/man/1/cmp) for this, with the command `cmp base.apk com.Triplee.TripleeSocial.apk`.

[^4]: Tried with Android Studio 4.1.2, running under Linux Mint 19.3.

[^5]: See also the "Android Emulator for long-term access" section in my [previous blog post]({{ BASE_PATH }}/2021/02/09/four-android-emulators-two-apps).

[^6]: PRONOM version at the time of writing: DROID_SignatureFile_V97.xml, 1st October 2020.

[^7]: DROID Container Signature File 20201001.xml

[^8]: See e.g. [here](https://stackoverflow.com/a/29743193/1209004), [here](https://www.reddit.com/r/jailbreak/comments/4dhbtb/question_ipa_location_in_ios_9/) and [here](https://medium.com/@lucideus/extracting-the-ipa-file-and-local-data-storage-of-an-ios-application-be637745624d).

[^9]: I used the [ioninja.io](https://iosninja.io/ipa-library) site. I have no idea about the site's legal status or the safety of the downloads on offer, so proceed with caution! I only used the downloaded IPAs for some simple technical tests without installing them.

[^10]: For example using a service like [Corellium](https://corellium.com/).

[^11]: This is typically the `Payload/Application.app` folder (where "Application" is replaced with the app's name).

[^12]: There is [Ipa-metadata](https://github.com/matiassingers/ipa-metadata), which is a tool for extracting "metadata and provisioning info about an .ipa file". Although I was able to install it, running it on any of my test files would just return a "Callback must be a function" error, and nothing else.

[^13]: The demo script that I wrote for my tests is [available here](https://github.com/KBNLresearch/mobile-apps/blob/main/scripts/readplist.py).

[^14]: I'm well aware that the possibilities for emulating iOS-based devices are still very limited (for both technical and legal reasons), but that may be the subject of another post.

[^15]: This needs further confirmation from a more in-depth look at the available documentation.

[^16]: I'd be happy to have a go at this myself.

[^17]: An Executable Past: The Case for a National Software Registry. In: Preserving.exe: Toward a National Strategy for Preserving Software. Library of Congress, 2013. Link: <https://www.digitalpreservation.gov/multimedia/documents/PreservingEXE_report_final101813.pdf>.

[^18]: The theory and craft of digital preservation. Johns Hopkins University Press, 2018. Link: <http://www.trevorowens.org/theory-and-craft-of-digital-preservation/>

[^19]: ARize app Youtube channel: <https://www.youtube.com/channel/UCPEDDkRVC7jegjC02-7VsUw/videos>; Immer app Youtube channel: <https://www.youtube.com/channel/UCnrnDrJ5MXccQJxaicEi7cA>.