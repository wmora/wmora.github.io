---
title: Martian Run! Open-Source Game with LibGDX
date: "2015-01-02"
author: William Mora
tags: 
- gamedev
- indiedev
- video game development
- libgdx
- box2d
- scene2d
- google play
- martian run
- google play game services
- admob
---

A few months ago I wrote a [tutorial for a 2D video game with libGDX, using box2d and scene2d](http://williammora.com/a-running-game-with-libgdx-part-1/). Many of the things I explained in the tutorial have changed a bit to improve performance and make the final game more of a complete product since one of my goals since day 1 of that project was to publish it in [Google Play](https://play.google.com/store/apps/details?id=com.gamestudio24.cityescape.android). So far I've received mostly positive feedback for the tutorial (thanks!). However, there were a few things I did not cover in the tutorial that I'm sure people would be interested in: I did not explain how to add the menu layer to the game nor how to integrate the game with Google Play Game Services or AdMob, both which were included in the final version.

<!--more-->
Although it may be trivial for many, I decided it was best to fully open source the game for everyone to see how the final version, the one published on Google Play, is implemented. This would include a few tweaks with the assets to improve performance, implementation of the game difficulty change, and overall management of achievements with Google Play Game Services for Android. This past week I worked hard to make this happen and the [GitHub project](https://github.com/wmora/martianrun) has been updated. You can clone it, add a few string resources for the Android module and run the game just as it is published.

The documentation might be a bit incomplete, but the code shouldn't be too hard to follow if you [quickly go over the tutorial](http://williammora.com/a-running-game-with-libgdx-part-1/).

And in case you are interested, you can download [Martian Run! here](https://play.google.com/store/apps/details?id=com.gamestudio24.cityescape.android).

Hope the project helps you if you are looking to get started with libGDX and video game development in general.

Any feedback is welcome in the comments section. Happy coding!