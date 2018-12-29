+++
date = "2018-05-11"
draft = false
title = "AccessControlKitty: an Xcode extension to manage access levels"
categories = ["Xcode"]

+++

I’ve put together a [little Xcode extension](https://github.com/zoejessica/AccessControlKitty) to change the access level of a selection of Swift code. For example, if you’ve decided to make a separate module out of some existing code, you can select some source code, click Editor -> Change Access Control -> Make public, and voila! All the selected classes, structs, properties, protocols, variables etc. etc. will magically be made public.

I find it super handy when going through a refactoring that’s pulling things out into their own framework, when the api is in flux and I’m not sure what needs to be public.

![Demo of extension](https://media.giphy.com/media/7zxZhrrxurVXg1oh5m/giphy.gif)

* Supports `public`, `private`, `fileprivate`, `internal` and removing any access annotation
* It’s not particularly smart, so for example it doesn’t know if a function _can’t_ be made public because it relies on an internal type.  And it certainly doesn’t know about anything going on in any other file.
* It also doesn’t support `open` or `final` for the moment, mostly because it’s a bit more work and just ship it already, and partly because I sort of feel those notations should require a bit more forethought when planning a framework.
* Eventually I’ll make it available as a downloadable app, but I’m sure there are still tons of corner cases I missed. For now if you’d like to play, please download it from [GitHub](https://github.com/zoejessica/AccessControlKitty). To install, you’ll need to archive the Mac app, export it without resigning to create an app you can just use locally. Then launch that app, and the extension will appear in System Preferences under the Extensions tab. Activate it, and after restarting Xcode all the options will appear at the bottom of the Editor menu. Phew.

### Tips for writing Xcode source code extensions

* Once the app has been exported, launched and installed, sometimes it will show up, but be grayed out in the Editor menu. I’ve found that this can be due to previous versions of the app hanging around in Derived Data (shakes fist) and shadowing the real app. Sometimes those old versions seem to persist despite cleaning, trashing the folder, emptying trash and restarting. (I KNOW.) A good way to begin debugging this is with the `pluginkit` command line tool:
	* `pluginkit -m -p com.apple.dt.Xcode.extension.source-editor ` lists all plugins known to Xcode
	* `pluginkit -m -p com.apple.dt.Xcode.extension.source-editor -A -D -vvv  `  to be extra verbose, handy to find what folders they’re hanging around in
*  Naming in the `plist` is a bit inscrutable:
	* Bundle name = the Editor menu item
	* Bundle display name = name of the extension in the Extensions pane of System Preferences
	* Extension bundle identifier = what shows up in the `pluginkit` listing in Terminal
