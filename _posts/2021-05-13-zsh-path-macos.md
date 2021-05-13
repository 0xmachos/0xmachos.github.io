---
title:		"How $PATH is Constructed on macOS"
subtitle:	"A quick fix that escalated into a blog post"
excerpt:	"I ran into some issues with my $PATH the other day while trying to work with Ruby. This prompted me to investigate how $PATH is constructed by Zsh/ macOS."
tags:		[macOS, Shell, Zsh]
---

(_This started out as a [Today I Learned](https://0xmachos.com/til/) (TIL) post then escalated into a full blog post._)


I ran into some issues with my `$PATH` the other day while trying to work with Ruby. 

<blockquote class="twitter-tweet" align="center"><p lang="en" dir="ltr">Am I being an idiot or is impossible to get paths to the front of your <a href="https://twitter.com/search?q=%24PATH&amp;src=ctag&amp;ref_src=twsrc%5Etfw">$PATH</a> on Big Sur because of /usr/libexec/path_helper? <br><br>How do you get e.g. /usr/local/opt/ruby/bin to the front of your path? When I add it to .zshrc path_helper comes along and fucks them to the end</p>&mdash; mikey (@0xmachos) <a href="https://twitter.com/0xmachos/status/1392139097657991170">May 11, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I put all the code building my `$PATH` in `.zshenv` but for some reason I wasn't able to get my custom paths to the beginning of `$PATH`. 

This is an issue as macOS ships with a version of Ruby at `/usr/bin/ruby`, so any invocation of `ruby` is going to call the macOS version not the version I've installed via `brew`. 

**Note:** At some point in the future Apple will [remove scripting language runtimes](https://tidbits.com/2019/06/25/apple-to-deprecate-scripting-languages-in-future-versions-of-macos/).

This prompted me to investigate how `$PATH` is constructed by Zsh/ macOS. 

I found [Armin Briegel's](https://twitter.com/titanonearth) book ["Moving to zsh"](https://scriptingosx.com/2019/10/new-book-moving-to-zsh/) extremely helpful while writing this post. 

# TL;DR

Put all code which constructs your `$PATH` into `.zshrc`. 

`/etc/zprofile` is sourced before `.zshrc`, so any changes you make to `$PATH` won't be reordered by `path_helper`. 

If you put code that builds `$PATH` in `.zshenv` it will be reordered by `path_helper` because `/etc/zprofile` is sourced after it. Only use `.zshenv` for setting environment variables that are not `$PATH`. 

The order of files sourced by Zsh when it starts up:
1. `$HOME/.zshenv`
2. `/etc/zprofile`
3. `/etc/zshrc`
4. `/etc/zshrc_Apple_Terminal`
4. `$HOME/.zshrc`


# `/etc/zprofile`

["Moving to zsh"](https://scriptingosx.com/2019/10/new-book-moving-to-zsh/) page 76 informs us that on macOS Zsh sources `/etc/zprofile` and `/etc/zshrc`.

`/etc/zshrc` contains code that enables UTF-8 support and configures some other options which aren't relevant to this post. 

`/etc/zprofile` on the other hand, is very relevant. As of macOS `11.3.1`, it contains the following code:
```shell
if [ -x /usr/libexec/path_helper ]; then
	eval `/usr/libexec/path_helper -s`
fi
```


# `path_helper`

`man path_helper` gives us a good idea of what `path_helper` does.

> path_helper -- helper for constructing PATH environment variable

> The path_helper utility reads the contents of the files in the directories /etc/paths.d and /etc/manpaths.d and appends their contents to the PATH and MANPATH environment variables respectively. 

## How does `path_helper` construct `$PATH`?

Before any setup happens the default `$PATH` is 
```shell
PATH=/usr/bin:/bin
```

You can see this yourself by adding `echo $PATH` to the beginning of `/etc/zprofile` (requires r00t) and opening a new Terminal tab.

From this base, `path_helper` looks to `/etc/paths` to construct a sort of "extended default" `$PATH`. `/etc/paths` contains:
```shell
/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin
```

So, at this point `$PATH` looks like: 
```shell
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:
```

Next `path_helper` iterates through the files in `/etc/paths.d/` and appends them to the default `$PATH`.

My `/etc/paths.d/` looks like:

```shell
$ ls /etc/paths.d
100-rvictl* MacGPG2     TeX
```

According to the `path_helper` man page:
>  Files in these directories should contain one path element per line.

As an example, this is what `/etc/paths.d/MacGPG2` contains:
```shell
$ cat /etc/paths.d/MacGPG2
/usr/local/MacGPG2/bin
```

After `path_helper` has iterated over these files my `$PATH` looks like
```shell
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin:
```

That's how `$PATH`, consisting of the system defaults and custom paths added by installed programs, is constructed.

How does Zsh/ macOS handle custom user alterations to `$PATH` coded in files like `.zshrc` & `.zshenv`? Time to do some debugging...

<figure align="center">
	<img src="/img/posts/here-we-go-again.png"/>
	<figcaption>✨Debugging✨</figcaption>
</figure>

## How to Debug This

### `set -x`

The `set` Builtin can help us out here, specifically `set -x`

Though I'm using Zsh the behavior is the same as Bash, as far as I'm aware, so we can consult the GNU bash manual. ["The Set Builtin"](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html) article states:

> Print a trace of simple commands, for commands, case commands, select commands, and arithmetic for commands and their arguments or associated word lists after they are expanded and before they are executed.

For example consider the following code saved in a file called `test`.
```shell
#!/usr/bin/env zsh
set -x
string0="Hello"
string1=", World!"
echo "This a ${string0}${string1} Example."
set +x
```

When executed, `./test`, it prints the following to `stdout`:
```shell
+./test:3> string0=Hello 
+./test:4> string1=', World!' 
+./test:5> echo 'This a Hello, World! Example.'
This a Hello, World! Example.
+./test:6> set +x
```

We can see how each variable is assigned and how they're expanded into `echo` as well as the final output of `echo`. 

(Counterintuitive, `set -` enables an option while `set +` disables it 🤷‍♂️)


### Debugging `.zshenv`

Consider a `.zshenv` file which contains the following:
```shell
set -x

if [[ -x "/usr/local/opt/ruby/bin/ruby" ]]; then
  export PATH="/usr/local/opt/ruby/bin:$PATH"
fi

set+
```

When we open a new Terminal tab the following is printed to `stdout`:
```shell
+/Users/mikey/.zshenv:7> [[ -x /usr/local/opt/ruby/bin/ruby ]]
+/Users/mikey/.zshenv:8> export PATH=/usr/local/opt/ruby/bin:/usr/bin:/bin
+/Users/mikey/.zshenv:13> set +x
```

This shows that `/usr/local/opt/ruby/bin` is at the start of `$PATH`.


However executing `echo $PATH` produces the following:
```shell
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin:/usr/local/opt/ruby/bin
```

You can see that `/usr/local/opt/ruby/bin` is now at the end of `$PATH`. So something must be reordering `$PATH` after `.zshenv` is sourced.

If we move `set +x` out of `.zshenv` and place it at the end of `.zshrc` then open a new Terminal tab the following is printed to `stdout`:

```shell
+/Users/mikey/.zshenv:7> [[ -x /usr/local/opt/ruby/bin/ruby ]]
+/Users/mikey/.zshenv:8> export PATH=/usr/local/opt/ruby/bin:/usr/bin:/bin
+/etc/zprofile:6> [ -x /usr/libexec/path_helper ']'
+/etc/zprofile:7> /usr/libexec/path_helper -s
+/etc/zprofile:7> eval 'PATH="/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin:/usr/local/opt/ruby/bin";' export 'PATH;'
+(eval):1> PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin:/usr/local/opt/ruby/bin 
+(eval):1> export PATH
```

We can see that `/etc/zprofile` is sourced after `.zshenv` which executes `/usr/libexec/path_helper` which is the reason `$PATH` has been reordered.

### Debugging `.zshrc`

Consider the following code added to the end of `.zshrc`:
```shell
echo $PATH
set -x

if [[ -x "/usr/local/opt/ruby/bin/ruby" ]]; then
  export PATH="/usr/local/opt/ruby/bin:$PATH"
fi
echo $PATH
```

If we open a new Terminal tab the following is printed to `stdout`:
```shell
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin
+/Users/mikey/.zshrc:151> [[ -x /usr/local/opt/ruby/bin/ruby ]]
+/Users/mikey/.zshrc:152> export PATH=/usr/local/opt/ruby/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin
+/Users/mikey/.zshrc:155> echo /usr/local/opt/ruby/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin
/usr/local/opt/ruby/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/Library/Apple/usr/bin
```

We can see from the first `echo $PATH` that `$PATH` has already been constructed by `/etc/zprofile`/`/usr/libexec/path_helper`. Since `.zshrc` is sourced after `/etc/zprofile` the modification of `$PATH` by `export PATH="/usr/local/opt/ruby/bin:$PATH"` is not affected. You can see this in the final line of the output which has `/usr/local/opt/ruby/bin` as the first path in `$PATH`.

# Conclusion

Put all code which constructs your `$PATH` into `.zshrc`. Only use `.zshenv` for setting environment variables that are not `$PATH`.

The order of files sourced by Zsh when it starts up:
1. `$HOME/.zshenv`
2. `/etc/zprofile`
3. `/etc/zshrc`
4. `/etc/zshrc_Apple_Terminal`
4. `$HOME/.zshrc`

I highly recommended [Armin Briegel's](https://twitter.com/titanonearth) book ["Moving to zsh"](https://scriptingosx.com/2019/10/new-book-moving-to-zsh/) and his [website](https://scriptingosx.com/)/ [newsletter](https://tinyletter.com/scriptingosx) Scripting OS X.


<figure align="center">
	<img src="/img/dogs/dog5.jpg"/>
	<figcaption>Photo by <a href="https://unsplash.com/@laulaco">Regine Tholen</a> on <a href="https://unsplash.com">Unsplash</a></figcaption>
</figure>


