---
title: "Migrating FreeBSD to a new hard drive"
date: "2011-09-05T14:58:00-07:00"
slug: "migrating-freebsd-to-a-new-hard-drive"
---

I arrived at work to find a post-it note on my desk which read "Your disk was spinning <span style="text-decoration: underline;">a lot</span> on Sunday". The office was quiet, so a colleague could hear the disk spinning out of control. I leaned forward to hear the sound of a failing disk. High rpm spin up, slow down, and repeat. The usual behavior before the chug of death starts. [Smartctl](http://en.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology) had reported roughly 30,000 seek errors in the past 20 minutes.

The [dump and restore](http://www.freebsd.org/doc/handbook/backup-basics.html) utilities are part of [FreeBSD](http://www.freebsd.org/) which allow you to backup and restore entire [file systems](http://en.wikipedia.org/wiki/File_system)fil 20 minutes.

The [dump and restore](http://www.freebsd.org/doc/handbook/backup-basics.html) utilities are part of [FreeBSD](http://www.freebsd.org/) which allow you to backup and restore entire [file systems](http://en.wikipedia.org/wiki/File_system)file systems. These utilities can work with [pipes](http://en.wikipedia.org/wiki/Pipeline_(Unix)), allowing you to easily manipulate the data, and send it elsewhere, ie [gzip](http://en.wikipedia.org/wiki/Gzip) it and store it remotely with [ssh](http://en.wikipedia.org/wiki/OpenSSH). They can work on live file systems. These features allow for a painless migration to a new hard drive, without the hassle of reinstalling the operating system. While this could lead to instability on a drive failing with a high number of read errors or [bad sectors](http://en.wikipedia.org/wiki/Bad_sector), in this instance it was a low risk transfer.

Linux users may find the drive naming structure odd, though its a much better design. Linux [sata drives](http://en.wikipedia.org/wiki/Serial_ATA) show up in the order found, as /dev/sd[a-z] and may have a trailing digit to specify partitions on the drive. FreeBSD uses the sata controller the drive is physically plugged into to define where it lives in the system, followed by the slice number, then partition. This scheme keeps drives named as they should be, regardless if other disks are added or removed from the system, or which order they are detected on boot.

You may use fdisk and disklabel to cut the slice and set up partitions on your new drive, though [sysinstall](http://www.freebsd.org/cgi/man.cgi?query=sysinstall&amp;sektion=8) or the stand alone [sade](http://www.freebsd.org/cgi/man.cgi?query=sade&amp;apropos=0&amp;sektion=8&amp;manpath=FreeBSD+8.2-RELEASE&amp;arch=default&amp;format=html) are better suited for ease of use.  In this example my current installation was on /dev/ad4, and I was moving to a new sata drive attached on /dev/ad1.

My current partition scheme and their mount points looked like this:
<blockquote>/dev/ad4s1a /
/dev/ad4s1e /tmp
/dev/ad4s1f  /usr
/dev/ad4s1d /var</blockquote>
The commands to migrate are simple.
<blockquote>newfs /dev/ad1s1a
mount /dev/ad1s1a /mnt
cd /mnt
dump 0afL - / | restore rf -</blockquote>
This was to move my root partition. Take note the L argument for dumping from a live file system. Follow the same process to dump usr, tmp, and var, piping them to their new location. Edit fstab on the new drive to correctly reflect where it is or will be. Shutdown the machine, remove your failing disk, boot into your migrated installation, and enjoy not having to reinstall or configure anything.

Finally, stare at the failing drive now sitting on your desk, and curse the manufacturer. Nobody produces quality drives anymore.
