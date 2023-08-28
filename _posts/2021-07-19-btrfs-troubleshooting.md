---
layout: post
title: Btrfs troubleshooting and tricks
tags: btrfs linux
---

Btrfs is a great filesystem and your data is safe with it, however it does come with a few shenanigans and when brought to the limits it may give you some headaches.

This post aims to provide solutions to the most common issues with Btrfs.

At the end you will also find some tricks that exploit btrfs' great functionality to speed up common tasks.

<!--more-->

---

**Since I know where this is going:** this page comes entirely from my experience.

I'm not a btrfs developer, nor I'm expert in the way it works internally. I have been, however, a user of this amazing filesystem for something like 8 years as I'm writing this, so I'm pretty sure I've hit all snags that one could possibly hit without using unstable features.

If you think something here is wrong, have any improvements or another issue that isn't listed here, feel free to add a comment or to send me a pull request.

If you have strong opinions, insults and other shit like that, please send them to `/dev/null`. Thank you.

## Foreword

Regardless of what filesystem you use, you should be backing up your important data.

If for whatever reason you lose your data, instead of blaming the filesystem, the hardware, or the OS, you should look around your house, find a corner and go cry there while thinking about your stupid life choices.

Btrfs will likely warn you of any corruption or hardware failure before most other filesystems since it checksums all data. However, you should not rely on that. Once data is lost, it is lost and you better have a plan B or a place to cry.


## Notes

Pretty much everything below needs to run as root, unless specified.

I also assume you're not completely stupid and that you are not using experimental functionality such as RAID56.

