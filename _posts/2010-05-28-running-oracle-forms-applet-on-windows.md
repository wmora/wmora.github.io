--- 
title: Running an Oracle Forms Applet on Windows 7
date: "2010-05-28T19:26:00.000-03:00"
author: William Mora
tags: 
- windows 7
- Oracle 10g
- Java
- Internet Explorer
- Forms
- Reports
- NriCmd
- graphics card
- Oracle
- jdk
redirect_from: 
- /2010/05/running-oracle-forms-applet-on-windows.html
---

I was trying to run my application built in Oracle Forms and Reports on a new  laptop with Windows 7. However, when running the Java forms applet, the applet’s  graphics stayed frozen and I was not able to see my application properly. 

This problem is caused due to the graphics card running on the computer  running Windows 7. To run certain applications, Windows 7 must change its theme  back to Windows 7 Basic (I believe it was the same issue with Vista). So, basically, the graphics card is too much for the Java applet to handle.

<!--more-->
A quick, simple solution is to modify the java.exe found in  `%ORACLE_HOME%\jdk\bin` so it runs with only 256 colors (8 bits). The problem with  this solution is that the computer looks really bad since it is only running on  256 colors (that’s what the original Nintendo used!). I needed to run the applet in order to show my application to future clients, so I needed the application to run  on 32 bits. After looking at several settings that may cause the issue, I realized that if I quickly refreshed the screen once the applet was initialized (e.g. change the monitor display settings so it runs on 16 bits and then cancel the changes), the applet would run without any problems. So, I needed to somehow refresh the monitor once the applet started in order to make my application run on 32 bits without the image staying frozen.

Although it was not the most elegant solution (I was on a deadline!), I decided to include a function in the html that launches my application so it runs a batch file that quickly refreshes the screen. This is how I did it:

1) Downloaded the command-line tool NirCmd, which allows you to change the monitor settings without the user interface. You can download the tool [here](http://www.nirsoft.net/utils/nircmd.html).

2) Unzipped the nircmd executables to c:\Windows

3) Created the .bat (I saved it as screenRefresher.bat) file to be launched with the application with the lines:

```bash
echo off
nircmd setdisplay 1360 768 16
nircmd setdisplay 1360 768 32
```

Of course, the 1360 768 should correspond to your particular screen settings. What the batch file is basically doing is quickly changing the screen display colors to 16 bits and then to 32 bits. This is just a dummy operation so the screen refreshes at runtime.

4) Finally, edited the html to run the batch file when loading. This is the section I modified:

```html
<html>
<head>
<script type="text/javascript">
function refreshMonitor()
{
WshShell = new ActiveXObject("WScript.Shell");
WshShell.Run ('file://C:/appLocation/html/screenRefresher.bat',1,true);
}
</script>
<title>AppName - Welcome</title>
</head>
<body onload="refreshMonitor();">
<!--Body content-->
<body>
<html>
```

5) Changed the security settings of Internet Explorer so it allows running Active X objects

6) And done! Now I can start the instance, run the application and while the applet is loading the batch file will execute simultaneously. The applet now runs on 32 bits without any problems!

NOTE: This has only been tested with Internet Explorer and this is not a solution suggested in any way by Oracle. It is just a workaround I implemented for my application to run on Windows 7