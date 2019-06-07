---
title:		"Frequency of Automatic Update Checks in Mojave"
subtitle:	""
excerpt:    ""
share-img:	"/img/posts/..."
image:		"/img/posts/..."
tags:		[SoftwareUpdate]
---

# Introduction

I was reccently reading [Rich Trouton](https://twitter.com/rtroutonx)'s post [Enabling automatic macOS software updates for OS X Yosemite through macOS Mojave](https://derflounder.wordpress.com/2018/12/28/enabling-automatic-macos-software-updates-for-os-x-yosemite-through-macos-mojave/) and realised that [mOSL](https://github.com/0xmachos/mOSL) wasn't properly auditing/fixing the behaviourt of automatc update. 

While I was fixing this, [#17](https://github.com/0xmachos/mOSL/issues/17), I also realise that I hadn't verfied that the `ScheduleFrequency` key in `com.apple.SoftwareUpdate` was still valid in Mojave.


Before we go on i'd like to poiint out that the exsistance of this key isn't new, [Change the frequency of software updates checks](https://www.defaults-write.com/change-the-frequency-of-software-updates-checks/), I'm just confirming that it still works as ecpected in Mojave.
{: .notice--info}

# Automatic macOS Updates

## Check vs Install

There's a difference between automatically checking for updates and automatically installing updates. 

The following controls the behaviour of automativally **checking** for updates:
```
/Library/Preferences/com.apple.SoftwareUpdate.plist AutomaticCheckEnabled
```

The following controls the behaviour of automativally **installing** for updates:

```
/../com.apple.SoftwareUpdate.plist AutomaticallyInstallMacOSUpdates
```

`AutomaticallyInstallMacOSUpdates` only controls the automatic installation of **OS** updates. It doesn't affect the automatic intsallation of XProtect, MRT and Gatekeeper updates. 
{: .notice--danger}

## Check Frequency

If `com.apple.SoftwareUpdate.plist AutomaticCheckEnabled` is set to `true` then macOS will automatically check for new updates every seven (`7`) days. 


We can control how often the automatic update check occours with the following key:

```
com.apple.SoftwareUpdate ScheduleFrequency 
```

For example, the following will have macOS check for updates every one (`1`) day:

```
defaults write com.apple.SoftwareUpdate ScheduleFrequency -int 1
```

# `install.log`


The CLI utility `softwareupdate` and it's deamon `softwareupdated` both log to:

```
/private/var/log/install.log
```


