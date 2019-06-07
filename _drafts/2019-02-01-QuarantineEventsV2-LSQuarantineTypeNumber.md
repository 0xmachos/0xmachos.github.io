---
title:		"QuarantineEventsV2: LSQuarantineTypeNumber"
subtitle:	""
excerpt:    ""
share-img:	"/img/posts/..."
image:		"/img/posts/..."
tags:		[Quarantine, Gatekeeper]
---

# `LSQuarantineTypeNumber`

This field in the `QuarantineEventsV2` datqabase indicates the type of downloaded file.

The constants `LSQuarantineTypeNumber` maps to are documented[^1] by Apple, however Apple provides no mapping.

- [kLSQuarantineTypeCalendarEventAttachment](https://developer.apple.com/documentation/coreservices/klsquarantinetypecalendareventattachment)
- [kLSQuarantineTypeEmailAttachment](https://developer.apple.com/documentation/coreservices/klsquarantinetypeemailattachment)
- [kLSQuarantineTypeInstantMessageAttachment](https://developer.apple.com/documentation/coreservices/klsquarantinetypeinstantmessageattachment)
- [kLSQuarantineTypeOtherAttachment](https://developer.apple.com/documentation/coreservices/klsquarantinetypeotherattachment)
- [kLSQuarantineTypeOtherDownload](https://developer.apple.com/documentation/coreservices/klsquarantinetypeotherdownload)
- [kLSQuarantineTypeWebDownload](https://developer.apple.com/documentation/coreservices/klsquarantinetypewebdownload)

# Mapping the constants

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">It seems related to the quarantine agent: in my case Firefox (browsers) is 0, and Transmission (torrent) is 1. Processes like jDownloader, curl etc. don&#39;t seem to be quarantine-relevant. <a href="https://t.co/LQzEUxhCJX">https://t.co/LQzEUxhCJX</a> Are there more quarantine types? <a href="https://twitter.com/howardnoakley?ref_src=twsrc%5Etfw">@howardnoakley</a></p>&mdash; bucky (@buckymsj) <a href="https://twitter.com/buckymsj/status/1090268854268309505?ref_src=twsrc%5Etfw">January 29, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">I&#39;ve also reproduced this. No entries for <a href="https://t.co/pWUvkl65qM">https://t.co/pWUvkl65qM</a> in the DB. Most of my browser downloads have LSQuarantineTypeNumber set to 0 but I have a .zip and keynote (.key) that are set to 7. iMessage attachments are 3 and sharingd (airdrop) 6</p>&mdash; mikey (@0xmachos) <a href="https://twitter.com/0xmachos/status/1091089974194327554?ref_src=twsrc%5Etfw">January 31, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 


LSQuarantine.h

MacOSX.sdk/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Headers


## Known 
- kLSQuarantineTypeWebDownload = `0`
- kLSQuarantineTypeInstantMessageAttachment = `3`

## Maybe 
- kLSQuarantineTypeOtherAttachment = `6`
- kLSQuarantineTypeOtherDownload = `7`

## Unknown
- kLSQuarantineTypeCalendarEventAttachment
- kLSQuarantineTypeEmailAttachment


[^1]: For some definition of "documented", cheers Apple...
