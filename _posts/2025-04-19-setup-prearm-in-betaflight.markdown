---
layout: post
title: Setup pre-arm in betaflight
date: 2025-04-21 06:45:00 +01:00
categories: [tools, drone, betaflight, safety]
---

I have recently taken up <abbr title="First Person View">FPV</abbr> drone flying.
It is both difficult and satisfying.
[/r/fpv](https://reddit.com/r/fpv) has been a helpful community in getting started and finding great gear that works well together.
One thing I came across was a recommendation to enable pre-arm on your drones, but it took some research to figure it all out as a beginner.

Setting up pre-arm means you need both hands engaged to arm your drone before flight. This can protect you from accidental arming a drone and getting cut on the propellers. Common stories include locating a downed drone and picking it up, the radio dangling on its leash and the arm button is bumped. Graphic pain follows.

## Steps to setup pre-arm

The setup is easy and straight forward once you know how.
It is directly supported in betaflight, so it will merely take a few minutes to configure and no advanced steps necessary.

Start out by considering what button on your radio you want to use for pre-arming.
I picked a momentary switch (one that outomatically switches to OFF after clicking it) on the opposite side of the radio from the ARM toggle, just to make sure I need to use both hands to arm the drone.

  1. Using Chrome on your computer, open the betaflight web version at [app.betaflight.com](https://app.betaflight.com). This is easier than installing betaflight natively on your machine. As of the time of writing, only Chrome supports communicating with USB peripherals.
  1. Connect to your drone with USB or bluetooth.
  1. Click `Modes` menu on the left.
  1. Power on your Radio and connect it to the drone (your remote controller).
  1. Find "PREARM" in the list of modes in betaflight.
  1. Click "Add Range", pick "AUTO" as the input in the dropdown.
  1. Toggle the switch on the radio you want to serve as your pre-arm switch. In this setup I use the momentary switch `SF` on the RadioMasterBoxer. The "AUTO" will now automatically switch to the actual button identity. Turns out this is `AUX6` for this particular radio.
    ![Screenshot of betaflight mode for PREARM, without the button pressed]({{ site.url }}/assets/images/posts/2025-04-21-prearm/button-off.png)
    See what the on/off outputs of your button are, and adjust the range to match by moving the sliders. I only want "pre-arm" to be true when the button is held.
    ![Screenshot of betaflight mode for PREARM, with the button pressed and the ranges adjusted]({{ site.url }}/assets/images/posts/2025-04-21-prearm/button-on.png)

Once you think everything is setup right, click your chosen botton on/off a few times, and see if PREARM lights up when you expect it to. Click "Save" and you're done!

## Pre-arming before flying

* Hold the `SF` <abbr title="Simply means it's spring-loaded and automatically switches off when you release it">momentary</abbr> button.
* Then arm the drone by switching the default `SA` switch `ON`.
* Drone is now armed and ready to fly.
* Disarming: switch `SA` to `OFF` and the drone immediately switches off.

## Other options

The above is a setting within the specific drone. If you have multiple drones you want to behave the same, you could also configure a logical pre-arm sequence in your Radio.
