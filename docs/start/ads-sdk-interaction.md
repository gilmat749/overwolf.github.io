---
id: ads-sdk-interaction
title: Managing Ads
sidebar_label: Managing Ads
---

## The Ads object

Calling a new `OwAd()` will return a javascript object which you can use to control the displayed ad.

> To prevent ads being called but not shown, use **removeAd()** on minimizing/hiding and **refreshAd()** on restoring.

### Functions

This object has the following functions you can call on-demand:

| Function        | Description                                                                                                            |
| :-------------: |------------------------------------------------------------------------------------------------------------------------|
| removeAd        | Removes current ad from the DOM                                                                                        |
| refreshAd       | Refreshes ad and loads a new one <br> Note that this will cause the ad to scroll into view if not already visible      |

### Notes

On minimizing/hiding an OW window with Ads, no need to delete or destroy the **owAd** object.  
Instead, call the **removeAd()** method.  
When the window is restored, call the same **owAd** instance’s **refreshAd()** method.

### Events

You can use Overwolf ad objects created by calling `new OwAd()`) to listen to events related to the displayed ad. Registering an event can be done by calling the function **addEventListener** on the ad object. Like other libraries, the first parameter sent to **addEventListener** is the name of the event, and the second is a handler function for the event.

The following events are supported:

| **Event Name**      | **Fired When**           
| -------------       | -------------
| player_loaded	      | Ad video player successfully loaded on page*
| display_ad_loaded   | Display ad was served instead of a video ad    
| play                | Ad started playing
|impression	          | Video “impression” – Depends on the advertiser, the impression event gets triggered after 0-6 seconds *
|complete             | Video ad played fully until completed *
|error                | Error occurred while trying to load ad *

\* Only relevant for video ads

### Multiple ad placements 
If you have more than one ad placement on your app, you can easily do so by creating multiple instances of `OwAd`. Just make sure you pass a different container element for each instance.


## Checking the window state change

To check when the window is minimized or restored, you can use the [overwolf.windows.onStateChanged](../api/overwolf-windows#onstatechanged) event.

Note that the *onStateChanged* event is being fired for all the declared windows listening to this event (background, in-game, desktop, etc.). Make sure to test the window name/id arguments that are passed to the event to see if the window with the Ad Is the window that triggered the state change. 

Otherwise, you might end up with unnecessary wrong calls to the **refreshAd()** and **removeAd()**.

### In-game windows with Ads

As we mentioned above, when you change a window state (minimize, hide, restore), the **onStateChanged** event is fired. But that is not the case if you minimize the GAME WINDOW itself, Alt+Tab from it, use Win+D to minimize all your open apps, or even click outside of the windowed game.

> When your game window loses focus, the in-game window state inside it will not change, and the **onStateChanged** event will not be fired.

That means if your in-game window contains an Ad, and the containing GAME WINDOW is minimized/restored, you will not be able to know when to call **removeAd()** and **refreshAd()**. 

In order to handle that scenario, you can listen to [onGameInfoUpdated](../api/overwolf-games#ongameinfoupdated) event and stop the ad if **gameInfo.isInFocus** is false, and refresh it when **gameInfo.isInFocus** changes to *true*.  (note that LoL might have different behavior, so double-check it).

## Ads Type Definitions

Type definitions for the `OwAd` class and the various interfaces it uses can be found at our [types](https://github.com/overwolf/types/blob/master/owads.d.ts) github.
Import this file to your app to get autocompletion and type inference with the `OwAd`.


#### Sample code for handling ads with minimized/restored window

```
// define the event handler
function onWindowStateChanged(state) {
  console.log(`Window state changed: ${JSON.stringify(state)}`);

  if(state) {
    // when state changes to minimized, call removeAd()
    if (state.window_state === "minimized") {
      owAdInstance.removeAd();
    }
    // when state changes from minimized to normal, call refreshAd()
    else if(state.window_previous_state === "minimized" && state.window_state === "normal"){
      owAdInstance.refreshAd();
    }
  }
}

// call the overwolf api
overwolf.windows.onStateChanged.removeListener(onWindowStateChanged);
overwolf.windows.onStateChanged.addListener(onWindowStateChanged);
```
