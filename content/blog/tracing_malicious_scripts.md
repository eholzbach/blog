---
title: "Tracing malicious scripts on poorly configured gnu/linux servers"
date: "2011-04-07T23:44:00-07:00"
slug: "tracing-malicious-scripts-on-poorly-configured-gnulinux-servers"
---

Warning: This post is worthless if you are not me.

I have recently been poking at a few old servers that still run apache handler 2.0. You got it. The good old days of all processes running as <a href="http://en.wikipedia.org/wiki/Nobody_(username)" target="_blank">"nobody</a>" from start to finish.

<a href="http://en.wikipedia.org/wiki/Trusted_Computer_System_Evaluation_Criteria#Accountability" target="_blank">Accountability</a> is for the birds.

These servers enjoy consistently being part of various <a href="http://en.wikipedia.org/wiki/Botnet" target="_blank">botnets</a>. <a href="http://www.perl.org/" target="_blank">Perl</a> scripts are written to <a href="http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard" target="_blank">/tmp</a> with random filenames (as the "nobody" user),  executed by a call to the perl <a href="http://en.wikipedia.org/wiki/Executable_and_Linkable_Format" target="_blank">binary</a> (not just as ./blah), the <a href="http://en.wikipedia.org/wiki/Fork-exec" target="_blank">process forked</a>, and the original file <a href="http://en.wikipedia.org/wiki/Rm_(Unix)" target="_blank">rm</a>'ed. Insult to injury the <a href="http://en.wikipedia.org/wiki/Procfs" target="_blank">procfs</a> (which has been <a href="http://en.wikipedia.org/wiki/Deprecation" target="_blank">deprecated</a> in <a href="http://www.freebsd.org/" target="_blank">FreeBSD</a> in favor of <a href="http://www.freebsd.org/cgi/man.cgi?query=procstat&amp;apropos=0&amp;sektion=0&amp;manpath=FreeBSD+8.2-RELEASE&amp;format=html" target="_blank">procstat</a> and <a href="http://www.freebsd.org/cgi/man.cgi?query=sysctl&amp;apropos=0&amp;sektion=0&amp;manpath=FreeBSD+8.2-RELEASE&amp;format=html" target="_blank">sysctl</a>) entries are altered. This leaves little to trace as the usual task of using <a href="http://en.wikipedia.org/wiki/Lsof" target="_blank">lsof</a> against the <a href="http://en.wikipedia.org/wiki/Process_identifier" target="_blank">pid</a> will yield unproductive results on this configuration. You get the pid for a deleted file which has already been processed by perl, hence the entire perl exe is associated. As this particular altercation destroys the <a href="http://en.wikipedia.org/wiki/File_descriptor" target="_blank">file descriptor</a>, trying <a href="http://en.wikipedia.org/wiki/Cp_(Unix)" target="_blank">cp</a> the file via its /proc file descriptor results in zero bytes actually copied.

These types of processes are frequently spoofed to view to <a href="http://en.wikipedia.org/wiki/Ps_(Unix)" target="_blank">ps</a> or <a href="http://en.wikipedia.org/wiki/Top_(software)" target="_blank">top</a> as:
<blockquote>/usr/local/apache/bin/httpd -k start -DSS
/usr/sbin/acpid
syslogd
bash
everything else that ever runs on any server ever</blockquote>
How does this happen? Perl makes it simple.
<blockquote>perl -e '$0 = "/usr/local/sbin/h4x_y3r_f4c3"; system "ps -f $$"'</blockquote>
I must note that gnu/linux systems by default will not tell you its a perl script, and FreeBSD will. The following is the output of the above command first on a linux system, then a FreeBSD system.
<blockquote>UID        PID  PPID  C STIME TTY      STAT   TIME CMD
username   1616 30537  0 23:50 pts/0    S+     0:00 /usr/local/sbin/h4x_y3r_f4c3</blockquote>
On a FreeBSD system..
<blockquote>PID  TT  STAT      TIME COMMAND
9487   0  S+     0:00.01 /usr/local/sbin/h4x_y3r_f4c3 (perl5.10.1)</blockquote>
&nbsp;

Yes, I had to include a reason why FreeBSD is a better system. Yes, I think it's funny this site is hosted on a GNU/Linux system.

Easy solution is to block the outbound port in <a href="http://www.netfilter.org/projects/iptables/index.html" target="_blank">iptables</a> (I have grown fond of <a href="http://www.rfxn.com/projects/advanced-policy-firewall/" target="_blank">apf</a> for its ease of use) and be done with it, but thats not good enough for me. First thought was wishing <a href="http://en.wikipedia.org/wiki/Mount_(computing)" target="_blank">mount</a> or <a href="http://en.wikipedia.org/wiki/Chattr" target="_blank">chattr<a> had a "stupid and almost never useful" setting of read/write/nodelete. This would have easily solved the problem as I'm sure the <a href="http://en.wikipedia.org/wiki/Dropper" target="_blank">dropper</a> rm's the file without truncating it first. It is a production server, so I'm unable to unmount the <a href="http://en.wikipedia.org/wiki/Disk_partitioning" target="_blank">partition</a> and grep blocks for strings. Another issue would be getting permission to make such a change on a production server to research something that's basically a waste of time in the scope of daily operations. These servers will not be around much longer. No, really. I mean it this time.

I'm not a programmer (can read and noodle with <a href="http://en.wikipedia.org/wiki/C_(programming_language)" target="_blank">c</a>/perl/<a href="http://en.wikipedia.org/wiki/Bash_(Unix_shell)" target="_blank">bash</a> and was proficient in <a href="http://en.wikipedia.org/wiki/Assembly_language" target="_blank">assembly</a>) so I totally missed the "easy" solution. I knew you could view the memory  ranges a process consumes from the proc entry, but i did not know <a href="http://www.gnu.org/software/gdb/" target="_blank">gdb</a> (which is horrid for anything you don't have the source code for) offered this function, nor that gcore was a stand alone utility to produce <a href="http://en.wikipedia.org/wiki/Core_dump" target="_blank">coredumps</a> from running processes without terminating them.
<blockquote>gcore -o dumpfile pid</blockquote>
Not an ideal answer as I now have a 50-100 (usually closer to 64) megabyte memory dump to parse through, though it contains the data that solves my personal curiosity. I'm able to extract all the <em>information I want</em> from a standard coredump. What the abused server has been doing, what other servers it's communicating with, and the actual "important" part.

What user account the offending script was dropped from.

A standard coredump contains the <a href="http://en.wikipedia.org/wiki/Unix_shell" target="_blank">shell</a> environment variables, which includes the working (or previously working) directory. A simple command to search for a string (look for /home/ or PWD=) can produce the original working directory. Now that you know the account and path, it should be a trivial task to locate the origins or enabler of the offending file.

