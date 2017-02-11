---
title: "Sniffing packets on the Raspberry Pi 3 with FreeBSD, Netmap, and Suricata"
date: "2017-02-11T12:40:00-07:00"
---
I run Suricata on a Raspberry Pi 3 at home. Suricata is in af_packet mode, cluster type is cpu, and I have the ring size dialed in to comsume most of the systems memory. The source network isn't of interest. Browsing traffic of a few people, mobile devices, streaming services, cat videos and garbage devices that clog up the tubes. The pi 3 surprisingly holds up well, for a device with a usb nic. Suricata drops < 3% of packets on average. It works. That's fine and all, but I've been wondering how netmap on FreeBSD would function on restrictive and unsupported hardware. FeeBSD can run on the Raspberry Pi 3. Guess it's time to find out.

The Pi 3 is not officially supported platform, though it's been a board option in [crochet](https://github.com/freebsd/crochet) for several months. I started with a FreeBSD 12.0-CURRENT build machine.

```bash
pkg install git subversion sysutils/u-boot-rpi3 aarch64-binutils
svn co https://svn0.us-west.freebsd.org/base/head /usr/src
cp /usr/src/sys/arm64/conf/GENERIC-NODEBUG /usr/src/sys/arm64/conf/SNORTY
sed -i 's/^ident GENERIC.*/ident SNORTY/' /usr/src/sys/arm64/conf/SNORTY 
echo 'device netmap' >> /usr/src/sys/arm64/conf/SNORTY
git clone https://github.com/freebsd/crochet.git
```
Follow the crochet build instructions. Once finished, dd the image to a microsd card, booted the system, and confirmed the netmap device made it in.
```bash
root@snorty:~ # ls -l /dev/netmap
crw-------  1 root  wheel  0xc Feb  4 06:33 /dev/netmap
```
Great success, though no wifi. The nic is attached via SDIO which is currently unsupported by FreeBSD. Nothing a usb ethernet dongle can't solve.

After poking around a bit I noticed the kernel scrolling events. Reboot attempts also fail.

```bash
Feb  2 20:50:42 snorty kernel: cpufreq0: rejecting change, SMP not started yet
Feb  2 20:51:13 snorty last message repeated 117 times
Feb  2 20:53:14 snorty last message repeated 465 times
Feb  2 21:03:15 snorty last message repeated 2309 times
```
This error reads like an additional cpu can't start. [This hunk of code](https://github.com/freebsd/freebsd/blob/master/sys/kern/kern_cpu.c#L265-L276) confirms it. A google search lead me to a FreeBSD developer who has been working on the Pi3, and a [blog post containing a quick solution](https://kernelnomicon.org/?p=718). [The change was commited to crochet](https://github.com/freebsd/crochet/commit/43dd6a458849fd1046cb643292351377f29a0a97) a few days ago so you might not have to follow the solution from Gonzo's post. 

I also had some unnecessary mount points. I assume this was an artifact of using growfs option in crochet. I unmounted them and commented out of fstab. 
```bash
root@rpi3:~ # cat /etc/fstab
/dev/mmcsd0s1   /boot/efi       msdosfs rw,noatime      0 0
/dev/mmcsd0s2a  /               ufs rw,noatime          1 1
#md             /tmp            mfs rw,noatime,-s30m    0 0
#md             /var/log        mfs rw,noatime,-s15m    0 0
#md             /var/tmp        mfs rw,noatime,-s12m    0 0
```
 
There are no FreeBSD-12 packages for arm64, but the binaries for 11 work just fine. I bootstrapped the package system, configured pkg to use it moving forward, and installed the dependencies for Suricata. The FreeBSD port for Suricata is a little stale, but I'll start there and roll my own updated port if need be.
```bash
env ABI=FreeBSD:11:aarch64 pkg bootstrap
echo 'ABI = "FreeBSD:11:aarch64";' >> /usr/local/etc/pkg.conf
pkg install -y autoconf autoconf-wrapper automake automake-wrapper ca_root_nss gettext-runtime gmake gmp gnutls indexinfo jansson libffi libgcrypt libgpg-error libhtp libiconv libidn libltdl libnet libprelude libtasn1 libtool libyaml m4 nettle p11-kit pcre perl5 pkgconf tpm-emulator trousers dialog4ports 
portsnap fetch extract
cd /usr/ports/security/Suricata
make install clean
```
I configured Suricata to use netmap, and the monitor interface to come up at boot with "up -arp -rxcsum promisc" in rc.conf. I loaded around 20,000 rules. Suricata was humming away, eve and dns logs scrolling. After a while I checked the stats.log. No dropped packets. On the interface I see the same. This seems too good to be true.

```bash
root@snorty:/var/log/suricata # netstat -idbn -I ue0
Name    Mtu Network       Address              Ipkts Ierrs Idrop     Ibytes    Opkts Oerrs     Obytes  Coll  Drop
ue0    1500 <Link#2>      b8:27:eb:fe:e3:47 44204928     0     0   14682148        0     0          0     0     0
root@snorty:/var/log/suricata # egrep 'kernel|bytes|drop' stats.log | tail -3
decoder.bytes                              | Total                     | 18130815425
capture.kernel_packets                     | Total                     | 17036011
decoder.bytes                              | Total                     | 18228215774
```

It is. Maybe. [Dropped packets are not logged by Suricata in netmap mode](https://redmine.openinfosecfoundation.org/issues/1799). I don't expect drops at the interface. This was never an issue for the linux/af_packet combination. For the small amount of traffic most all the drops were in software.

Not the end results I was looking for.
