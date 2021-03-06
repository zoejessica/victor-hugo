---
date: "2019-12-30"
draft: false
title: "Photos -> Hazel -> Dropbox -> Blot -> zoejessica.com 🎉"
slug: hazel-to-blot

categories:
- Workflow
tags:
- photography
- productivity
- macos
- workflow
- review
description: Hazel to Blot
---

Inspired by [@bsag](https://www.wingsopenwide.org.uk), I’ve set up a very minimal photo travel blog on [zoejessica.com](http://www.zoejessica.com). I’ve long been wanting to share more photos again (rip old school flickr), but without falling into Instagram by default. I don’t *_mind_* insta per se, I actually love it for design inspiration and find it incredibly useful for bodybuilding tips, but I’m not so much about hashtagging and captioning and all that. And I didn't want anything too complicated that I could over-organize my way to never using: I think I must have used up my patience for metadata-ing things back in the Limewire media firehose days 😬.

Anyway. [Blot](blot.im) is a fantastic, light static site generator for just getting stuff online without fuss or ceremony, with really nice, minimal templates that look fantastic ootb. It watches a folder in Dropbox (or git, if you’re getting twitchy about Dropbox) and pushes whatever’s in there to a domain of choice.  

The neat part is that Blot'll extract metadata encoded in filenames. This, in conjunction with [Hazel](https://www.noodlesoft.com)’s folder actions, makes for a super-simple workflow:

I’ve set up Hazel to watch a `BlotBox` folder, using the [recursive rule](https://www.noodlesoft.com/manual/hazel/advanced-topics/processing-subfolders/) to apply a renaming action to any files in subfolders. When I export photos, I put them in a subfolder named for the tag I want to apply – I’m using locations, so something like `BlotBox/Avignon` .  

![Hazel renaming action](/images/20191230/hazel-rule.png)

The renaming action (on the top level `BlotBox` folder) pulls the capture date (in `YYYY-MM-DD` format) using Hazel's specialized `date taken` variable, while the name of the source subfolder is applied as both tag (in `[]`) and name (see metadata exhaustion syndrome, above). So publishing becomes:

1. Export jpegs from Photos into a subfolder named for the tag
2. There is no step 2! Hazel runs the renaming action, moves the photos to `Dropbox/Apps/Blot` where Blot picks ‘em up and publishes them.

So easy. So much more time freed up for theme tweaking 😅.

(Took me bloody ages, but I managed to create an [iOS Shortcut](https://www.icloud.com/shortcuts/58cd0cbbe31340649d08f99dcdf8ca53) which does the same thing.)
