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
assets_url: /assets/snackbar-android-library
screenshot_width: 180
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
However, I recommend you use the `SnackbarManager` to handle the Snackbars queue:

```java
// Dismisses the Snackbar being shown, if any, and displays the new one
SnackbarManager.show(
    Snackbar.with(myActivity)
    .text("Single-line snackbar"));
```
If you are using `getApplicationContext()` as the `Context` to create the `Snackbar` then you must
specify the target `Activity` when calling the `SnackbarManager`:

```java
// Dismisses the Snackbar being shown, if any, and displays the new one
SnackbarManager.show(
    Snackbar.with(getApplicationContext())
    .text("Single-line snackbar"), myActivity);
```
If you want an action button to be displayed, just assign a label and an `ActionClickListener`:

```java
SnackbarManager.show(
    Snackbar.with(getApplicationContext()) // context
        .text("Item deleted") // text to display
        .actionLabel("Undo") // action button label
        .actionListener(new ActionClickListener() {
            @Override
            public void onActionClicked(Snackbar snackbar) {
                Log.d(TAG, "Undoing something");
            }
        }) // action button's ActionClickListener
     , this); // activity where it is displayed
```
If you need to know when the `Snackbar` is shown or dismissed, assign a `EventListener` to it.
This is useful if you need to move other objects while the `Snackbar` is displayed. For instance,
you can move a Floating Action Button up while the `Snackbar` is on screen:

```java
SnackbarManager.show(
    Snackbar.with(getApplicationContext()) // context
        .text("This will do something when dismissed") // text to display
        .eventListener(new EventListener() {
            @Override
            public void onShow(Snackbar snackbar) {
                myFloatingActionButton.moveUp(snackbar.getHeight());
            }
            @Override
            public void onShown(Snackbar snackbar) {
                Log.i(TAG, String.format("Snackbar shown. Width: %d Height: %d Offset: %d",
                        snackbar.getWidth(), snackbar.getHeight(),
                        snackbar.getOffset()));
            }
            @Override
            public void onDismiss(Snackbar snackbar) {
                myFloatingActionButton.moveDown(snackbar.getHeight());
            }
            @Override
            public void onDismissed(Snackbar snackbar) {
                Log.i(TAG, String.format("Snackbar dismissed. Width: %d Height: %d Offset: %d",
                                    snackbar.getWidth(), snackbar.getHeight(),
                                    snackbar.getOffset()));
            }
        }) // Snackbar's EventListener
    , this); // activity where it is displayed
```
There are two `Snackbar` types: single-line (default) and multi-line (2 lines max). You can also set
the duration of the `Snackbar` similar to a
<a href="http://developer.android.com/reference/android/widget/Toast.html">`Toast`</a>.

The lengths of a Snackbar duration are:
* LENGTH_SHORT: 2s
* LENGTH_LONG: 3.5s
* LENGTH_INDEFINTE: Indefinite; ideal for persistent errors

You could also set a custom duration.

Animation disabling is also possible.

```java
SnackbarManager.show(
    Snackbar.with(getApplicationContext()) // context
        .type(Snackbar.SnackbarType.MULTI_LINE) // Set is as a multi-line snackbar
        .text("This is a multi-line snackbar. Keep in mind that snackbars are " +
            "meant for VERY short messages") // text to be displayed
        .duration(Snackbar.SnackbarDuration.LENGTH_SHORT) // make it shorter
        .animation(false) // don't animate it
    , this); // where it is displayed
```
You can also change the `Snackbar`'s colors.

```java
SnackbarManager.show(
    Snackbar.with(getApplicationContext()) // context
        .text("Different colors this time") // text to be displayed
        .textColor(Color.GREEN) // change the text color
        .color(Color.BLUE) // change the background color
        .actionLabel("Action") // action button label
        .actionColor(Color.RED) // action button label color
        .actionListener(new ActionClickListener() {
            @Override
            public void onActionClicked(Snackbar snackbar) {
                Log.d(TAG, "Doing something");
            }
         }) // action button's ActionClickListener
    , this); // activity where it is displayed
```
Finally, you can attach the `Snackbar` to a AbsListView (ListView, GridView) or a RecyclerView.

```java
SnackbarManager.show(
    Snackbar.with(getApplicationContext()) // context
        .type(Snackbar.SnackbarType.MULTI_LINE) // Set is as a multi-line snackbar
        .text(R.string.message) // text to be displayed
        .duration(Snackbar.SnackbarDuration.LENGTH_LONG)
        .animation(false) // don't animate it
        .attachToAbsListView(listView) // Attach to ListView - attachToRecyclerView() is for RecyclerViews
        , this); // where it is displayed
```
It uses [Roman Nurik's SwipeToDismiss sample code](https://github.com/romannurik/android-swipetodismiss)
to implement the swipe-to-dismiss functionality. This is enabled by default. You can disable this if
you don't want this functionality:

**NOTE:** This has no effect on apps running on APIs < 11; swiping will always be disabled in those cases

```java
SnackbarManager.show(
    Snackbar.with(SnackbarSampleActivity.this) // context
        .text("Can't swipe this") // text to be displayed
        .swipeToDismiss(false) // disable swipe-to-dismiss functionality
    , this); // activity where it is displayed
```

The library project is on [GitHub](https://github.com/nispok/snackbar) along with an example app. If you would like to add features to it or report any bugs, refer to the [issues](https://github.com/nispok/snackbar/issues) section.
Here are a few screenshots of the library in action:

<img src="{{ page.assets_url }}/screenshot_1.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_2.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_3.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_4.png" width={{ page.screenshot_width }}>
<img src="{{ page.assets_url }}/screenshot_5.png" width={{ page.screenshot_width }}>