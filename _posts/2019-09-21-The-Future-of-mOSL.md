---
title:		"The Future of mOSL"
subtitle:	"It's time to get schwifty"
excerpt:	"TL;DR: mOSL will be rewritten in Swift and the Bash version will be deprecated."
share-img:	"/img/posts/schwifty0.png"
thumbnail-img:		"/img/posts/schwifty0.jpg"
tags:		[Admin]
---

## TL;DR

[mOSL](https://github.com/0xmachos/mOSL) will be rewritten in [Swift](https://developer.apple.com/swift/) and the Bash version will be deprecated.

# Why?

Below I'll briefly cover why a rewrite is needed, some of the options I've look at, why I rejected them and why I settled on Swift. 

Before we get into it though its important to understand the following: One of my main goals with mOSL is **no dependencies**. Anyone should be able to run it on a fresh install of macOS.

## Architecture

mOSL currently supports 21 settings. Internally this translates to 21 functions all of which have a `title`, `audit_command` and `fix_command` local variable.

This is the function to [disable remote login](https://github.com/0xmachos/mOSL/blob/acb2971c0f1e36fb347c8e40545889995ab59920/Lockdown#L608-L621): 

```bash
function disable_remote_login {

  local mode=${1:?No mode passed}
  local title
  local audit_command
  local fix_command
  
  title="Disable Remote Login"

  audit_command="sudo systemsetup -getremotelogin | grep -q 'Remote Login: Off'"
  fix_command="sudo systemsetup -f -setremotelogin off >/dev/null"

  mode_check "${mode}" "${title}" "${audit_command}" "${fix_command}"
}
```

This is barbaric.

The correct way to do this would be to have a configuration file in YAML/JSON/etc and parse that. This is how it was [implemented](https://bitbucket.org/objective-see/lockdown/src/master/Lockdown/3rd-Party/SummitRoute/commands.yaml) in [Patrick Wardle](https://twitter.com/patrickwardle)'s [Lockdown](https://objective-see.com/products/lockdown.html) tool. He embedded the `commands.yaml` from [osxlockdown](https://github.com/SummitRoute/osxlockdown) and parsed it.

It's not easy to parse a configuration file in Bash without using some seriously [hardcore sed/awk](https://gist.github.com/briantjacobs/7753bf850ca5e39be409) or a dependency like [jq](https://stedolan.github.io/jq/).

I had [hoped to address this](https://github.com/0xmachos/mOSL/issues/9) in v2.0 but never got round to it.

## Bash/ Zsh

With macOS Catalina the default shell will be changed from Bash to Zsh. Bash will still be present but at some point I imagine that Apple will remove the, over 10 year old, version of Bash.

Zsh suffers from the same limitations as Bash mentioned in the Architecture section above. In addition the static analysis tool I use, [Shellcheck](https://www.shellcheck.net) doesn't support Zsh. 

## Scripting Language Deprecation

The [Catalina Beta Release Notes](https://developer.apple.com/documentation/macos_release_notes/macos_catalina_10_15_beta_8_release_notes) have a "Scripting Language Runtimes" section which states:

> Scripting language runtimes such as Python, Ruby, and Perl are included in macOS for compatibility with legacy software. Future versions of macOS won’t include scripting language runtimes by default, and might require you to install additional packages. If your software depends on scripting languages, it’s recommended that you bundle the runtime within the app. 

Essentially, future versions of macOS won't come bundled with Python, Ruby or Perl. So if macOS wont be shipping with these runtimes I can't rewrite mOSL in those languages.

## Code Signing

Currently the executable component of mOSL, [Lockdown](), is signed using [Minisign](https://jedisct1.github.io/minisign/) by [Frank Denis](https://twitter.com/jedisct1). This only offers opportunistic verification as Minisign is a dependency. I expect that most users don't have it installed and won't install it to verify one tool.

You can actually [code sign a Bash script](https://carlashley.com/2018/09/23/code-signing-scripts-for-pppc-whitelisting/) (or any text file) using `codesign`. However the signature is attached to the file as an Extended Attribute (XA) and unfortunately Git doesn't track XAs.

With a fully fledged macOS Application/ Command Line Tool I can properly code sign/ notarise mOSL with my Apple Developer certificate. This is a massive win. 

## More Power

Currently mOSL is limited to settings which can be changed via system binaries. I've not researched this yet but I suspect that using system APIs will enable mOSL to audit and fix more settings.

## Conclusion

Swift 5, [released earlier this year](https://swift.org/blog/swift-5-released/) is now Application Binary Interface (ABI) stable and as a result the runtime is now bundled with every Apple operating system (OS). This means that mOSL written in Swift will run on fresh installs of macOS Catalina and later.

Finally, I'd like to thank [Richie Cyrus](https://twitter.com/rrcyrus) for taking the time to chat to me about his experience with porting his tool [Venator](https://github.com/richiercyrus/Venator) from Python to Swift. 

![center-aligned-image](/img/dogs/dog1.jpg){: .align-center alt="Wolf looking dog with tongue out"}
Photo by <a href="https://unsplash.com/@justinveenema">Justin Veenema</a> on <a href="https://unsplash.com/photos/brown-and-white-siberian-husky-standing-near-river-UlmPSBvHTY0">Unsplash</a>
  
