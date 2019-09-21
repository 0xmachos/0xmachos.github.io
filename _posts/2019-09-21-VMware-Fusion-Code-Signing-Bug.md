---
title:		"VMware Fusion Code Signing Bug"
subtitle:	"Updating VMware Fusion broke its code signature"
excerpt:	"VMware have released Fusion 11.5. The update addresses a bug which would cause the code signature of the VMware Fusion application bundle to be broken after applying updates."
share-img:	"/img/posts/vmware-bug/vmware.png"
image:		"/img/posts/vmware-bug/vmware.png"
tags:		[VMware, Bug]
---

This week VMware released Fusion `11.5` . Fusion is VMware's virtualisation software for macOS. The update, among other things, addresses a bug which would cause the code signature of the VMware Fusion application bundle (`.app`) to be broken after applying updates.

To upgrade to Fusion `11.5` you'll need to download a package from VMware, you can't update via the builtin software updated because it suffers from the bug. You can get the `11.5` installer from [vmware.com/go/getfusion](https://www.vmware.com/go/getfusion). 

## The Bug

When upgrading Fusion, Fusion would download updated files from the update server and combine them into a new `VMware Fusion.app`. Adding these new files breaks the code signature. 

## Discovery

On Christmas Eve 2018 I was writing a [bash script](https://github.com/0xmachos/macos-scripts/blob/master/signature_check) to check the code signing status of every application bundle on my system. 

The script gets a list of all application bundles by
executing `system_profiler SPApplicationsDataType`, for each application it checks their code signature by executing `pkgutil --check-signature`.

I noticed that it was reporting VMware Fusion 11 (`11.0.2`) as not being properly signed, specifically `pkgutil` gave the following error.

```
Package "VMware Fusion.app":
   Status: package is invalid (checksum did not verify)
```

I assumed this was an error with `pkgutil` or that I'd accidentally modified `VMware Fusion.app` and broken the signature. I decided to see if it was reproducible so I went on to [Abertay Ethical Hacking Society](https://twitter.com/abertayhackers)'s Slack and jumped in the #mac channel.

![no-alignment](/img/posts/vmware-bug/slack-question.png)  

[Ninji](https://twitter.com/_Ninji) was kind enough to attempt to reproduce it for me. They ran the `pkgutil` command and the signature was fine. They then realised that they had Fusion `11.0.0` so they updated to `11.0.2` and reran the `pkgutil` command and sure enough the signature was broken.

## VMware Response

I had a very positive experience disclosing this issue to VMware. They were prompt in their replies and happy to share technical details of what they learned about the bug and their fix. 

In an email on January 26th VMware indicated that the fix wasn't straightforward and would be shipped in the next major release. They asked me to wait until the fix was available before publicly disclosing which I agreed to.

The bug is listed in the "Resolved Issues" section of the `11.5` [release notes](https://docs.vmware.com/en/VMware-Fusion/11.5.0/rn/VMware-Fusion-1150-Release-Notes.html#resolvedissues). 

## Timeline

#### 2018
- December 24th - Discovered and reported
- December 25th - Report acknowledged

#### 2019
- January 26th 	- Bug confirmed by VMware
- May 10th - VMware confirmed the bug would be fixed in the next major release in September
- September 6th - VMware confirm the major release is out in the second half of September
- September 16th - Update is released
- September 21nd - I publicly disclose the bug

{% include figure image_path="/img/dogs/dog2.jpg" alt="Happy border collie with tongue out lying on grass" caption="Photo by Berkay Gumustekin on Unsplash" %}
