---
layout: post
title: Installing Arch Linux on WSL, the Arch Way™️
tags: archlinux wsl windows
---

My company's official OS is Windows 10. GNU/Linux is not allowed, not even Ubuntu or Red Hat. PAINFUL.

But lately, not so much, since there is Windows Subsystem for Linux.

Of course, I'm not going to want to use Ubuntu, I want **the** distribution, Arch Linux.

<!--more-->

### Update

- I switched jobs, I don't have to use Windows anymore, and I can now happily do my work on Arch Linux. Natively.
- I'm not a Windows expert, nor a WSL expert. I have done my own research and my findings and work are shared here.
  If you ask me *"how do I do this Windows/WSL-specific thing"* and it is not already mentioned here, I probably have no clue.
- I only know the major differences between WSL1 and WSL2 and I can't give you advice. It's proprietary software, I have no idea. Ask Bill. 
- If you ask me whether you should use Arch on WSL or natively, I have an answer for you: delete Windows now.


![My work computer desktop]({{ site.baseurl }}/images/2020-03-05-installing-arch-wsl-manually/screenshot.jpg)
My work computer desktop

The only way to install it seems following [this guide](https://github.com/yuk7/ArchWSL), but even though giving a quick look at the code everything seems rather okay, there's a *magic* prebuilt rootfs (and no generation scripts) and I don't really want to inspect its contents thoroughly to see if there's anything malicious.

So, I tried to find a way to install it manually from a rootfs downloaded from [archlinux.org](https://archlinux.org).

**Note:** another machine already running a GNU/Linux distro natively, preferrably Arch Linux, is required.

~~**Note 2:** I'm talking about WSL1. I do not know if this will work on WSL2, I haven't upgraded yet.~~

**Update 2021-05-01:** I was eventually forced to upgrade to WSL2, with a few additional steps everything still works fine.

See the WSL2 section at the end.

## Fixing the rootfs

Rootfs's downloaded from Arch mirrors contain the actual root filesystem under `/root.x86_64`.

We have to fix that, because when Windows extracts it it expects the root of the tar to be the root of the system

So, on the native Arch machine:

```
fakeroot -- bash -c 'bsdtar -xf [path/to/arch-rootfs.tar] && \
bsdtar -czaf arch-wsl.tar.gz root.x86_64/*'
```

Copy the new rootfs to the Windows machine.

## Installing the rootfs

Pick a suitable location for your WSL installation, such as `\Users\%username%\WSL\ArchLinux`. Picking a handy location makes it easy to copy files **from** WSL to somewhere else.

**Note:** apparently, you can only copy files **from** WSL and **not to**. Either they won't show up in WSL at all, giving out 'Input/output error' on access, or it will crash Windows.

Then run in the command prompt:

```
wsl --import DistroName \path\to\WSL\dir \path\to\arch-wsl.tar.gz
```

It looks like paths with spaces won't work with this tool on the command line, so make sure you `cd` into the path and use relative paths from there if your path has spaces.

Anyway, that's about it. You can already run `wsl` or `bash` with no arguments to get into your WSL shell.

There's a few catches, however, so go on.

## First Arch setup

- Initialize the pacman keyring

  ```
  pacman-key --init
  pacman-key --populate archlinux
  ```
  Run again if it fails.

- Pick a mirror in `/etc/pacman.d/mirrorlist`
- Set up timezone, localization (see 
  [Arch installation guide](https://wiki.archlinux.org/index.php/installation_guide#Time_zone))
- Add a DNS server to `/etc/resolv.conf`
- `pacman -Syu`


## Making AUR packages

The system is almost ready, except... you can't make any package.

That's because unless you're running WSL 2 which runs an actual Linux kernel, Microsoft hasn't implemented Unix SYSV IPC, which is used by `fakeroot`.

A workaround is to use `fakeroot-tcp` from AUR which uses TCP/IP sockets instead of Unix IPC, the problem is that in order to make it you need `fakeroot`...

The easiest solution is to build the package on a real Arch machine, then send it over and install it. The a-bit-less-easy solution is to make the package (it will fail). Then you overwrite `libfakeroot.so` in `/usr/lib` with the one you just built (in `src/fakeroot-tcp`), run `makepkg -R` and it should work fine this time.

## Running GUI apps

I use [vcxsrv](https://sourceforge.net/projects/vcxsrv/). However, setting up an X server isn't enough at times.

If you're running GNOME GUI utilities, you'll likely run into the following issues:

- `gnome-keyring` is not running as SSH agent
- Programs that use `gsettings` will not run, not store settings, mysteriously crash without any errors
- Lack of theming
- Lack of inter-process communication (i.e. DBus is not running)

Over time I developed this set of scripts that will fix that for you, creating a session environment file that you will have to make sure is always loaded so programs will run properly:

[https://github.com/depau/wsl-startup](https://github.com/depau/wsl-startup)


## WSL2

*Added on 2021-05-01*

This setup works on WSL2 as well, but a few more steps are required.

First of all you have to add a Windows Defender Firewall rule to allow WSL to access Vcxsrv, as described [here](https://github.com/cascadium/wsl-windows-toolbar-launcher#firewall-rules).

Then, instead of enabling the `40_xorg_setup` wsl-startup script you will have to enable `40_xorg_setup_wsl2`, which will set the correct `DISPLAY` variable that points to the main Windows network.

More info here: [https://stackoverflow.com/a/61110604/1124621](https://stackoverflow.com/a/61110604/1124621)


# Other stuff

If you're in my same situation and you're using Arch on WSL at work just because you must use Windows, it may be possible you're also using an SSH client called SecureCRT.

If you are, you may want to check out my script [`shcrt`](https://github.com/depau/shcrt). I'm planning to rewrite it in Python using urwid for a better (and faster) experience, but I haven't started yet.

*That's it* ;)

Let me know if you found this useful. Also let me know any suggestions: ~~I have to deal with Windows' crap every day at work, if you know something that will make it less painful, send me an email!~~

**Update:** I don't have to use Windows daily anymore. Don't send me any emails :) If you have any suggestions that could be useful to other readers, be sure to add them to the comments below.
