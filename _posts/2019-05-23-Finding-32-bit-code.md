---
title:		"Finding 32-bit code on macOS"
subtitle:	""
excerpt:	"During the Platform State of the Union at WWDC 2018 Apple confirmed that macOS Mojave (10.14.x) will be the last version of macOS to run 32-bit code."
share-img:	"/img/posts/mojave-32-bit-apps.png"
tags:		[Mojave]
---

{% include figure image_path="/img/posts/mojave-32-bit-apps.png" alt="2018 Platform State of the Union" caption="2018 Platform State of the Union" %} 

During the [Platform State of the Union](https://developer.apple.com/videos/play/wwdc2018-102/?time=1179) at WWDC 2018 Apple confirmed that macOS Mojave (`10.14.x`) will be the [last version of macOS to run 32-bit code](https://support.apple.com/en-gb/HT208436).

# `system_profiler`

This one-liner is courtesy of [@MacLemon](https://twitter.com/MacLemon). 

```bash
system_profiler SPApplicationsDataType \
    | grep -E '^    .*:$|64-Bit \(Intel\):|Location' \
    | grep -A1 ': No' \
    | grep 'Location' \
    | sed -e 's/.*Location: //' 
```

On my system this command finds:
```
/System/Library/Input Methods/InkServer.app
/System/Library/Frameworks/QuickLook.framework/Versions/A/Resources/quicklookd32.app
```

# Spotlight (`mdfind`)

[Rich Trouton](https://twitter.com/rtrouton) recommends [using Spotlight](https://twitter.com/rtrouton/status/1130482310762115072).

```bash
mdfind "kMDItemExecutableArchitectures == '*i386*' && kMDItemExecutableArchitectures != '*x86*'"
```

This finds significantly more 32-bit code:
```
/System/Library/Input Methods/InkServer.app
/System/Library/Frameworks/QuickLook.framework/Versions/A/Resources/quicklookd32.app
...
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/Metadata.framework/Versions/A/Support/libmdworker.dylib
/System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/NavigationServices.framework/Versions/A/NavigationServices
...
/System/Library/PrivateFrameworks/CoreMediaIOServicesPrivate.framework/Versions/A/CoreMediaIOServicesPrivate
/System/Library/Printers/Libraries/libConverter.dylib
```

Base on my limited testing I'd advise following Rich's recommendation, use `mdfind` to find 32-bit code.

{% include figure image_path="/img/dogs/dachshund01.jpg" alt="Dachshund photo by Carissa Weiser on Unsplash" caption="Photo by Carissa Weiser on Unsplash" %}
