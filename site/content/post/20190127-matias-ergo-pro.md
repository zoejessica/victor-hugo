---
date: "2019-01-27"
draft: false
title: "Matias Ergo Pro keyboard: long term review"
slug: matias-ergo-pro

categories:
- Keyboard
tags:
- review
- keyboard
description:Matias Ergo Pro keyboard: long term review
---

## Why?
My wrist tendons gave up a while ago, but recently I’ve had problems with my arms too – typing on a traditional straight keyboard positioned in front of me pulled my arms forward, shoulder blades out and up, which in turn impinged on the tendon at the top of the bicep. Also, the hunching made my neck crazy tight. Ow.

Separating a keyboard into two halves means that each half can be closer to the body, so elbows sit at 90º at your sides, forearms are angled slightly outwards, and shoulder blades are back and down where they’re meant to be. It’s helped a ton, even having a real effect on my posture, and I always notice a painful difference if I go back to mindlessly stretching out to a laptop keyboard for any length of time.

I’ve been using the [Matias Ergo Pro](https://matias.ca/ergopro/pc/) for a couple of years now:

![Photo of my desk setup](/images/20190127-matias-ergo-pro/photo.jpg)

## Pros
* Ergonomic for shoulders, not just wrists. 
* To an outside observer, touch typing on this thing looks like witchcraft. I would buy it with blank keycaps if I could.
* Mechanical. So clunk,  much travel.
* Dedicated `undo/cut/copy/paste` buttons, inverted-T arrow keys, function keys, (forward) delete key.

## Cons
* Wired, with big fugly wires and a spoolie between the two sides.
* Slightly glitchy. If you were thinking to yourself, *but better wires on my desktop than the vagaries of Bluetooth connections!* know that my keyboard, at least, needs unplugging and plugging back in every couple of weeks to fix repeating/non-functioning keys.
* Super heavy.
* Fairly spendy (but not as spendy as eight weeks of physio).
* Weird semi-Mac layout. Ginormous `Ctrl` key on the left, no `Option` key on the right, and I just noticed that there *is* a `Ctrl` key on the right, but it's above the spacebar. However, from mac 10.12 onwards [you can rebind keys yourself on the command line](https://developer.apple.com/library/archive/technotes/tn2450/_index.html), and I was happy to sacrifice the `End` and `Page Down` keys following [this SO post](https://apple.stackexchange.com/a/283253):

## Remapping Keys
* Use the `hidutil` command to remap keys: here I’m remapping `End` to be `Option`, and `Page Down` to be `Ctrl`, since they sit next to the `Command` key on the right half of the Matias. Those codes can be found at the bottom of the [TN](https://developer.apple.com/library/archive/technotes/tn2450/_index.html).

  ```
  hidutil property --set '{"UserKeyMapping":[{"HIDKeyboardModifierMappingSrc":0x70000004D,"HIDKeyboardModifierMappingDst":0x7000000e6},{"HIDKeyboardModifierMappingSrc":0x70000004e,"HIDKeyboardModifierMappingDst":0x7000000e4}]}'
  ```


* You can validate you got the right result by using the Keyboard Viewer (`System Preferences/Keyboard/Input/Show Input Menu`, then choose `Show Keyboard Viewer` from the menu bar icon).
* If you mess up, `hidutil property —set ‘{“UserKeyMapping”:[{}]}’` will remove all overridden keys set using `hidutil` (but not, for instance, `Caps Lock` set to the `Escape` key set in `System Preferences/Keyboard/Keyboard/Modifier Keys`).
* To persist this setting in between logins, put the command in an `.sh` file which will be run at every login:

  ```
  $ chmod 755 /path/to/keyboardmapping.sh
  $ sudo defaults write com.apple.loginwindow LoginHook ~/Library/Scripts/keyboardmapping.sh
  ```

## TL;DR
My physio approves, and I will only buy split keyboards in the future, but the search for "the one" continues. *Obviously*.
