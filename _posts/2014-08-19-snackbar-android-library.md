--- 
title: Snackbar Android Library
date: "2014-08-19T22:00:00.000-03:00"
author: William Mora
tags: 
- mobile
- Java
- android l
- snackbar
- gradle
- software development
- github
- Android
- material design
permalink: /2014/08/snackbar-android-library.html
---

This library project is on [GitHub](https://github.com/nispok/snackbar) along with an example app.

Google's [Material Design documentation](http://www.google.com/design/spec/material-design/introduction.html) mentions a new component used for quick feedback called [Snackbars](http://www.google.com/design/spec/components/snackbars-and-toasts.html). The document describes them as follows:
> Snackbars provide lightweight feedback about an operation in a small popup at the base of the screen on mobile and at the lower left on desktop. They are above all over elements on screen, including the floating action button. 
> 
> They automatically disappear after a timeout or after user interaction elsewhere on the screen, whichever comes first. Snackbars can be swiped off screen. They do not block input on the screens they appear on and cannot receive input focus. Show only one snackbar on screen at a time.Here are a couple of screenshots of how they should look:

[![](http://3.bp.blogspot.com/-SSic6M1r-Zk/U_PGY8ekgYI/AAAAAAAAGxw/oKA_Hb2jt3E/s320/components-toasts-specs-spec_toast_03_1_large_mdpi.png)](http://3.bp.blogspot.com/-SSic6M1r-Zk/U_PGY8ekgYI/AAAAAAAAGxw/oKA_Hb2jt3E/s1600/components-toasts-specs-spec_toast_03_1_large_mdpi.png)&nbsp; &nbsp;[![](http://3.bp.blogspot.com/-Bt2HjtFWpFc/U_PGhBOnoZI/AAAAAAAAGx4/eCzIRXEUPIs/s320/components-toasts-specs-spec_toast_03_2_large_mdpi.png)](http://3.bp.blogspot.com/-Bt2HjtFWpFc/U_PGhBOnoZI/AAAAAAAAGx4/eCzIRXEUPIs/s1600/components-toasts-specs-spec_toast_03_2_large_mdpi.png)

Since there's no native widget for this (at least not yet), I created a small Android library to quickly include this component on top of any activity. It only supports phones; it would still pop up at the base of the screen on tablets/desktops. 

Using the `Snackbar` class is easy, this is how you would display it on an `Activity`: 

```java
Snackbar.with(getApplicationContext()) // context
    .text("Single-line snackbar") // text to display
    .show(this); // activity where it is displayed
```

If you want an action button to be displayed, just assign a label and an `ActionClickListener`: 

```java
Snackbar.with(getApplicationContext()) // context
    .text("Item deleted") // text to display
    .actionLabel("Undo") // action button label
    .actionListener(new Snackbar.ActionClickListener() {
        @Override
        public void onActionClicked() {
            Log.d(TAG, "Undoing something");
        }
     }) // action button's ActionClickListener
     .show(this); // activity where it is displayed
```

There are two `Snackbar` types: single-line (default) and multi-line (2 lines max). You can also set the duration of the `Snackbar` similar to a [`Toast`](http://developer.android.com/reference/android/widget/Toast.html). Animation disabling is also possible. 
<pre>Snackbar.with(getApplicationContext()) // context
    .type(Snackbar.SnackbarType.MULTI_LINE) // Set is as a multi-line snackbar
    .text("This is a multi-line snackbar. Keep in mind that snackbars are " +
        "meant for VERY short messages") // text to be displayed
    .duration(Snackbar.SnackbarDuration.LENGTH_SHORT) // make it shorter
    .animation(false) // don't animate it
    .show(this); // where it is displayed
</pre>If you need to know when the `Snackbar` is shown or dismissed, assign a `EventListener` to it. This is useful if you need to move other objects while the `Snackbar` is displayed. For instance, you can move a Floating Action Button up while the `Snackbar` is on screen: 
<pre>
Snackbar.with(getApplicationContext()) // context
    .text("This will do something when dismissed") // text to display
    .eventListener(new Snackbar.EventListener() {
        @Override
        public void onShow(int height) {
           myFloatingActionButton.moveUp(height);
        }        
        @Override
        public void onDismiss(int height) {
           myFloatingActionButton.moveDown(height);
        }
    }) // Snackbar's DismissListener
    .show(this); // activity where it is displayed
</pre>Finally, you can change the `Snackbar`'s colors. 
<pre>Snackbar.with(getApplicationContext()) // context
    .text("Different colors this time") // text to be displayed
    .textColor(Color.GREEN) // change the text color
    .color(Color.BLUE) // change the background color
    .actionLabel("Action") // action button label
    .actionColor(Color.RED) // action button label color
    .actionListener(new Snackbar.ActionClickListener() {
        @Override
        public void onActionClicked() {
            Log.d(TAG, "Doing something");
        }
     }) // action button's ActionClickListener    
    .show(this); // activity where it is displayed
</pre>The documentation also states that they can be swiped off the screen. I used [Roman Nurik's SwipeToDismiss sample code](https://github.com/romannurik/android-swipetodismiss) to implement this.

The library project is on [GitHub](https://github.com/nispok/snackbar) along with an example app. If you would like to add features to it or report any bugs, refer to the [issues](https://github.com/wmora/snackbar/issues) section.
Here are a few screenshots of the library in action:

[![](http://1.bp.blogspot.com/-5OkYxr59g10/U_Ps-4vV3XI/AAAAAAAAGyQ/RPX1BAd9eHU/s320/Screenshot_2014-08-19-19-14-07.png)](http://1.bp.blogspot.com/-5OkYxr59g10/U_Ps-4vV3XI/AAAAAAAAGyQ/RPX1BAd9eHU/s1600/Screenshot_2014-08-19-19-14-07.png)&nbsp;&nbsp;[![](http://3.bp.blogspot.com/-rqMpr9nysSY/U_Ps-zvhgOI/AAAAAAAAGyM/38M0N_j4i6U/s320/Screenshot_2014-08-19-19-14-16.png)](http://3.bp.blogspot.com/-rqMpr9nysSY/U_Ps-zvhgOI/AAAAAAAAGyM/38M0N_j4i6U/s1600/Screenshot_2014-08-19-19-14-16.png)&nbsp;&nbsp;[![](http://2.bp.blogspot.com/-AwjqlrBiAfs/U_Ps-2L_uqI/AAAAAAAAGyI/YJRtC21ocp8/s320/Screenshot_2014-08-19-19-14-24.png)](http://2.bp.blogspot.com/-AwjqlrBiAfs/U_Ps-2L_uqI/AAAAAAAAGyI/YJRtC21ocp8/s1600/Screenshot_2014-08-19-19-14-24.png)
[![](http://2.bp.blogspot.com/-W5S5LB61fOM/U_PtADkAmWI/AAAAAAAAGys/xFAb3FbYnls/s320/Screenshot_2014-08-19-19-14-31.png)](http://2.bp.blogspot.com/-W5S5LB61fOM/U_PtADkAmWI/AAAAAAAAGys/xFAb3FbYnls/s1600/Screenshot_2014-08-19-19-14-31.png)&nbsp;&nbsp;[![](http://2.bp.blogspot.com/-mpoO1PpIZfU/U_PtAbT9NdI/AAAAAAAAGyU/xvDYuIC1nsM/s320/Screenshot_2014-08-19-19-14-43.png)](http://2.bp.blogspot.com/-mpoO1PpIZfU/U_PtAbT9NdI/AAAAAAAAGyU/xvDYuIC1nsM/s1600/Screenshot_2014-08-19-19-14-43.png)&nbsp;&nbsp;[![](http://1.bp.blogspot.com/-6FuxqQH1d3E/U_PtBKyjcsI/AAAAAAAAGyY/kc-qMazyk9c/s320/Screenshot_2014-08-19-19-15-07.png)](http://1.bp.blogspot.com/-6FuxqQH1d3E/U_PtBKyjcsI/AAAAAAAAGyY/kc-qMazyk9c/s1600/Screenshot_2014-08-19-19-15-07.png)
