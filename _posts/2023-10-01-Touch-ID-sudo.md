---
title:		"Persistent Touch ID for sudo"
subtitle:	"A macOS Sonoma Bonus Feature"
excerpt:	"Touch ID can be allowed for sudo with a configuration that persists across software updates using /etc/pam.d/sudo_local."
share-img:	"/img/posts/touchid.png"
thumbnail-img:	"/img/posts/touchid.png"
tags:		[Sonoma, Shell]
gh-repo: 0xmachos/macos-scripts

---


Apple first shipped Touch ID for Mac in the [2016 MacBook Pros](https://support.apple.com/kb/SP749?locale=en_GB) (aka the first "Touch bar" Macs) running macOS Sierra (`10.12`). 

Almost immediately [folks wrote their own](https://github.com/Reflejo/pam-touchID) pluggable authentication module (PAM) plugin modules to enable using TouchID to authenticate `sudo`.

These days Apple ships a PAM plugin module with macOS, `/usr/lib/pam/pam_tid.so.2`, that can be used to enable Touch ID for `sudo`. As best [I can tell](https://apple.stackexchange.com/a/306324) this was first shipped with macOS High Sierra (`10.13`) in September 2017 but I might be wrong.

All you had to do was add the following to `/etc/pam.d/sudo` and you could use Touch ID to authenticate `sudo`.

```
auth       sufficient     pam_tid.so
```

![center-aligned-image](/img/posts/touchid-sudo.png){: .align-center}

The only issue with this is that `/etc/pam.d/sudo` is overwritten on every macOS update. Major, minor or patch; it is always overwritten and reset back to its default state.

Not ideal. 

As far as I understand it there was no way around this, other than to edit `/etc/pam.d/sudo` after every update.

Looking at the [BSD documentation for PAM](https://docs.freebsd.org/en/articles/pam/#pam-config-pam.d) we see the following:

>OpenPAM and Linux-PAM support an alternate configuration mechanism, which is the preferred mechanism in FreeBSD. In this scheme, each policy is contained in a separate file bearing the name of the service it applies to. These files are stored in /etc/pam.d/.

Reading the above it appears that the only place we can configure how PAM authenticates `sudo` is `/etc/pam.d/sudo`. We cannot create another file in `/etc/pam.d/` that PAM will read configuration for `sudo` from.


## macOS Sonoma

In their "[What's new for enterprise in macOS Sonoma](https://support.apple.com/en-us/HT213893)" document Apple listed the following in the "Bug fixes and other improvements" section:

> Touch ID can be allowed for `sudo` with a configuration that persists across software updates using `/etc/pam.d/sudo_local`. See `/etc/pam.d/sudo_local.template` for details.

![center-aligned-image](/img/posts/about-time.gif){: .align-center}

All we need to do is create the file `/etc/pam.d/sudo_local` with `auth       sufficient     pam_tid.so` as its contents. 

To match the rest of the PAM configuration files in `/etc/pam.d/`, `sudo_local` must have the following permissions:

* owner = `root`
* group = `wheel`
* read only (`444`)


I've got a quick shell script that will do all this for you [0xmachos/macos-scripts/enable-touchid-sudo](https://github.com/0xmachos/macos-scripts/blob/master/enable-touchid-sudo).


![no-alignment](/img/dogs/dog7.jpg)
Photo by <a href="https://unsplash.com/@alexandra_photography">Alexandra Lau</a> on <a href="https://unsplash.com/photos/YRUzuSC48Zs">Unsplash</a>
