---
layout: post
title: Unbranding a Huawei ME906s WWAN/4G module
tags: me906s lt4132 reverse-engineering linux
---
(or rebranding it)

<!--more-->

### **Before you ask:** both the X1 Yoga and the X1 Carbon have NO ANTENNA, unless you have the 4G version. The SIM slot should work fine on all models (but it's not my fault if it doesn't)

Around one year ago I bought a new laptop, a Lenovo ThinkPad X1 Yoga (1st gen). To save some money I bought it on eBay; it was not the top model with the 4G module I wanted, but I thought I may live without it anyway.

Some weeks ago I was wandering through Lenovo's [parts lookup](https://support.lenovo.com/it/en/partslookup) website and noticed my laptop supports two WWAN modules:

- Sierra EM7455
- Huawei ME906S

I decided to look one up on eBay and see how much it costs. I found the Huawei module for 35€. Without reading too much, I bought it.

I received it today and realized that:

- My laptop has no WWAN antenna.
- The module I received is **HP-BRANDED**. Which means it's not in Lenovo's whitelist.

I still haven't solved the antenna issue - I'll probably buy a cheap-ass one and tape it on top of the battery. Hopefully signal will be good enough.

> **UPDATE**: I ended up buying [this one](https://www.ebay.it/itm/2pcs-IPEX-MHF4-Antenna-for-NGFF-M-2-ME906E-ME906S-158-EM7345-EM7455-7260-7265/272694663387) from eBay. Signal is not the best I could get but it works perfectly.

I did, though, solve the whitelist issue.

## Information gathering

First thing I did was look for info in the Internet.
I found this [GitHub page](https://github.com/abcdw/configs/blob/master/x1carbon5.org#wwan) I read while waiting for my module, which explains how [@abcdw](https://github.com/abcdw) managed to make his *Sierra* module work.

It didn't help, the modules are too different. Then I found an [official firmware upgrade](https://support.lenovo.com/it/en/downloads/ds118646) from Lenovo.

I hot-plugged the module and tried to run the updater on Windows. No such luck: the module wasn't recognized.

I tried in way too many ways to extract the binaries from the executable. Usually either `7zip`, `RAR`, or `cabextract` work.
Not this time. I ran the executables in a virtual machine, gathered the self-extracted files from Windows' temporary files (hint: type `%localappdata%` in Explorer's path box), then switched back to GNU/Linux and used `binwalk` to analyze them.

A very interesting string came to my eyes multiple times: `balong_modem.bin`. I tried to look it up online and found interesting stuff, though nothing applied to my case.
I found [balongflash](https://github.com/forth32/balongflash) and [balong-usbdload](https://github.com/forth32/balong-usbdload) + a bunch of articles that explained how to use them to flash some external USB modem by shorting some pins, assuming users already knew where to find the firmware images. _Nope._

After trying every possible query containing `balong`, I gave up and started to look for `HP lt4132 LTE/HSPA+ 4G Module`, the name the device reported itself as.  I found this [interesting article](https://toreanderson.github.io/2017/07/31/huawei-me906s-hp-lt4132-linux-ipv6.html) which definitely didn't help me unbrand my module, but it did help me ensure it was, _somehow_, working.

## The solution

After reading the article, I stumbled upon [this](http://ftp.hp.com/pub/softpaq/sp79501-80000/sp79601.html): the HP-branded firmware. I tried to download and analyze it (to download it, change `html` to `exe`). The executables looked very similar: very similar size, same icon, similar filename, similar `binwalk`s.

At some point, a software I read of in a magazine a while back came to my mind: [Resource Hacker](http://www.angusj.com/resourcehacker/#download). It's a closed-source, freeware program to extract resources embedded in Windows executables. I ran it and compared Lenovo's and HP's update tools:

HP on the left - Lenovo on the right
![Configuration file]({{site.baseurl}}/images/unbranding-me906s/shot-2018-04-28_00-40-17.png)
![Device identification XML]({{site.baseurl}}/images/unbranding-me906s/shot-2018-04-28_00-41-43.png)

Everything is identical, except for these two files (and a huge firmware blob). Looking closely at the XML you can notice the only difference between Lenovo and HP is that HP added two lines containing its rebranded device names _(read: the installer only checks the device's name, not the IDs)_.

I made some backups of both update binaries, opened Lenovo's updater with Resource Hacker and swapped the XML with HP's.

***BOOM***. Unbranded.

## Do it yourself

- **Note 1:** it will likely **not** work on VirtualBox. You can, however, use it to see if the updater recognizes the device: remove any *catchall* USB filter from VBox and select the module; the updater will try to reboot the device into download mode, but VirtualBox won't automatically reconnect it back to the VM, so nothing will happen.
- **Note 2:** keep a few copies of the patched executable. If flashing fails, delete the copy you used and use another. The updater writes something somewhere in it's own directory and makes sure subsequent flashes fail.
- **Note 3:** you'll have to make the executables yourself. Redistributing them is illegal.
- **Note 4:** the device is blacklisted in non-HP devices. To bypass the blacklist, plug it in while the computer is on or in standby, otherwise it will refuse to boot.
- **Note 5:** if you have an HP device, unbranding it will probably get you in the blacklist. You may be able to use HP's updater to rebrand it (see below).
- **Note 6:** You take all the responsibility for what you do with your hardware. *It worked on my machine*, it's not my responsibility if it doesn't work on yours.

1. Download the updaters from both [Lenovo](https://support.lenovo.com/it/en/downloads/ds118646) and [HP](http://ftp.hp.com/pub/softpaq/sp79501-80000/sp79601.exe).
1. Run each installer and find where files are extracted. You'll find one file. Copy it somewhere else.
1. Run these new files too, one at a time. Go to `%localappdata%\Temp`, you should find a directory named `XXXX.tmp`, copy it somewhere else.
1. In this directory you'll find `UpdateWizard.exe`. It contains everything we need. You'll also find some drivers, they might be helpful. Install them.
1. Using Resource Hacker (it doesn't work well on Wine, use Windows) find the XML in `STRINGS` in the Lenovo updater. Replace it with the one from HP's `UpdateWizard.exe`.
1. Run the patched updater and... you're done.

![Flashing firmware]({{site.baseurl}}/images/unbranding-me906s/windows1.png)
![Upload completed]({{site.baseurl}}/images/unbranding-me906s/windows2.png)

## Update (Jun 2019) - Rebranding

User *bubusan* sent me an email earlier this week reporting that he successfully re-branded a Huawei-branded ME906s to HP.

He said he had to take the XML from the Lenovo updater and place it into the HP updater.

I haven't tested it myself. As always, your mileage may vary.

*Thank you bubusan for your feedback! :)*
