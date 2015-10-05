---
title: "Coreboot on the lenovo x230"
date: "2015-10-05T11:24:00-07:00"
---

I've been a fan of the Thinkpad line since I first saw the little red dot mouse in the middle of the keyboard dubbed the "trackpoint" by IBM. Ten-ish years ago IBM's personal computer line was sold off to Lenovo. The new owners have upheld the brand and continue to assemble decent hardware. They also has a long track record of not caring about privacy of end users and consistently backdoor the products they ship. This year alone they have been turned out for shipping back-doored versions of Windows, [spyware that's persistent in the bios](http://www.v3.co.uk/v3-uk/news/2422015/lenovo-caught-installing-bloatware-again-with-windows-bios-backdoor), [preloaded 3rd party applications that circumvents ssl](http://www.forbes.com/sites/thomasbrewster/2015/02/19/superfish-need-to-know/), and now [Lenovo is preloading fun tracking software](http://www.computerworld.com/article/2984889/windows-pcs/lenovo-collects-usage-data-on-thinkpad-thinkcentre-and-thinkstation-pcs.html). I have not (as far as I know) been impacted by the discovered and disclosed issues because I don't run Windows on Lenovo hardware. This doesn't mean FreeBSD or GNU/Linux users safe. It just means we have yet to be targeted (as far as we know) by Lenovo.

The farther we can get from locked firmware the better off we are. [Coreboot](http://www.coreboot.org) helps distance us from black box software. The [x230 has been supported](http://review.coreboot.org/gitweb?p=coreboot.git;a=commit;h=e7e9502d46735e4f1aafd9b362d912070b9bb29d&utm_source=anzwix) for a while, but I couldn't find any data from end users about their experience running it. I threw caution to the wind. Trial and error often pays off.

The instructions found [on the coreboot wiki](http://www.coreboot.org/Board:lenovo/x230) are straight forward for those who know what they are doing. I'm not one of those people. 

The firmware is locked so the process requires an external flasher. I used a Pomona 5250 8 pin SOIC clip and a Raspberry Pi. There is a [good diagram of the GPIO pin out here](https://github.com/bibanon/Coreboot-ThinkPads/wiki/Hardware-Flashing-with-Raspberry-Pi). From SOIC to GPIO it goes 1 -> 24, 2 -> 21, 4 -> 25, 5 -> 19, 6 -> 23, 8 -> 17. Flashrom detected a few possibilities of possible chipsets. I didn't have a magnifying glass handy to read what was printed on the chip, so I made backup's of all 4 types it suggested so I could brute force a disaster recovery. Later I found my IC is detected by [flashrom](http://flashrom.org/Flashrom) as "MX25L3206E/MX25L3208E". A common Macronix chip.

Coreboot highly recommends using their patched tool chain to build coreboot roms. Don't bother trying to build the roms on your Pi. It will take forever and oom-killer may get to it. I used make nconfig to select coreboot options. Here is my configuration:

<pre>general  --|
           |-[*] Compress ramstage with LZMA
           |-[*] Include the coreboot .config file into the ROM image
mainboard -|
           |-Mainboard vendor (Lenovo)
           |-Mainboard model (ThinkPad X230)
           |-ROM chip size (12288 KB (12 MB))
           |-(0x100000) Size of CBFS filesystem in ROM
devices ---|
           |-[*] Use native graphics initialization
generic ---|
           |-[*] PS/2 keyboard init
console ---|
           |-[*] Squelch AP CPUs from early console.
           |-[*] Send console output to a CBMEM buffer
           |-[*] Send POST codes to an external device
           |-[*] Send POST codes to an IO port
sys table -|
           |-[*] Generate SMBIOS tables
payload ---|
           |-Add a payload (SeaBIOS)
           |-SeaBIOS version (master)
           |-(10) PS/2 keyboard controller initialization timeout (milliseconds)
           |-[*] Hardware init during option ROM execution
           |-[*] Include generated option rom that implements legacy VGA BIOS compatibility
           |-[*] Use LZMA compression for payloads</pre>

I went with SeaBIOS as a payload because I wanted to run FreeBSD without the hassle of grub2. The only issue I had during the migration process was self induced. I had assumed that all modern keyboards, laptops included, would be attached to the usb bus. Not the case. Almost all laptop keyboards are ps/2 devices. Once I recompiled and re-flashed with ps2 enabled I could select other boot devices in SeaBIOS.

I've booted FreeBSD 10.2-RELEASE and Debian 8.1 on this laptop without issue. Everything I wanted to work, works. A full list of tested and working features are on the [device specific coreboot wiki page](http://www.coreboot.org/Board:lenovo/x230).
