+++
date = "2016-09-15T16:09:36-07:00"
draft = false
tags = ["yubikey", "ssh", "keybase", "keybase.io"]
title = "Yubikey, Keybase and SSH"

+++

## History

About a year ago, I purchased a [Yubikey
NEO](https://www.yubico.com/products/yubikey-hardware/yubikey-neo/),
a hardware-based two-factor authentication keyfob. I'd had an earlier version
that could generate tokens upon being pressed, but the NEO included a few
things I wanted to test out, including the ability to store gpg keys on it.
I was also curious to see how hard it'd be to use the Yubikey with SSH
connections. I went through the entire setup a year ago, got frustrated with
using GPGTools on OS X and the keychain, and then promptly forgot about it for
a year until my key expired and I'd forgotten the passphrase I'd use to
encrypt it. I've just recently rectified that and went through the whole
process again, so I thought I'd document it here for posterity.

## Setup

### Generating the keys offline

I followed [these
instructions](https://blog.josefsson.org/2014/06/23/offline-gnupg-master-key-and-subkeys-on-yubikey-neo-smartcard/) by Simon Josefsson to create a private key on an offline machine, and then subkeys that I could transfer to my Yubikey for daily use.

Note here that I downloaded a recent ISO installer for Mint Linux. I'm booting
it on a 2013 MacBook Air, and the Broadcom drivers are non-free, and don't come
bundled with recent versions of Debian that I've seen.

Simon argues for a short key validity period[^1], and I tend to agree with him.
If I somehow lose my keys and can't revoke them, they're only good for at most
another 100 days. Minimizing blast radius sounds good to me.

### SSH Setup

[This](https://www.rempe.us/blog/yubikey-gnupg-2-1-and-ssh/) helped me get my
keys usable for SSH in OS X. Like Glenn, the biggest issue I saw while using
gpg-agent with my Yubikey was that it wouldn't recognize my reinserted card
until I killed and restarted it. Glenn suggests using ControlPlane for
listening for the USB device to be inserted/removed. As I'm already using the
awesome [Hammerspoon](http://www.hammerspoon.org/) for scripting a lot of
things like window resizing and shortcut keys, I added a script for that to my
dotfiles[^2]. Now whenever I insert the Yubikey, gpg-agent is restarted. As an
added bonus, removing it automatically starts my screensaver and locks the
display.

## Keybase and some final thoughts

I started this whole mess because I received an invite to Keybase
([here](https://keybase.io/steeef)'s my profile if you're curious). I'm not too
keen on their suggestion to upload my private key to their servers, so I'm
keeping it on the Yubikey for now. I haven't had too much time to use it, but
it's an interesting idea that I'll be watching.

One thing that bugs me about the Yubikey NEO is that it only supports RSA keys
of up to 2048 bits in length. The newer [Yubikey
4](https://www.yubico.com/products/yubikey-hardware/yubikey4/) will support
4096, which is what I'm used to using with file-based keys. Plus, there've been
some speed and security enhancements I'd like to take advantage of. I'd lose
out on NFC, but since I've got an iPhone now, I can't use it anyway.

[^1]: https://blog.josefsson.org/2014/08/26/the-case-for-short-openpgp-key-validity-periods/
[^2]: My Hammerspoon scripts are here: https://github.com/steeef/dotfiles/tree/master/hammerspoon.
