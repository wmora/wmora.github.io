--- 
title: Change Screen Brightness From Terminal (Ubuntu 10.04)
date: "2010-05-12T12:30:00.001-03:00"
author: William Mora
tags: 
- screen brightness
- ubuntu 10.04
- setpci
redirect_from: 
- /2010/05/change-screen-brightness-from-terminal.html
---

If you want to change the screen brightness in Ubuntu (I can’t change it using the keyboard shortcuts or the Ubuntu Power Management menu), open a terminal and execute the following:

```bash
sudo setpci -s 00:02.0 F4.B=xx
```

Where xx is the desired brightness in hex ranging from 0 (brightest) to FF (no brightness at all). I usually change it to CA when working on battery.

Cheers!