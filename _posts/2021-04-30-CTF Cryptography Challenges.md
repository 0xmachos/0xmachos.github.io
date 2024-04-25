---
title:		"CTF Writeup"
subtitle:	"Crypto Challenges Writeup"
excerpt:	"GCHQ's CyberChef tool is used extensively throughout this post."
share-img:	"/img/posts/ctf-crypto/crypto.jpg"
image:		"/img/posts/ctf-crypto/crypto.jpg"
tags:		[CTF, Crypto]
---

This post describes how I solved the Crypto (Crypto means Cryptography) challenges for a CTF I played in 2020.

GCHQ's [CyberChef](https://gchq.github.io/CyberChef) tool is used extensively throughout this post.

# Basic Crypto (Easy)

```base64
ZGhjdGZ7aXNfdGhpc19hc19lYXN5X2FzX2l0X2dldHM/P30=
```

We're presented with the above string and asked if we can decode it. 

The first thing I noticed was the equals sign (`=`) at the end of the string. This usually indicates the string is encoded with Base64 (There are other base encoding [like 16 and 32](https://tools.ietf.org/html/rfc3548) but it's usually Base64).

I jumped into CyberChef and tried the [From Base64 operation](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)&input=WkdoamRHWjdhWE5mZEdocGMxOWhjMTlsWVhONVgyRnpYMmwwWDJkbGRITS9QMzA9).

Sure enough, this gave me the flag: `dhctf{is_this_as_easy_as_it_gets??}`.

# Zippy McZip Face (Medium)

The challenge gives us an encrypted zip file, `zippy.zip`, and asks if we can retrieve the flag from it. 

As this is a CTF I assumed the password had to be something guessable in a reasonable amount of time. This lead me to try the RockYou wordlist, it ships with Kali and passwords in it are commonly used in CTFs.

macOS comes with the `unzip` command which can be used to expand zip archives. The `-P` option tells `unzip` that the zip archive is encrypted and to use the string after it as the decryption key (password).

I put together a small Bash script: 

```bash
#!/bin/bash
while IFS= read -r password
do
  unzip -P "${password}" zippy.zip
done < rockyou.txt
```

_(The above script won't work on macOS with stock `unzip`. It will fail with `need PK compat. v5.1 `. Find a newer unarchiving tool with a command line interface (CLI).)_

The script reads in `rockyou.txt` and tries each line as the password for `zippy.zip`.

The zip archive decrypts with the password `monkey`.

I can't remember what the flag is for this challenge 🤷‍♂️.

(The password could also be discovered by using `zip2john` to generate a hash that can then be cracked by John the Ripper (JTR))

# Broken Windows (Easy) 

We were given a Windows Group Policy Object and told to extract the flag.

When I looked at the XML I saw a field called `cpassword`, given this is a Crypto challenge this seemed like a good starting point.

```plain
cpassword="ROYKcCqVNQ+QFlpphdMOOaTURT4h1WGcBR4BmH/nCVO1wl+Q5nXKkUdjE3xs0FhB3Fa5hJPcCmkTPNC6phCW1g"
```

I Googled something like "cracking password windows gpo" and found a Rapid7 blog post from 2016 by Bill Harshbarger titled "[Pentesting in the Real World: Group Policy Pwnage](https://blog.rapid7.com/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/)".

Skim reading this I discovered there's a tool called [`gpp-decrypt`](https://github.com/BustedSec/gpp-decrypt).

We can pass the value of `cpassword` to `gpp-decrypt` and we get the flag. 

```bash
gpp-decrypt ROYKcCqVNQ+QFlpphdMOOaTURT4h1WGcBR4BmH/nCVO1wl+Q5nXKkUdjE3xs0FhB3Fa5hJPcCmkTPNC6phCW1g
```

This gets us the flag: `dhctf{cpassword_suck555}`.

## Onion (Medium)

```binary
01001110 01101010 01010001 01100111 01001110 01101010 01100111 01100111 01001110 01101010 01001101 01100111 01001110 01111010 01010001 01100111 01001110 01101010 01011001 01100111 01001110 00110010 01001001 01100111 01001110 01101101 01010001 01100111 01001110 01101010 01000101 01100111 01001110 01101101 01010101 01100111 01001110 01111010 01101011 01100111 01001110 01010111 01011001 01100111 01001110 01101010 01011001 01100111 01001110 01101101 01011001 01100111 01001110 01111010 01001001 01100111 01001110 01101101 01010001 01100111 01001110 01111010 01001101 01100111 01001110 01010111 01011001 01100111 01001101 01111010 01000001 01100111 01001110 01101101 01010101 01100111 01001110 01101010 01010101 01100111 01001110 01010111 01011001 01100111 01001110 01101010 01011001 01100111 01001110 01101101 01001101 01100111 01001101 01111010 01010001 01100111 01001110 01101010 01100011 01100111 01001110 00110010 01010001 00111101
```
We're presented with the above and told that it's encoded in many layers. 

I cheated slightly...

The above data consists only of ones (1s) and zeros (0s) so its likely binary data (Base2). I pasted it in to CyberChef and tried the [From Binary operation](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space')&input=MDEwMDExMTAgMDExMDEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAxMTEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDExMDEwMTEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMDAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAwMTEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMTAwMDEgMDAxMTExMDE).  

The result is the following string: 
```base64
NjQgNjggNjMgNzQgNjYgN2IgNmQgNjEgNmUgNzkgNWYgNjYgNmYgNzIgNmQgNzMgNWYgMzAgNmUgNjUgNWYgNjYgNmMgMzQgNjcgN2Q=
```

Based on the equals sign (`=`) at the end of the string I assumed this is Base64 encoded.

CyberChef lets us pipe the output of one operation into another. I added the [From Base64 operation](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space')From_Base64('A-Za-z0-9%2B/%3D',true)&input=MDEwMDExMTAgMDExMDEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAxMTEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDExMDEwMTEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMDAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAwMTEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMTAwMDEgMDAxMTExMDE).

This got me:
```hex
64 68 63 74 66 7b 6d 61 6e 79 5f 66 6f 72 6d 73 5f 30 6e 65 5f 66 6c 34 67 7d
```

The string consists of groups of two characters with numbers and letters where the letters never go above `f`, this look like hexadecimal (hex) encoding.

I added the [From Hex operation](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space')From_Base64('A-Za-z0-9%2B/%3D',true)From_Hex('Auto')&input=MDEwMDExMTAgMDExMDEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAxMTEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDExMDEwMTEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMDAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAwMTEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMTAwMDEgMDAxMTExMDE) to the CyberChef recipe.

This gave me the flag: `dhctf{many_forms_0ne_fl4g}`.

I mentioned above that I cheated slightly. CyberChef has an operation called Magic which "attempts to detect various properties of the input data and suggests which operations could help to make more sense of it."

I used the [Magic operation](https://gchq.github.io/CyberChef/#recipe=Magic(3,false,false,'')&input=MDEwMDExMTAgMDExMDEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAxMTEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMDAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDExMDEwMTEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMTEwMTAgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMDAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTAxMDEgMDExMDAxMTEgMDEwMDExMTAgMDEwMTAxMTEgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDEwMTEwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDExMDEgMDEwMDExMDEgMDExMDAxMTEgMDEwMDExMDEgMDExMTEwMTAgMDEwMTAwMDEgMDExMDAxMTEgMDEwMDExMTAgMDExMDEwMTAgMDExMDAwMTEgMDExMDAxMTEgMDEwMDExMTAgMDAxMTAwMTAgMDEwMTAwMDEgMDAxMTExMDE) on the initial binary data and got the flag straight away.

## Major Decode 5 (Hard)

```plain
dhctf{f3f95ee1a0058b4fffe1019b9c5da015_dc5d6e5c0ffd6d1cd249286ced098382_f4842dcb685d490e2a43212b8072a6fe_1bc29b36f623ba82aaf6724fd3b16718}
```

This challenge gives us the flag but informs us that it's been hashed. 

I assumed that each underscore (`_`) separated hash was an individual word. I then noticed that the hashes are likely MD5. MD5 produces a hash which is 128-bits (16 bytes) in length; the above hashes are the right length to be MD5. 

To test this I Google'd the first hash (`f3f95ee1a0058b4fffe1019b9c5da015`).

![no-alignment](/img/posts/ctf-crypto/md5-google.png)

The hash resolved to an actual string, `DMU`, which looks like it should be part of the flag. Since MD5 is very cheap to compute the Internet is full of sites that have hashed random strings and published the mapping of plaintext > MD5 hash.

Then it was a case of using [Hash Toolkit](https://hashtoolkit.com/decrypt-md5-hash/) to find the plaintext of the other hashes.

Flag: `dhctf{DMU_hackers_vs_md5}`.

![center-aligned-image](/img/dogs/dog3.jpg){: .align-center alt="Alaskan Malamute staring into camera"}
Photo by <a href="https://unsplash.com/@reskp">Jametlene Reskp</a> on <a href="https://unsplash.com/photos/four-assorted-color-puppies-on-window-VDrErQEF9e4">Unsplash</a>


