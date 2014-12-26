--- 
title: OpenFeed - Android L Twitter Client with RecyclerView and CardView Widgets
date: "2014-07-14T23:09:00.002-03:00"
author: William Mora
tags: 
- mobile
- Google Play
- software development
- RecyclerView
- github
- CardView
- Android
- Twitter
- OpenFeed
- nexus 5
- Material theme
permalink: /2014/07/openfeed-android-l-recyclerview-cardview.html
---

With Google's announcement of the new version of [Android (codename L)](http://www.androidauthority.com/android-l-release-official-397212/) last month. I installed the [preview SDK](https://developer.android.com/preview/setup-sdk.html) on my Nexus 5 and all of my apps worked pretty good except for Twitter which would not even open.

I decided to create an app to make use of the new [`RecyclerView` and `CardView`](https://developer.android.com/preview/material/ui-widgets.html) widgets along with the new [Material theme](https://developer.android.com/preview/material/index.html). Since Twitter didn't work on my phone, I started my own client: [OpenFeed](https://github.com/wmora/openfeed). It's a work in progress and its functionality will be limited since Twitter's API does not provide everything I need (i.e. I can't load a conversation thread).

Feel free to [browse through the code](https://github.com/wmora/openfeed), download the project and run it on your emulator or device. Let me know what you think in the comments section.