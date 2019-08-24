---
title:		"Preventing macOS VM Authentication Delays"
subtitle:	"A quick fix"
excerpt:	"When you virtualise macOS on a host with a Secure Enclave Processor the virtualised OS looks for the SEP to perform authentication actions. As described below by @macshome this leads to a delay whenever you try to perform some authentication actions via the UI.  "
tags:		[VMware]
---

# Problem 

When you virtualise macOS on a host with a Secure Enclave Processor (SEP) (any Mac with Touch ID or T2 chip) the virtualised operating system (OS) looks for the SEP to perform authentication actions. 

As described below by [@macshome](https://twitter.com/macshome), this leads to a delay whenever you try to perform some authentication actions via the User Interface (UI).  

<blockquote class="twitter-tweet tw-align-center"><p lang="en" dir="ltr">If you reflect the host board then macOS will assume there is a SEP and TouchID module that it should be using. In the VM though there isn’t one so you are forced to wait for a timeout on every authentication while it desperately looks for the module.</p>&mdash; macshome (@macshome) <a href="https://twitter.com/macshome/status/1139665263266324480?ref_src=twsrc%5Etfw">June 14, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

You can see an example of this delay, twenty four (24) seconds, in the below video: 
![no-alignment](/img/posts/vm-auth/auth-delay.mov)

# Solution

<blockquote class="twitter-tweet tw-align-center"><p lang="en" dir="ltr">With so many people running macOS in a VM right now here is a ProTip for Fusion on a TouchID Mac.<br><br>Set:<br>board-id.reflectHost = &quot;FALSE&quot;<br><br>Now your auth dialog spins are gone!</p>&mdash; macshome (@macshome) <a href="https://twitter.com/macshome/status/1137003871975481345?ref_src=twsrc%5Etfw">June 7, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Once this change is applied there's no longer a delay. The authentication flow only takes six (6) seconds.
![no-alignment](/img/posts/vm-auth/auth-fixed.mov) 

## Editing the `.VMX` File

You need to edit the `.VMX` file which is located inside the VMBundle of your VM. While it appears as a single file in Finder the VMBundle is actually an archive. This is where your VMs hard disk and a bunch of other files and logs reside. See [VMware KB article 1021016](https://kb.vmware.com/s/article/1021016) for more info on VMbundle contents.  

![no-alignment](/img/posts/vm-auth/vmbundles.png)

By default VMware saves your VMs to:
```
$HOME/Virtual Machines
```

You can either `cd` into the VMBundle
![no-alignment](/img/posts/vm-auth/command-line.png)

or use the Finder's "Show Package Contents" context menu option.
![no-alignment](/img/posts/vm-auth/finder.png)

Once you're inside the VMBundle open the VMs `.vmx` file (from the examples above it would be `macOS Mojave.vmx`) and change 

```
board-id.reflectHost = "TRUE"
```

to 

```
board-id.reflectHost = "FALSE"
```

**The `.vmx` file is included in snapshots** so you'll want to perform this edit as soon as you create your VM. If you make this change and restore a snapshot you created before editing the `.vmx` file the change won't persist, the **previous state of the `.vmx` file will be restored by the snapshot**.

{% include figure image_path="/img/dogs/collie0.jpg" alt="Happy border collie with tongue out lying on grass" caption="Photo by daniel plan on Unsplash" %}
