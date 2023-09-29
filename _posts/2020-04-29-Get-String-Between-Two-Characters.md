---
title:		"Get String Between Two Characters"
subtitle:	"A simple regex"
excerpt:	"By default awk splits strings into columns based on whitespace. We can tell awk to use a different input field separator (IFS) via the -F option."
tags:		[Awk, Regex, Shell]
---

Let's use the [EICAR test file](https://en.wikipedia.org/wiki/EICAR_test_file) as an example.

```
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

How do we extract the string `EICAR-STANDARD-ANTIVIRUS-TEST-FILE`?

```shell
$ echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' | awk -F '[$|!]' '{print $3}'
EICAR-STANDARD-ANTIVIRUS-TEST-FILE
```
By default `awk` splits strings into columns based on whitespace. We can tell `awk` to use a different input field separator (IFS) via the `-F` option.

In this case our IFS is `'[$|!]'` which reads as `$` OR `!`. This means that everything between `$` and `!` becomes a column which we can print using the standard `awk` syntax of `'{print $3}'`.

You'll need a bit of trial and error to find which column to print.

Learned while working on [0xmachos/checkMach](https://github.com/0xmachos/checkMach) trying to extract the `CFBundleExecutable` string from an App `Info.plist` file. 

![no-alignment](/img/dogs/dog1.jpg)
<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@jennymarvin?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Jenny Marvin"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Jenny Marvin</span></a>

