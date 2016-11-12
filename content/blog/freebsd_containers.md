---
date: "2012-01-07T21:58:00-07:00"
title: "FreeBSD containers made easy"
slug: "freebsd-containers-made-easy"
---

FreeBSD jails are far from new. This [feature](http://www.freebsd.org/cgi/man.cgi?query=jail&amp;apropos=0&amp;sektion=0&amp;manpath=FreeBSD+4.0-RELEASE&amp;arch=default&amp;format=html) was committed to RELEASE in 2000 with [4.0-RELEASE](http://www.freebsd.org/releases/4.0R/notes.html). While the feature set has drastically grown, I rarely hear it mentioned in conversation discussing the pros and cons of [VMware](http://www.vmware.com/) vs [VirtualBox](https://www.virtualbox.org/) vs [openVZ](http://wiki.openvz.org/Main_Page) vs [Linux-KVM](http://www.linux-kvm.org/page/Main_Page).  Those familiar with system [virtualization](http://en.wikipedia.org/wiki/Virtualization) have most likely fiddled VMware,  VirtualBox or Linux-KVM. These [hypervisors](http://en.wikipedia.org/wiki/Hypervisor) have the ability to emulate hardware environments and run os'es other than the host. Virtualization solutions like [FreeBSD](http://www.freebsd.org) jails and [OpenVZ](http://wiki.openvz.org/Main_Page) operate differently. They take the monolithic approach, sharing the host kernel with the virtualized operating system container. This provides a low overhead solution, though neither are able to run a different operating system.

The [FreeBSD handbook](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/) does a fine job of walking one through [creation and configuration of jails](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/jails.html) without use of ports, but it's not user friendly. The sysutils/ezjail port simplifies the process.

Before you get started, take a look at [netstat(1)](http://www.freebsd.org/cgi/man.cgi?query=netstat&amp;apropos=0&amp;sektion=1&amp;manpath=FreeBSD+8.3-RELEASE&amp;arch=default&amp;format=html) or [sockstat(1)](http://www.freebsd.org/cgi/man.cgi?query=sockstat&amp;sektion=1). Any service that is bound to all ip's (*:*) needs to be assigned to a single ip to avoid conflict.

Add an ip aliases to the server:
>ifconfig msk0 192.168.1.100 netmask 255.255.255.255 alias

To make it persistent, update /etc/rc.conf:

> ifconfig_msk0_alias0="192.168.1.100 netmask 255.255.255.255"

Compile the world. If you have not installed the FreeBSD source you may do so using sysinstall by adding the full source distribution. Its a necessary process, and doing it now allows one to pay less attention in the future. Might as well install ezjail now too.

> cd /usr/src ; make buildworld ; cd /usr/ports/sysutils/ezjail ; make install clean

By default the jail data lives at /jails. In my instance I wanted the jails to live on another disk at /blahmountpoint, so I updated the ezjail_jaildir variable in /usr/local/etc/ezjail.conf.

Create the base environment for your jails:

> ezjail-admin update -p -i

Create your jails with "ezjail-admin create FQDN ip_address":

> ezjail-admin create jailname.yourdomain.net 192.168.1.100

Depending on your application of the virtualized environment, you may want the jail (container) to have the privilege to create raw sockets (ping, traceroute, 3r33th4x) from within your jail. This is disabled by default. You may allow raw sockets to your jail by setting security.jail.allow_raw_sockets to 1, and can be made persistent by updating /etc/sysctl.conf:
> sysctl security.jail.allow_raw_sockets=1

Now add ezjail_enable="YES" to /etc/rc.conf and start your jail with:

> /usr/local/etc/rc.d/ezjail start

SSHD is not running by default. You may start services by entering the container (jail) from the host and configuring the network services. You may use "jls" to list the installed jails and enter the container (jail) with jexec:

> jexec 1 csh

> echo '"sshd_enable="YES"' >> /etc/rc.conf ; /etc/rc.d/sshd start

Now shell your FreBSD vps and go wild.


