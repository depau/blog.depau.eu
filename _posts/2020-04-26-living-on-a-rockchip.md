---
layout: post
title: Living on a Rockchip
tags: pinebookpro archlinux wayland sway linux 
---

I've been using my Pinebook Pro on a daily basis for a month or so. This is my experience report.

<!--more-->

### Note:

The name of the article was shamelessly stolen from Alyssa Rosenzweig's blog post [Living on a Rockchip](https://rosenzweig.io/blog/living-on-a-rockchip.html), which you should read if you haven't. She's the mind behind Panfrost, the Free and Open-Source driver for some MALI GPUs. It is running on my Pinebook as I'm typing this post. Thanks to Alyssa and to all the other Panfrost contributors for writing this great piece of FOSS.

![My Pinebook Pro and my ThinkPad]({{ site.baseurl }}/images/2020-04-26-living-on-a-rockchip/img1.jpg)
My Pinebook Pro and my ThinkPad

----

So, the reason why I switched to using it on a daily basis is pretty much because... I bought it?

I must say I do regret buying it, it's just that I happened to be on the Pine64 store and my finger slipped on the "Purchase" button, then I tripped, fell on the computer and accidentally typed the full credit card data and completed the purchase. It was definitely an *accident*. /s

I switched to it because now I'm stuck with it. I didn't really need it but now it's too late to cancel the order, so I might as well just use it.

More seriously, I think it's actually a great device. You can find plenty of reviews online — YouTube keeps spamming me with them, it hasn't realized I don't really wanna watch them — but here's mine.


## Bad or *sort of* bad things
### Wi-Fi/Bluetooth

Horrible. But anyway it's from Broadcom, so 🤷.

Plus it requires binary blobs and it might be vulnerable to [Broadpwn](https://blog.exodusintel.com/2017/07/26/broadpwn/).

Do I need to say anything else? It does connect to Wi-Fi networks and Bluetooth devices, so I guess the chip does what it was placed in there for.

### Keyboard

It's not too bad, really. It's not high-quality, the Tab key gets stuck inside way more often than I'd want it to, but typing on it feels very nice.

I feel that the Ctrl key might be a little too far away, but it's probably just me since my ThinkPad has the Fn and Ctrl keys swapped.

Some keys such as Page up/down, Home/End, Delete are Fn keys. You get used to it, it's not terrible. I think MacBook keyboards are waaaay worse than this.

One thing I hate though is that the power button is where I'd expect the Delete key to be. I had to disable it since I'd constantly turn it off while typing.

### Touchpad

Absolutely terrible. Though some [reverse engineering](https://github.com/jackhumbert/pinebook-pro-keyboard-updater#revised-firmware) is going on to provide a custom firmware which actually does improve it, the touchpad is still quite bad.

First of all, the clicking sound it makes is super loud and it requires a considerable amount of force to press.

Then, it feels quite inaccurate. It's quite hard to move the pointer to a specific, tiny location on the screen, I feel like its resolution is way too low.

I'm hoping the reverse engineering goes along nicely, I don't think I'm going to try contributing just yet.

### Sound

I'll just say that if you raise too much the volume, the speakers might start drawing too much power and USB devices will disconnect.

On the input side, there's a constant noise on the internal microphone and there is no external microphone input. The headphone jack can't be used with microphone earphones. I had to use an external sound card.

Also, there's additional louder noise if it's charging and you also touch the laptop.

Not good.

## Good things
### Screen

Screen is great. I love it, thin edges, I'm not a color-profile nerd but the colors look great to me, and there's only one dead pixel.

### CPU

I'm loving the Rockchip RK3399. It's fast, seriously. It's 2x 2GHz cores, 4x 1.5GHz. Compared to my Intel-based ThinkPad, with 2 cores — 4 threads,  3GHz max, it actually feels faster.

That one extra GHz is handy with single-thread workloads, but 6 cores are very good for multitasking.

Today I held a class on [Nginx](https://slides.poul.org/2020/linux/Nginx/) for [POuL](https://poul.org), using the Pinebook.

Here we're all still in self-isolation for COVID-19, I held it at home and we used Microsoft Teams since we haven't deployed and tested yet a proper FOSS replacement.

Well, the Pinebook was perfectly able to handle X11 grab by MS Teams on Chromium, X11 grab and encode to h264 with ffmpeg and normal browser usage, all of that with only GL hardware acceleration, no hardware codecs.

CPU was at a constant 80% (I haven't checked the load avg) but it was quite usable, the recording came out fine. But most importantly, it didn't heat up much, even with no active cooling. It was quite warm, but I can assure my ThinkPad would have been on fire by that time.


## Freedom (almost)

U-Boot is (almost) mainline, and as far as I know it doesn't need any blobs since RK3399 is fully supported.

The only binary blobs I need are:

- Broadcom Wi-Fi firmware
- Some `rockchip-dp` firmware. I don't know what's this for, I haven't checked, but it seems to work fine without it.

# My setup

Of course I run Arch Linux ARM on it. I use the almost-mainline kernel from the Manjaro guys, plus their nice device-specific packages for ALSA config and firmware.

Not much to say about my setup since I've [already written about this](https://blog.depau.eu/2020/01/06/why-i-use-arch/).

Since I do use some custom packages, I'm planning to create a custom repo with packages built by mi Drone CI instance. I'll write a blog post once it's ready.

In general, I don't think I've had to give up or replace anything, with one exception: Telegram Desktop.

Due to bugs that that seem to be caused by Qt5, it crashes all the time on ARM. I just use Telegram React on Firefox.

I use only free software for everything else, so pretty much everything is either available or it can be easily built.

Oh btw, I still use my ThinkPad to watch videos. No va-api/vdpau yet, it lags quite a bit :/

# Conclusion

It's much better than I expected, really. At least considering its price. The only things I miss from my ThinkPad are the touchscreen+Wacom digitizer, a great touchpad, the fingerprint reader and the TrackPoint.

Still, I wouldn't really recommend it. If you want a new laptop, just get a good one. If you want an upgrade from your old laptop, you can probably get a more powerful used device for less. If you want a laptop which can run fine with only free software or you just to hack on it, it might be the one for you.

However, if you just want to play with an ARM device, just get a RockPro64, Rock64 or something like that. I also got the Rock64 and I have only good things to say about it: it's cheap, it works great and it's **Broadcom-free**, which is **always** a good thing.

![Anybody has nice stickers?]({{ site.baseurl }}/images/2020-04-26-living-on-a-rockchip/img2.jpg)
