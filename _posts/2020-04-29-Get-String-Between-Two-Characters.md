---
title:		"Get String Between Two Characters"
subtitle:	"A simple regex"
excerpt:	"By default awk splits strings into columns based on whitespace. We can tell awk to use a different input field separator (IFS) via the -F option."
tags:		[Awk, Regex, Shell, TIL]
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
Photo by <a href="https://unsplash.com/@jennymarvin">Jenny Marvin</a> on Unsplash</a>
