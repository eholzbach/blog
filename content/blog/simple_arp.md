---
title: "Simple arp poisoning resolution"
date: "2011-11-29T00:58:00-07:00"
slug: "simple-arp-poisoning-resolution"
---

[Address resolution protocol](http://en.wikipedia.org/wiki/Address_Resolution_Protocol) [poisoning](http://en.wikipedia.org/wiki/ARP_spoofing) is a problem which plagues most of us on switched ip v4 networks. Unfortunately this accounts for 95% of the worlds network infrastructure. Due to the limitations of ip v4, legacy installations, ease of use, lack of hardware solutions on low end gear, and lack of resources to maintain port security in high end gear, this 20 year old attack vector is all too common. Old problems lacking modern solutions. At least the [Lawrence Berkeley National Laboratory](http://ee.lbl.gov/) provided us with a detection utility titled arpwatch, which has been available since 1992.

After identifying a suspected arp poison attack and pinpointing the affected ip address range, you may easily locate the attacker with this tool. Assuming your attacker didn't destroy your network and you can still access the server,  you should quickly halt the attack. Running this simple script will correct the arp table for all ip addresses on the default interface on a FreeBSD server:
```bash
#!/bin/sh
FACE=$(netstat -rn |grep default|awk '{print $6}')
GATE=$(netstat -rn |grep default|awk '{print $2}')
for i in `ifconfig |sed -n "/$FACE/,/status/p" |grep "inet " |awk '{print $2}'` ;\
do arping -c 1 -i $FACE -S $i $GATE ;done
```
Install arpwatch on a server on the affected ip block, and update the mac address manufacture codes if desired:
```bash
cd /usr/ports/net-mgmt/arpwatch/ ; make install clean
cd /usr/local/arpwatch ; fetch http://standards.ieee.org/regauth/oui/oui.txt
./massagevendor oui.txt > ethercodes.dat ; rm oui.txt
```

Confirm /usr/local/arpwatch/arp.dat was created before you start arpwatch, else touch the file.

If the event has already taken place, you will see mismatched entries start to pour in to syslog. A sample of noted mismatched mac addresses on monitor-server, this went on for hundreds of lines.

```bash
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.97 0:16:17:28:b2:8d (bc:30:5b:d0:9e:82)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.96 0:16:17:28:b2:8d (bc:30:5b:d0:9e:82)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.95 0:16:17:28:b2:8d (78:2b:cb:1c:0:1d)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.94 0:16:17:28:b2:8d (78:2b:cb:8:6c:d8)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.93 0:16:17:28:b2:8d (78:2b:cb:22:27:d)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.92 0:16:17:28:b2:8d (78:2b:cb:1f:f0:a8)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.91 0:16:17:28:b2:8d (78:2b:cb:1f:f0:a8)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.90 0:16:17:28:b2:8d (78:2b:cb:8:6c:d8)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.89 0:16:17:28:b2:8d (78:2b:cb:3a:3c:88)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.88 0:16:17:28:b2:8d (78:2b:cb:19:ba:d7)
Nov 28 15:19:11 monitor-server arpwatch: ethernet mismatch 192.168.1.87 0:16:17:28:b2:8d (78:2b:cb:19:ba:d7)</blockquote>
```

Such large numbers are a clear indicator 0:16:17:28:b2:8d is the attacker. If the tool has been running for a while the arp.dat file will be populated with data. One entry per line formatted by mac address, ip address, time of last change in [unix epoch](http://en.wikipedia.org/wiki/Unix_time), and resolved hostname if available. Now simply grep the data file for the suspected attacker:
```bash
grep 0:16:17:28:b2:8d /usr/local/arpwatch/arp.dat
0:16:17:28:b2:8d 192.168.1.234 1322536658 rooted-server
```

Now you have an ip of the alleged attacker, and can continue with your incident response procedure. If your monitoring before/during the event, large amounts of "changed station" entries will be logged. The utility can be configured to email you of changes, ad can parse data from snmpwalk for easy integration.

Of course if this doesn't fit your environment,  you can always start flipping through you bandwidth logs until you find the lowly server now being DoS'ed into tomorrow while the rest of your gear is trying to jam 1000Mbs down its nic. What ever works best for you.
