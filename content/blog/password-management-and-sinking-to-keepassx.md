---
title: "Password management and sinking to KeePassX"
date: "2011-02-04T17:10:00-07:00"
slug: "password-management-and-sinking-to-keepassx"
---

For the past several years I have been using <a href="http://pwman.sourceforge.net/" target="_blank">pwman</a> for password management. It's an <a href="http://http://en.wikipedia.org/wiki/Ncurses" target="_blank">ncurses</a> interface for an <a href="http://en.wikipedia.org/wiki/XML" target="_blank">xml</a> file encrypted with <a href="http://www.gnupg.org/" target="_blank">gpg</a>. This program was an ideal solution. It provided an easy to use interface for a file encrypted with an industry standard utility, which I could use remotely, regardless if pwman was installed as long as I had my gpg key. Almost perfect.

Unfortunately pwman is unstable and tends to produce <a href="http://en.wikipedia.org/wiki/Segmentation_fault" target="_blank">segmentation faults</a>. This is  unacceptable for regular use as most "<a href="http://en.wikipedia.org/wiki/Unix-like" target="_blank">unix like</a>" systems have <a href="http://en.wikipedia.org/wiki/Core_dump" target="_blank">core dumps</a> enabled, writing the entire password database in <a href="http://en.wikipedia.org/wiki/Plaintext" target="_blank">plaintext</a> to file. I'm convinced several conditions which result in a crash exist from adding, removing, and editing entries. There is one bug I've been able to easily replicate. Editing fields that end with a period (yes, a ".") will result in a crash. A review of the decrypted xml file shows no corruption from the standard format, nor extra characters from the period. I have not flipped through the source to isolate and correct the bug.

Why not?

I have a flashy <a href="http://www.android.com/">android</a> based phone. There have been several comments at my current place of employment regarding standardized <a href="http://en.wikipedia.org/wiki/Key_exchange" target="_blank">key exchange</a> through KeePassX databases. I want a solution where I can share the same database across platforms and employment, yet still access from a remote host via ssh.

Searching for a multi-platform solution not reliant on <a href="http://xorg.org" target="_blank">xorg</a> (yes, that's what's providing your mac with its interface) yielded no results that fit the bill. My next thought was to go back to using an encrypted text file without an interface. This proves to be another dead end as there is no real android gpg port.

The end. I give up.

I tried several console interfaces which handle <a href="http://www.keepassx.org/" target="_blank">KeePassX</a> databases, and a <a href="http://www.perl.org/" target="_blank">perl</a> script called <a href="http://kpcli.sourceforge.net/" target="_blank">kpcli</a> was the winner. It took a bit of work to get it running on the <a href="http://www.freebsd.org" target="_blank">FreeBSD</a> systems I regularly use.

Kpcli will not work on FreeBSD 8.1 out of the box. It ships with perl version 5.8 which does not allow <a href="http://en.wikipedia.org/wiki/Nested_function" target="_blank">nested</a> <a href="http://en.wikipedia.org/wiki/Regular_expression" target="_blank">regex</a> <a href="http://en.wikipedia.org/wiki/Quantification" target="_blank">quantifiers</a>. You first need to upgrade perl to 5.10 or higher, I chose the 5.10 branch, following the commands as listed in /usr/ports/UPDATING at 20090328.
<blockquote>pkgdb -Ff
env DISABLE_CONFLICTS=1 portupgrade -o lang/perl5.10 -f perl-5.8.\*
portupgrade -fr perl</blockquote>
Unfortunately on all the systems I tried this on I was still left with some ports failing to recompile and would no longer start. You can also recompile all installed ports with portupgrade -af if you're not willing to approach and correct these issues as they arise.

Now download the perl script to interact with the database if you haven't already.
<blockquote>fetch http://cdnetworks-us-1.dl.sourceforge.net/project/kpcli/kpcli-0.8.pl</blockquote>
If your not a perl person you most likely do not have the needed <a href="http://www.cpan.org/index.html" target="_blank">cpan modules</a> installed. For my systems I was lacking the following:
<blockquote>/security/p5-Crypt-Rijndael/
/textproc/p5-Sort-Naturally/
/shells/p5-Term-ShellUI
/devel/p5-ReadLine-Gnu/
/devel/p5-Term-ReadKey</blockquote>
After which I was only lacking the file::keepassx cpanel module. There is not currently a FreeBSD port for this, so you will need to install it thought the cpan interface.
<blockquote>perl -MCPAN -e shell
install File::KeePass</blockquote>
The interface for kpcli is familiar to say the least. The following is an example of working with it, accessing the KeePassX database.
<blockquote>kraken:~% kpcli

KeePass CLI (kpcli) v0.8 is ready for operation.
Type 'help' for a description of available commands.
Type 'help &lt;command&gt;' for details on individual commands.
* Please upgrade Term::ShellUI to version 0.87 or newer.

kpcli:/&gt; open .kpass.kdb
Please provide the master password:
kpcli:/&gt; ls
=== Groups ===
bills/
sites/
work/
backup/
kpcli:/&gt; cd bills
kpcli:/bills&gt; ls
=== Entries ===
0. Suntrust          	                                      suntrust.com

kpcli:/bills&gt; show 0

Title: Suntrust
Uname: fakeusername
Pass: realpassword
URL: suntrust.com
Notes:

kpcli:/bills&gt;</blockquote>