The [official Status page](https://btrfs.wiki.kernel.org/index.php/Status) is updated frequently. Give it a read.

The btrfs developers are (rightfully) quite conservative and will never tell you btrfs is 100% stable. I can tell you that in all the years that I've used this filesystem the following are the only problems I've experienced. Btrfs works great and it is one of the few filesystem that will notice early on if your hard disk has started failing. Btrfs will fail, other filesystems will silently read garbage.

# I had an issue while working and I rebooted / My system crashed

- I use Ubuntu:
    - Open the "Advanced options" in the GRUB boot menu
    - Boot into recovery
    - Open a shell
    - If the filesystem is not corrupted:
      - Remount the root filesystem in read-write mode:
        `mount -o remount,rw /`
    - You may now proceed with whatever you need to do to fix your filesystem.
- I use a systemd-based distro
  - **Arch users beware:** Arch does not include `/etc/shadow` in the initramfs by default, systemd will not let you get an early-boot shell. Once you fix it, add `/etc/shadow` to the `files=()` array in `/etc/mkinitcpio.conf` so you don't have to use a live USB next time. This DOES imply that your password's HMAC will be stored in plaintext in the initramfs, which may be a problem for some people.
  - Try a rescue shell: in the GRUB menu, press `E` to edit a boot entry. Add `1` or `systemd.unit=resque.target` at the end of the `linux` line. Remove all occurrences of `quiet` and `splash`, and maybe add `loglevel=3` to get more debugging info
  - Try an emergency shell: same as above, but add `emergency` or `systemd.unit=emergency.target` instead.
  - Try to boot directly into a shell: same as above but add `init=/bin/bash` instead. This won't work on some distros.
  - You may now proceed with whatever you need to do to fix your filesystem.
- All above fails / does not apply
  - Boot a live USB and work from there

# Filesystem full / "No space left on device"

All solutions below require the filesystem to be mounted read-write.

If you can't do that, you need to first solve any other issues that prevent you from doing so.

If your system is still running you should probably kill all processes that filled up your disk. Otherwise the moment you make some space they may fill it up immediately and give you more headaches. 

## Prevention

### For small filesystems

Small filesystem may benefit from having a shared data+metadata segment. Btrfs developers recommend to do it only for filesystems smaller than 16GB. My experience suggests this option prevents major headaches on filesystems smaller than 60GB. There are no major downsides other than potentially reduced performance.

To implement this, you need to recreate the filesystem with the `--mixed` parameter to `mkfs.btrfs`. There's no way around it.

See this [btrfs FAQ entry](https://btrfs.wiki.kernel.org/index.php/FAQ#Help.21_Btrfs_claims_I.27m_out_of_space.2C_but_it_looks_like_I_should_have_lots_left.21).

### For big filesystems

GNOME notifies you when a filesystem is almost full. Take action immediately.

On other desktops I recommend you enable some similar behavior if not provided by default.

Running your filesystem close to full is not a great idea since at best you will have degraded performance, regardless of whether it is btrfs or not. Plan your partitioning better or upgrade your hardware.

### For all filesystems

Enable compression if you don't have too many worries about performance. Compression has more benefits than downsides, and on fast machines with slow rotating storage it may even improve performance since less data has to be read/written. ZSTD compression works amazingly well.

See the [Arch wiki entry](https://wiki.archlinux.org/title/Btrfs#Compression).


## I use Docker

Docker has a btrfs back-end that stores images and volumes as btrfs subvolumes.

If it's okay for you to delete them: **(you will lose all Docker data)**

```bash
systemctl stop docker.service docker.socket  # if your system is still running-ish

# Find all Docker subvolumes and double-check for false-positives
btrfs subvolume list / | grep var/lib/docker

# Nuke them
cd /
btrfs subvolume list / | grep var/lib/docker | cut -d ' ' -f 9 | xargs btrfs subvolume delete -c
## -c means to commit the deletion at the end, once all deletions have been submitted

# Remove and recreate /var/lib/docker so it works next time you need it
rm -Rf /var/lib/docker
mkdir /var/lib/docker
```
{: .wrap-code}

## I do snapshots

You can find all snapshots in your filesystem and delete those you don't need.

```bash
btrfs subvolume list /path/to/mountpoint
btrfs subvolume delete -c /path/to/mountpoint/subvolume
```
{: .wrap-code}

## I have big files to delete but rm doesn't work

For some reason btrfs needs a tiny bit of space to delete (unlink) files. If your filesystem is stuffed like a turkey, `rm` won't work.

Try this instead:

```bash
truncate -s0 filename
# or
echo > filename
```

These will truncate the file and free up space. Once you free up enough space, you can `rm` these files.


## I have a USB drive / another disk

If you have a USB drive or another disk handy, you can plug it in, add it temporarily to your filesystem to give it some new free space.

```bash
btrfs device add /dev/XXX /path/to/mountpoint
rm /path/to/big/file
# [...]
btrfs device remove /dev/XXX /path/to/mountpoint 
```
{: .wrap-code}

## Clear free space cache

May or may not help.

```
mount -o remount,clear_cache /path/to/mountpoint
```
{: .wrap-code}

## Nothing above works

Read carefully the [btrfs FAQ entry](https://btrfs.wiki.kernel.org/index.php/FAQ#Help.21_Btrfs_claims_I.27m_out_of_space.2C_but_it_looks_like_I_should_have_lots_left.21).

You may want to copy the data temporarily to another disk as a last resort.


# Filesystem does not mount

## Unclean shutdown

The filesystem log tree may be corrupted due to the unclean shutdown.

By clearing it you may or may not fix the issue, but you will definitely lose any data written within 30 seconds before the unclean shutdown. This is usually acceptable, however.

See the [manpage for `btrfs-rescue(8)`](https://man.archlinux.org/man/core/btrfs-progs/btrfs-rescue.8.en) for more info.

```bash
btrfs rescue zero-log /dev/XXX
# man btrfs rescue - for more info
```
{: .wrap-code}

Then try mounting again.

Another potential failure may be caused by a corrupted free space cache. Try adding the `clear_cache` mount option, such as:

```bash
mount -o clear_cache,subvol=...,compress=.... /dev/XXX /mountpoint
```
{: .wrap-code}

This does not cause, as far as I know, data loss or other issues. It's usually safe.

## Hardware failure (rotational media)

As soon as you notice any signs of hardware issues on **hard disks** (i.e. you get random `Input/output error`s and the kernel log reports error about the device), hold the power button and SHUT DOWN THE SYSTEM IMMEDIATELY.

Then boot your favorite live USB Linux distro, mount the filesystem read-only and try to copy the data out.

Unfortunately there isn't a lot to do in this case and I can't give any canned solutions. You have backups, right?

## Hardware failure (solid-state media)

You have backups, right?

If you don't, now it's a good time to go cry.

Next time you should use `gsmartcontrol` (statistics tab) or `smartctl -x /dev/XXX` to monitor the remaining SSD endurance (it works with many NVMe drives too!) and get ready to order a replacement once you've used up ~60%.

Solid-state media fails silently and there's usually not a lot to do when an SSD dies.

Oh no wait, you're using an SD card and it died on you after you've been running a Linux distro off of it for a month on your Raspberry Pi? Well, congratulations, you just learned a lesson: NEVER use an SD card, no matter how expensive, to store important data. Also, 
[never use a Raspberry Pi](https://depau.github.io/3dprint-wiki/wiki/hardware/octoprint-devices/#easiest-option-raspberry-pi).


# Other soft-failures

## Can't `send` a submodule

You can only send read-only snapshots of submodules:

```bash
# I advise you add a prefix rather than a suffix for the new vol name
# so you don't risk pressing enter early and wiping the source volume.
btrfs subvolume snapshot -r my-subvol ro-my-subvol
btrfs send ro-my-subvol | ssh ...
btrfs subvol delete -c ro-my-subvol
```
{: .wrap-code}

# My btrfs tricks

## Data deduplication

Btrfs is able to deduplicate data. Unfortunately it can't do it transparently, yet. [Bees](https://github.com/Zygo/bees/) can do that for you, but beware:

- It uses a shitload of RAM
- It uses quite some CPU
- It kinda reduces I/O performance in my experience
- READ THE [GOTCHAS](https://github.com/Zygo/bees/blob/master/docs/gotchas.md) PAGE.

For what it's worth, I use it on my desktop which has 32GB of RAM. I don't on my laptop.

## Reflink copying

When copying a large file or directory, you can exploit btrfs's Copy-on-Write to duplicate the file without actually duplicating the data on the disk.

Add to your `.bashrc` or equivalent:

```bash
alias cp='cp --reflink=auto'
# "auto" will fallback to standard copying if reflink isn't available
```
{: .wrap-code}

Recent versions of Nautilus do this by default when copying from the file manager.

## Working with large projects

When working with very large projects such as the Android build system, a `make clean` operation will often take forever.

You can instead work in a subvolume, and snapshot it before building. Restoring the snapshot takes 0 time.


## Automated snapshots

There's a project from openSUSE called Snapper that can automatically manage Btrfs subvolume snapshots.

On Arch it can integrate with `pacman` and GRUB and allow booting the system to an earlier state.

See the [Arch wiki page](https://wiki.archlinux.org/title/Snapper).

I personally don't use it. I use [restic](https://restic.net/) to snapshot and back up my data.

This is because my back-up set up works on 3 stages (backups are pushed to a collection node, which then forwards them to the long-term storage machine). Btrfs send + incremental snapshots don't work well for this setup, since subvolumes can only be sent incrementally from the first node to a second one, but not from the second to a third one.

## Uncompressing large VM images

I once had to decompress and convert a very large (>2TB) raw VM image to QEMU's qcow2 format (with built-in compression).

The image actually contained ~60GB of data. Thanks to btrfs and its transparent compression I was able to uncompress it in my <100GB filesystem and convert it. This works since the file will automatically be recompressed, then the conversion tools can work on it as if my filesystem was large enough to hold the entire file.

