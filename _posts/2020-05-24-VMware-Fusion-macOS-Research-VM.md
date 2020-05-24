---
title:		"VMware Fusion macOS Research VM"
subtitle:	"A Living Guide..."
excerpt:	"If you're at university you can get VMware Fusion free or at a discounted price via OnTheHub"
share-img:	"/img/posts/vmware-bug/vmware.png"
image:		"/img/posts/vmware-bug/vmware.png"
tags:		[VMware, Guide]
---

If you're at university you can get VMware Fusion (Henceforth “Fusion”) free or at a discounted price via [OnTheHub](https://onthehub.com/download/software-discounts/vmware). 

## Initial Setup

(Alternatively you can use [vfuse](https://github.com/chilcote/vfuse) by [Joseph Chilcote](https://twitter.com/chilcote))

1. Download macOS XXX installer from App Store
2. Once the download completes quit the installer
    1. Open Disk Utility and unmount the `InstallESD` disk image
3. In Fusion create a new VM via the “Install from disc or image” option
    1. Select the `Install macOS XXX` installer located in `/Applications`
4. Click “Customise Settings” at the bottom of the Finish screen
5. Save the VM with an appropriate name
6. The “Creating Install Media” step can take up to five (5) minutes

## VM Settings

**Rational**: You almost certainly don’t need the camera. 2 cores and 3GB of RAM has worked well for me over the years, your mileage may vary. 

1. Remove Camera
3. Processors & Memory
    1. Processors: 2 cores
    2. Memory: 3072MB (3GB)
4. **Do not** enabled accelerated 3D graphics

## Edit VMX File

(**Not required if you're using Fusion ≥ 11.5**)

**Rational**: Prevents the VM from searching for a Touch ID sensor or SEP. This search introduces a delay when the system prompts for your password to carry out an action as root.

In the Virtual Machine Library (⇧⌘L):
1. Right click the VM
2. Press option key (⌥)
3. Click "Open Config File in Editor"

Change
```
board-id.reflectHost = "TRUE"
```
to 
```
board-id.reflectHost = "FALSE"
```

See [Preventing macOS VM Authentication Delays](https://0xmachos.github.io/2019-08-24-Preventing-macOS-VM-Authentication-Delays/) for more info.


## macOS Install

1. Boot the VM 
2. Select a language
3. Select “Install macOS” option in the “macOS Utilities” window then click “Continue“
4. Agree to the terms/ license
5. Select “Macintosh HD” as the install disk, click “Install”
6. Wait approximately thirty (30) minutes for install to finish
7. Select a country
8. Click through setup
9. Click “Set Up Later” for Apple ID
10. Agree to terms & conditions
11. Create an account (Will have Administrator privileges)
    1. Maybe consider some #opsec if you’re going to be running malware in this VM e.g. Don’t use your real name 
12. At the “Express Set Up” screen click “Customise Settings”
13. Disable Location Services
14. Select your time zone
15. Disable **all** analytics

## Install VMWare Tools

1. Virtual Machine > Install VMware Tools
2. Click “Install”
3. Double click “Install VMware Tools” package
4. Follow the installer instructions (the defaults are all fine)
5. When the “System Extension Blocked” window appears click “Open Security Preferences”
    1. Click “Allow”
6. Click “Restart”
7. Once the VM restarts if the display resolution doesn't automatically adjust you'll need to install VMware Tools again and restart the VM

## macOS Configuration 

**Optional:** Will probably make your lifer easier, basically stops the VM going to sleep.

1. System Preferences > Desktop & Screen Saver > Screen Saver
    1. Set “Start after” to “Never”
2. System Preferences > Energy Saver
    1. Set “Computer Sleep” & “Display Sleep” to “Never”
    2. Uncheck “Put hard disks to sleep when possible”
3. System Preferences > Security & Privacy
    1. Uncheck “Require password x minutes after sleep or screen saver begins”

## Third Party Tools

* [Hopper Disassembler](https://www.hopperapp.com/)
* [The Unarchiver](https://theunarchiver.com/)
* [Sublime Text](https://www.sublimetext.com/)
* [jtool2](http://www.newosxbook.com/tools/jtool.html)
* [Suspicious Package](https://mothersruin.com/software/SuspiciousPackage/get.html)
* [FireEye Monitor](https://www.fireeye.com/services/freeware/monitor.html)
    * Doesn’t work on Catalina (`10.15.x`)
* [Crescendo](https://github.com/SuprHackerSteve/Crescendo)
	* Only works on Catalina as it uses the Endpoint Security Framework
* [Appmon](https://bitbucket.org/xorrior/appmon/src/master/)
    * Only works on Catalina as it uses the Endpoint Security Framework
* [ProcessMonitor](https://objective-see.com/products/utilities.html#ProcessMonitor)
    * Only works on Catalina as it uses the Endpoint Security Framework
* [FileMonitor](https://objective-see.com/products/utilities.html#FileMonitor)
    * Only works on Catalina as it uses the Endpoint Security Framework

## Builtin Tools

* `fs_usage`
* `codesign`
* `pkgutil`
* `dtrace`
