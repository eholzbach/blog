---
title: "Tracing malicious scripts on poorly configured gnu/linux servers"
date: "2011-04-07T23:44:00-07:00"
slug: "tracing-malicious-scripts-on-poorly-configured-gnulinux-servers"
---

Warning: This post is worthless if you are not me.

I have recently been poking at a few old servers that still run apache handler 2.0. You got it. The good old days of all processes running as [nobody](http://en.wikipedia.org/wiki/Nobody_(username)) from start to finish.

[Accountability](http://en.wikipedia.org/wiki/Trusted_Computer_System_Evaluation_Criteria#Accountability) is for the birds.

These servers enjoy consistently being part of various [botnets](http://en.wikipedia.org/wiki/Botnet). [Perl](http://www.perl.org/) scripts are written to [/tmp](http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) with random filenames (as the "nobody" user),  executed by a call to the perl [binary](http://en.wikipedia.org/wiki/Executable_and_Linkable_Format) (not just as ./blah), the [process forked](http://en.wikipedia.org/wiki/Fork-exec), and the original file [rm](http://en.wikipedia.org/wiki/Rm_(Unix))'ed. Insult to injury the [procfs](http://en.wikipedia.org/wiki/Procfs) (which has been [deprecated](http://en.wikipedia.org/wiki/Deprecation) in [FreeBSD](http://www.freebsd.org/) in favor of [procstat](http://www.freebsd.org/cgi/man.cgi?query=procstat&amp;apropos=0&amp;sektion=0&amp;manpath=FreeBSD+8.2-RELEASE&amp;format=html) and [sysctl](http://www.freebsd.org/cgi/man.cgi?query=sysctl&amp;apropos=0&amp;sektion=0&amp;manpath=FreeBSD+8.2-RELEASE&amp;format=html) entries are altered. This leaves little to trace as the usual task of using [lsof](http://en.wikipedia.org/wiki/Lsof) against the [pid](http://en.wikipedia.org/wiki/Process_identifier) will yield unproductive results on this configuration. You get the pid for a deleted file which has already been processed by perl, hence the entire perl exe is associated. As this particular altercation destroys the [file descriptor](http://en.wikipedia.org/wiki/File_descriptor), trying to [cp](http://en.wikipedia.org/wiki/Cp_(Unix)) the file via its /proc file descriptor results in zero bytes actually copied.

These types of processes are frequently spoofed to view in [ps](http://en.wikipedia.org/wiki/Ps_(Unix)) or [top](http://en.wikipedia.org/wiki/Top_(software)) as:

> /usr/local/apache/bin/httpd -k start -DSS<br>
> /usr/sbin/acpid<br>
> syslogd<br>
> bash<br>
> everything else that ever runs on any server ever

How does this happen? Perl makes it simple.

> perl -e '$0 = "/usr/local/sbin/h4x_y3r_f4c3"; system "ps -f $$"'

I must note that gnu/linux systems by default will not tell you its a perl script, and FreeBSD will. The following is the output of the above command first on a linux system, then a FreeBSD system.

> UID        PID  PPID  C STIME TTY      STAT   TIME CMD<br>
> username   1616 30537  0 23:50 pts/0    S+     0:00 /usr/local/sbin/h4x_y3r_f4c3

On a FreeBSD system..

> PID  TT  STAT      TIME COMMAND<br>
> 9487   0  S+     0:00.01 /usr/local/sbin/h4x_y3r_f4c3 (perl5.10.1)

Yes, I had to include a reason why FreeBSD is a better system.

Easy solution is to block the outbound port in [iptables](http://www.netfilter.org/projects/iptables/index.html) (I have grown fond of [apf](http://www.rfxn.com/projects/advanced-policy-firewall/) for its ease of use) and be done with it, but thats not good enough for me. First thought was wishing [mount](http://en.wikipedia.org/wiki/Mount_(computing)) or [chattr](http://en.wikipedia.org/wiki/Chattr) had a "stupid and almost never useful" setting of read/write/nodelete. This would have easily solved the problem as I'm sure the [dropper](http://en.wikipedia.org/wiki/Dropper) rm's the file without truncating it first. It is a production server, so I'm unable to unmount the [partition](http://en.wikipedia.org/wiki/Disk_partitioning) and grep blocks for strings. Another issue would be getting permission to make such a change on a production server to research something that's basically a waste of time in the scope of daily operations. These servers will not be around much longer. No, really. I mean it this time.

I'm not a programmer (can read and noodle with [c](http://en.wikipedia.org/wiki/C_(programming_language))/perl/[bash](http://en.wikipedia.org/wiki/Bash_(Unix_shell)) and was proficient in [assembly](http://en.wikipedia.org/wiki/Assembly_language) so I totally missed the "easy" solution. I knew you could view the memory  ranges a process consumes from the proc entry, but i did not know [gdb](http://www.gnu.org/software/gdb/) (which is horrid for anything you don't have the source code for) offered this function, nor that gcore was a stand alone utility to produce [core dumps](http://en.wikipedia.org/wiki/Core_dump) from running processes without terminating them.

> gcore -o dumpfile pid

Not an ideal answer as I now have a 50-100 (usually closer to 64) megabyte memory dump to parse through, though it contains the data that solves my personal curiosity. I'm able to extract all the *information I want* from a core dump. What the abused server has been doing, what other servers it's communicating with, and the actual "important" part.

What user account the offending script was dropped from.

A standard core dump contains the [shell](http://en.wikipedia.org/wiki/Unix_shell) environment variables, which includes the working (or previously working) directory. A simple command to search for a string (look for /home/ or PWD=) can produce the original working directory. Now that you know the account and path, it should be a trivial task to locate the origins or enabler of the offending file.

