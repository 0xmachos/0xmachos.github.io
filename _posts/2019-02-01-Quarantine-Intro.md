---
title:      "Quarantine Introduction"
subtitle:   ""
excerpt:   "Quarantine, added in OS X Leopard (10.5), was the forerunner to Gatekeeper. It adds the extended attribute com.apple.quarantine to files downloaded from the internet, so long as the downloading application is quarantine aware."
share-img:  "/img/division_quarantine.jpg"
tags:       [Quarantine, Gatekeeper]
---

![no-alignment](/img/posts/division_quarantine.jpg)


# Quarantine

Quarantine, added in OS X Leopard (`10.5`), was the forerunner to Gatekeeper. It adds the extended attribute (XA) `com.apple.quarantine` [^1] to files downloaded from the internet, so long as the downloading application is quarantine aware[^2].

{% include figure image_path="/img/posts/quarantine_prompt.png" alt="Quarantine prompt" caption="Files with the extended attribute `com.apple.quarantine` will trigger this prompt when you attempt to open them." %} 

As well as the above prompt, the application will be subject to a full Gatekeeper check[^3]. The process of a full check is described by [Howard Oakley](https://twitter.com/howardnoakley) in [What happens when you open a quarantined app?](https://eclecticlight.co/2018/08/03/what-happens-when-you-open-a-quarantined-app/).

The original idea behind Quarantine was to alert the user that the file they're about to open was download from the internet and get their explicit permission before opening. This was later supplemented with the Gatekeeper check when Gatekeeper was added in OS X Mountain Lion (`10.8`).


# Examining Extended Attributes

Files which have `@` at the end of their permissions flags have XAs.

```bash
$ ls -l
-rw-r--r--@ 1 mikey  staff  6888904  1 Feb 15:10 Brisk.app.tar.gz
```

To view which XAs a file has we use the `xattr` utility:

```bash
$ xattr Brisk.app.tar.gz 
com.apple.lastuseddate#PS
com.apple.metadata:kMDItemDownloadedDate
com.apple.metadata:kMDItemWhereFroms
com.apple.quarantine
```

To see the value of a particular extended atribute[^4]: 

```bash
$ xattr -p com.apple.quarantine Brisk.app.tar.gz 
0083;5c54614c;Safari;B555DB5F-D82A-408B-B9A6-D4F4012FD520
```


# Examining `com.apple.quarantine`

From the above example: 
```
0083;5c54614c;Safari;B555DB5F-D82A-408B-B9A6-D4F4012FD520
```

 1. `0083`
   - Flags 
 2. `5c54614c`
   - Download unix time in hex 
 3. `Safari`
   - Name of downloading application
 4. `B555DB5F-D82A-408B-B9A6-D4F4012FD520`
   - UUID used to identify the file in the `com.apple.LaunchServices.QuarantineEventsV2` database


# `QuarantineEventsV2` Database


macOS maintains an SQLite database of all files which have been assigned the `com.apple.quarantine` attribute. 

```
 ~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2
```

The structure of a database entry for the above example:

<script src="https://gist.github.com/0xmachos/73711442b346a1cb3b7b43256b74f66c.js"></script>

The most interesting fields:

- `LSQuarantineEventIdentifier`
  - We can use this UUID to tie a file to a DB entry
- `LSQuarantineTimeStamp`
  - When was the file downloaded
- `LSQuarantineAgentName`
  - This seems to be is more reliable than `LSQuarantineAgentBundleIdentifier`
- `LSQuarantineDataURLString`/ `LSQuarantineOriginURLString`
  - Where the file was downloaded from
- `LSQuarantineSenderName`
  - I've only seen this populated for files shared via `sharingd` aka Air Drop


## Dumping downloaded file URLs

With some SQL we can dump a list of all files which have been downloaded by Quarantine aware apps.

```sql
select DISTINCT LSQuarantineDataURLString from LSQuarantineEvent
```

We can use `sqlite3` to run this SQL query against the database. If you'd like a UI I like [DB Browser for SQLite](https://sqlitebrowser.org).

```bash
sqlite3 ~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2 'select DISTINCT LSQuarantineDataURLString from LSQuarantineEvent'
```


{% include figure image_path="/img/dogs/dachshund0.jpg" alt="Dachshund" caption="Photo by Kurt Sunkel on Unsplash" %} 


[^1]: [Show me your metadata: extended attributes in macOS Sierra](https://eclecticlight.co/2017/08/14/show-me-your-metadata-extended-attributes-in-macos-sierra/)
[^2]: [About the "Are you sure you want to open it?" alert in OS X](https://support.apple.com/en-gb/HT201940)
[^3]: [xattr: com.apple.quarantine, the quarantine flag](https://eclecticlight.co/2017/12/11/xattr-com-apple-quarantine-the-quarantine-flag/)
[^4]: [How to set (restore) the com.apple.quarantine attribute?](https://apple.stackexchange.com/questions/256625/how-to-set-restore-the-com-apple-quarantine-attribute)

