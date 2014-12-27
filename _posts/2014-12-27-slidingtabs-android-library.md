---
title: SidingTabs Android Library
date: 2014-12-27
author: William Mora
tags:
- mobile
- Java
- Android
- Android Library
- AndroidDev
- Google
- SlidingTabs
- software development
- github
- Android Studio
assets_url: /assets/slidingtabs-android-library
screenshot_width: 180
---

This library project is on [GitHub](https://github.com/nispok/slidingtabs).

With the release of [Android Studio 1.0](http://android-developers.blogspot.com/2014/12/android-studio-10.html), one of the things to notice is that there's an option to import an Android code sample. All these samples are available on [GitHub/googlesamples](https://github.com/googlesamples).

<img src="{{ page.assets_url }}/as-import-sample.png" width=400 />

<!--more-->
While having those projects available is good, I think it would be even better to make all these samples available as libraries for everyone to use and contribute in a single codebase. For example, if you look at the code of both the SlidingTabs sample projects, [SlidingTabsBasic](https://github.com/googlesamples/android-SlidingTabsBasic) and [SlidingTabsColor](https://github.com/googlesamples/android-SlidingTabsColors/), you'll notice that both of them have the same exact code under a `com.example.android.common` package.

I'm working on a basic [open-source Imgur client](https://github.com/nispok/imgurdroid) for Android where I needed to include the Sliding Tabs from the samples, so I decided to make the job a little easier for everyone and publish the views from the `common` package as a library.

The library is called [Sliding Tabs](https://github.com/nispok/slidingtabs) and it consists of the `SlidingTabLayout` and `SlidingTabStrip` classes, with minor modifications to work from minSDK 8. All contributions are welcome; open a [PR](https://github.com/nispok/slidingtabs/pulls) or refer to the [issues](https://github.com/nispok/slidingtabs/issues) section.

You can see an example of this library in [Imgurdroid](https://github.com/nispok/imgurdroid). And although not using this library, the same code is used in [Google I/O Android App](https://github.com/google/iosched) and, of course, [Android SlidingTabsBasic Sample](https://github.com/googlesamples/android-SlidingTabsBasic/) and [Android SlidingTabsColors Sample](https://github.com/googlesamples/android-SlidingTabsColors/).

Here are a few screenshots of the library in action:

<img src="{{ page.assets_url }}/screenshot_1.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_2.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_3.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_4.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_5.png" width={{ page.screenshot_width }}>

