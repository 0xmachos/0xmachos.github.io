---
title:		"macOS USB Installer"
subtitle:	"No Third Party Software Required"
excerpt:	"You can create a bootable USB to install macOS with one command, without any third party software."
tags:		[macOS, TIL]
---

# macOS USB Installer

You can create a bootable USB to install macOS on other Macs with one command, without any third party software. The macOS Installer from the App Store contains a binary, `createinstallmedia` that will do it all for you.


```
sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume
```

See Apple's support page ["How to create a bootable installer for macOS"](https://support.apple.com/en-us/HT201372) for full instructions.


![no-alignment](/img/dogs/dog2.jpg)
Photo by <a href="https://unsplash.com/@stevetsang">Steve Tsang</a> on Unsplash


  