---
title: "Password management and sinking to KeePassX"
date: "2011-02-04T17:10:00-07:00"
slug: "password-management-and-sinking-to-keepassx"
---

For the past several years I have been using [pwman](http://pwman.sourceforge.net/) for password management. It's an [ncurses](http://en.wikipedia.org/wiki/Ncurses) interface for an [xml](http://en.wikipedia.org/wiki/XML) file encrypted with [gpg](http://www.gnupg.org/). This program was an ideal solution. It provided an easy to use interface for a file encrypted with an industry standard utility, which I could use remotely, regardless if pwman was installed as long as I had my gpg key. Almost perfect.

Unfortunately pwman is unstable and tends to produce [segmentation faults](http://en.wikipedia.org/wiki/Segmentation_fault). This is unacceptable for regular use as most [unix like](http://en.wikipedia.org/wiki/Unix-like) systems have [core dumps](http://en.wikipedia.org/wiki/Core_dump) enabled, writing the entire password database in [plain text](http://en.wikipedia.org/wiki/Plaintext) to disk. Several conditions which result in a crash exist from adding, removing, and editing entries. There is one bug I've been able to easily replicate. Editing fields that end with a period (yes, a ".") will result in a crash. A review of the decrypted xml file shows no corruption from the standard format, nor extra characters from the period. I have not flipped through the source to isolate and correct the bug.

Why not?

I have a flashy new [android](https://www.android.com/) based phone. There have been several comments at my current place of employment regarding standardized [key exchange](http://en.wikipedia.org/wiki/Key_exchange) through KeePassX databases. I want a solution where I can share the same database across platforms and employment, yet still access from a remote host via ssh.

Searching for a multi-platform solution not reliant on [xorg](http://xorg.org) (yes, that's what's providing your mac with its interface) yielded no results that fit the bill. My next thought was to go back to using an encrypted text file without an interface. This proves to be another dead end as there is no real android gpg port.

The end. I give up.

I tried several console interfaces which handle [KeePassX](http://www.keepassx.org/) databases, and a [perl](http://www.perl.org/) script called [kpcli](http://kpcli.sourceforge.net/) was the winner. It took a bit of work to get it running on the [FreeBSD](http://www.freebsd.org) systems I regularly use.

Kpcli will not work on FreeBSD 8.1 out of the box. It ships with perl version 5.8 which does not allow [nested](http://en.wikipedia.org/wiki/Nested_function) [regex](http://en.wikipedia.org/wiki/Regular_expression) [quantifiers](http://en.wikipedia.org/wiki/Quantification). You first need to upgrade perl to 5.10 or higher, I chose the 5.10 branch, following the commands as listed in /usr/ports/UPDATING at 20090328.

> pkgdb -Ff<br>
> env DISABLE_CONFLICTS=1 portupgrade -o lang/perl5.10 -f perl-5.8.\*<br>
> portupgrade -fr perl

Unfortunately on all the systems I tried this on I was still left with some ports failing to recompile and would no longer start. You can also recompile all installed ports with portupgrade -af if you're not willing to approach and correct these issues as they arise.

Now download the perl script to interact with the database if you haven't already.

> fetch http://cdnetworks-us-1.dl.sourceforge.net/project/kpcli/kpcli-0.8.pl

If your not a perl person you most likely do not have the needed [cpan modules](http://www.cpan.org/index.html) installed. For my systems I was lacking the following:

> /security/p5-Crypt-Rijndael/<br>
> /textproc/p5-Sort-Naturally/<br>
> /shells/p5-Term-ShellUI<br>
> /devel/p5-ReadLine-Gnu/<br>
> /devel/p5-Term-ReadKey/

After which I was only lacking the file::keepassx cpanel module. There is not currently a FreeBSD port for this, so you will need to install it thought the cpan interface.

> perl -MCPAN -e shell<br>
> install File::KeePass

The interface for kpcli is familiar to say the least. The following is an example of working with it, accessing the KeePassX database.

```bash
kraken:~% kpcli

KeePass CLI (kpcli) v0.8 is ready for operation.
Type 'help' for a description of available commands.
Type 'help <command>' for details on individual commands.
* Please upgrade Term::ShellUI to version 0.87 or newer.

kpcli:/> open .kpass.kdb
Please provide the master password:
kpcli:/> ls
=== Groups ===
bills/
sites/
work/
backup/
kpcli:/> cd bills
kpcli:/bills> ls
=== Entries ===
0. Suntrust          	                                      suntrust.com

kpcli:/bills> show 0

Title: Suntrust
Uname: fakeusername
Pass: realpassword
URL: suntrust.com
Notes:

kpcli:/bills>
```
