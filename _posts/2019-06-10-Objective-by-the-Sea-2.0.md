---
title:		"Objective by the Sea v2.0"
subtitle:	"Mac Security by the Sea"
excerpt:	"I took my first trip to Monaco for Objective by the Sea v2.0. An absolutely stunning venue for an excellent conference. "
share-img:	"/img/posts/obts2/logo.png"
image:		"/img/posts/obts2/logo.png"
tags:		[Conference]
---

I took my first trip to Monaco for Objective by the Sea ([#OBTS](https://twitter.com/hashtag/OBTS)) v2.0. An absolutely stunning venue for an excellent conference. Massive thanks to [Patrick](https://twitter.com/patrickwardle) & [Andy](https://twitter.com/andyrozen) for organizing it.

It was wonderful to finally meet so many people I know from Twitter.

Every talk was excellent but I'll let [the videos](https://www.youtube.com/playlist?list=PLliknDIoYszvTDaWyTh6SYiTccmwOsws8) speak for themselves, below I'll briefly cover my three favorite talks. 

# "Fun with Mac Malware Attribution"

#### By [Josh Long](https://twitter.com/theJoshMeister) from [Intego](https://www.intego.com/).

Josh came to [talk about](https://objectivebythesea.com/v2/talks.html#long) unmasking Mac malware authors and did not disappoint. 

He walked us through the incredible operation security (OPSEC) failures of the [Coldroot RAT](https://www.intego.com/mac-security-blog/osxcoldroot-and-the-rat-invasion/) developer.

The other highlight was some great Open Source Intelligence (OSINT). Josh picked up where Patrick left off in his [analysis of the CreativeUpdater cryptominer](https://digitasecurity.com/blog/2018/02/05/creativeupdater/). Patrick found reference to a "Tiago Brandão Mateus" in a `.DS_Store` file in the root directory of malware's `.DMG`. Josh took the name and ran with it. 

A highly informative and hilarious talk. Look out for his whitepaper. 


# "Bash-ing Brittle Indicators: Red Teaming macOS without Bash or Python"

#### By [Cody Thomas](https://twitter.com/its_a_feature_) from [SpecterOps](https://specterops.io/). ([Slides](https://www.slideshare.net/CodyThomas6/bashing-brittle-indicators-red-teaming-macos-without-bash-or-python))

Cody [talked about](https://objectivebythesea.com/v2/talks.html#thomas) his post-exploitation and red teaming framework [Apfell](https://github.com/its-a-feature/Apfell). Specifically, its use of JavaScript for Automation (JXA). 

> In this talk, I'll go into the research, development, and usage of a new kind of agent based on JavaScript for Automation (JXA) and how it can be used in modern red teaming operations. 

He covered a lot so I'll highlight just a couple of things I found interesting. 

### Download Cradle

Rather than `curl`ing a Bash script and piping it to `sh` you can use Apple Script (`osascript`). Using `osacompile` you can compile Javascript files containing JXA code. `osascript` is signed by Apple and the compiled code will run in memory. Less suspicious looking and no files on disk. 

![no-alignment](/img/posts/obts2/cody_slide_0.png)

### Information Discovery

Instead of invoking `sw_vers`, `system_profiler` or `ifconfig` we can read most of that info from the file system. File system reads are less suspicious than invoking these binaries in quick succession. 

Cody recommends checking the following locations: 

- `~/Library/Preferences`
- `/Library/Preferences`
- `/System/Library/Preferences`

![no-alignment](/img/posts/obts2/cody_slide_1.png)

# "Bad Things in Small Packages"

#### By [Jaron Bradley](https://twitter.com/jbradley89) from [CrowdStrike](https://www.crowdstrike.com/).

This was my favorite talk, the bug is hilariously simple yet powerful. The [abstract](https://objectivebythesea.com/v2/talks.html#bradley) was kinda vague but promised Local Privileged Escalation (LPE) and, more interestingly, a System Integrity Protection (SIP) bypass. 

> The vulnerability exists within PackageKit that could lead to privilege escalation, signature bypassing, and ultimately the bypassing of Apple's System Integrity Protection (SIP).

The vulnerability in question is `CVE-2019-8561`. 

Jaron dived straight in with a demo of LPE aspect of the bug. I answered a text and by the time I looked back up he had root, it was so simple I thought I must have missed something.

Essentially this is a Time of Check Time of Use (TOCTOU) bug. Once `Installer` loads a `.pkg` you can unpack the `.pkg` modify it, repack it and `installer` will execute the modified files. All while still showing that the package is signed.

An excellent find, really neat bug that's trivial to exploit. I think his Bash script exploit was around 5 lines. 

At this point I couldn't see how we could use this to bypass SIP. 

The answer is an Apple signed installer package with the `com.apple.rootless.install.heritable` [^1] entitlement. This indicates to the OS that even when SIP is enabled the executable is allowed to write to SIP protected locations. Crucially though this entitlement is appended with `heritable`, meaning any child process can also write to SIP protected locations.

I can't remember which Installer Package Jaron used for his SIP bypass demo, I think it was Final Cut Pro or one of Apple's creative pro applications.

The steps are almost identical, start the installer, unpack, modify, repack the `.pkg`. Now your malicious `.pkg` can write to SIP protected locations because it inherits `com.apple.rootless.install.heritable`. The only difference is which process you race. Once `Installer` notices the installer package is entitled to write to SIP protected locations it hands off to another process, again I can't remember the process name.

I think most of us couldn't believe how simple the bug is. You can subvert SIP with few lines of bash.

# OBTS v3.0

In his closing presentation Patrick announced that OBTS v3.0 will be early 2020 in Hawaii. If you're interested in Mac security I cannot recommended OBTS enough.

See you in Hawaii!

P.S thanks for the free t-shirt!

<blockquote class="twitter-tweet tw-align-center" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/0xmachos?ref_src=twsrc%5Etfw">@0xmachos</a> &quot;live-tweeted&quot; much of the &quot;Objective by the Sea&quot; conference (which is one of the reasons the hashtag, <a href="https://twitter.com/hashtag/OBTS?src=hash&amp;ref_src=twsrc%5Etfw">#OBTS</a>, was trending in Monaco 🤭)<br><br>For his efforts &amp; support (plus being an all-around swell chap), he won an official conf t-shirt 👕🍎🥳<br><br>Thanks again <a href="https://twitter.com/0xmachos?ref_src=twsrc%5Etfw">@0xmachos</a>! <a href="https://t.co/s1D2509EdA">pic.twitter.com/s1D2509EdA</a></p>&mdash; Objective-See (@objective_see) <a href="https://twitter.com/objective_see/status/1137874681950720001?ref_src=twsrc%5Etfw">June 10, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

{% include figure image_path="/img/dogs/dog0.jpg" alt="Golden in forest with sun behind" caption="Photo by Andreas Wagner on Unsplash" %}

[^1]: [com.apple.rootless.install.heritable](http://newosxbook.com/ent.jl?ent=com.apple.rootless.install.heritable&osVer=MacOS14)

