---
title: "Using mutt with imap and imapfilter on FreeBSD"
date: "2011-07-08T01:19:00-07:00"
slug: "using-mutt-with-imap-and-imapfilter-on-freebsd"
---

I generally prefer to use [console applications](http://en.wikipedia.org/wiki/Console_application) over anything with a [graphical user interface](http://en.wikipedia.org/wiki/Graphical_user_interface). I picked up this habit years ago from the desire to stuff everything I use in a [screen session](http://en.wikipedia.org/wiki/GNU_Screen), accessible from a [remote host](http://en.wikipedia.org/wiki/Secure_Shell). It still holds true that the console equivalent usually works just as well, runs faster, is easier to manage once configured without the need of reaching for a mouse and shuffling around windows, and generally [sucks less](http://suckless.org/manifest/).

I've always used [pine](http://en.wikipedia.org/wiki/Pine_(e-mail_client)), and later [alpine](http://en.wikipedia.org/wiki/Alpine_(e-mail_client)), as a mail client due to its overwhelming popularity and ease of use. The downfall is its [imap](http://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) support is less than desirable, and [ssl certificates](http://en.wikipedia.org/wiki/Secure_Sockets_Layer) require a manual install, no prompt to retain. Archaic. While alpine does include support for message filtering, they have to be run manually. Not an ideal solution for someone who requires sorted imap folders from remote devices. I'm a curmudgeon and resistant to change for things that are not necessarily broken. Since I was setting up [imapfilters](https://github.com/lefcha/imapfilter) to handle the sorting, and finally took the time to try out [mutt](http://www.mutt.org/). I've always heard good things about the client, and it carries the author quoted philosophy I can relate to.

First things first, I still float between [joe](http://joe-editor.sourceforge.net), [bsd vi](http://en.wikipedia.org/wiki/Vi), and [vim](http://en.wikipedia.org/wiki/Vim_(text_editor)) as text editors, though have been drifting to vim lately as it's the de facto editor in my current employers environment. It was only logical to set it as the default editor for mutt.

Install vim and correct its backspace to the expected behavior of delete character, instead of just moving the cursor.

> pkg_add -r vim<br>


Now for the sake of insecurity and simplicity, recompile openssl from the ports with "Build with [MD2 hash (obsolete)](http://en.wikipedia.org/wiki/MD2_(cryptography)) legacy support as it will error without it. You will most likely need to reconfigure the installation options for the port as it has either already been configured without, or never configured at all from installing from a package. Also confirm you have cyrsus-sasl2 installed.

> cd /usr/ports/security/openssl ; make config<br>
> make deinstall ; make install clean<br>
> pkg_add -r cyrus-sasl2

Install mutt and aspell, unless your a spelling wizard or like the clunky ispell. Configure mutt as you like, but I prefer the mutt-devel port with the sidebar patch, and cyrus-sasl2 enabled from the configuration menu.

> cd /usr/ports/mail/mutt-devel ; make install clean<br>
> pkg_add -r aspell

Configure mutt to use your imap account and create cache directories. Please note my configuration may be horribly malformed, redundant, and stupid. Please also note I have only skimmed the manpages for the software, and this functions for me, though I am an idiot.

> mkdir ~/.mutt ; mkdir ~/.mutt/cache

> set from = "your.email@your-stupid-domain.com"<br>
> set realname = "John doe"<br>
> set imap_user = "your.email@your-stupid-domain.com"<br>
> set folder = "imaps://mail.your-stupid-domain.com:993"<br>
> set spoolfile = "+INBOX"<br>
> set postponed ="+[INBOX]/Drafts"<br>
> set copy = yes<br>
> set record="imaps://mail.your-stupid-domain.com/INBOX/Sent"<br>
> set header_cache =~/.mutt/cache/headers<br>
> set message_cachedir =~/.mutt/cache/bodies<br>
> set certificate_file =~/.mutt/certificates<br>
> set smtp_url = "smtps://your.email@your-stupid-domain.com@mail.your-stupid-domain.com:465"<br>
> set ssl_force_tls = yes<br>
> set sort = threads<br>
> set editor = vim<br>
> set ispell="aspell -e -c"<br>
> set timeout=15
> set mail_check=600<br><br>
> \# set up the sidebar, default not visible<br>
> set sidebar_width=24<br>
> set sidebar_visible=yes<br>
> set sidebar_delim='|'<br>
> set sidebar_sort=yes<br><br>
> \# which mailboxes to list in the sidebar<br>
> set imap_check_subscribed=yes<br>
> \# color of folders with new mail<br>
> color sidebar_new yellow default<br><br>
> \# ctrl-n, ctrl-p to select next, prev folder<br>
> \# ctrl-o to open selected folder<br>
> bind index \CP sidebar-prev<br>
> bind index \CN sidebar-next<br>
> bind index \CO sidebar-open<br>
> bind pager \CP sidebar-prev<br>
> bind pager \CN sidebar-next<br>
> bind pager \CO sidebar-open<br><br>
> color hdrdefault yellow black<br>
> color quoted white black<br>
> color signature green black<br>
> color attachment red black<br>
> color message brightred black<br>
> color error brightred black<br>
> color indicator black red<br>
> color status brightgreen blue<br>
> color tree white black<br>
> color normal white black<br>
> color markers red black<br>
> color search white black<br>
> color tilde brightmagenta black<br>
> color index blue black ~F<br>
> color index red black "~N|~O"

Now on to [imapfilter](https://github.com/lefcha/imapfilter). It is a [lua](http://www.lua.org/) application, which is a bit annoying. It's the only reason I have lua (a "fad" programming language that will fall to the side) installed on my system. Unfortunately there is no package or port available. Extract the tarball and edit the configuration file to reflect where the required libraries and includes headers reside on FreeBSD. Change:

> incdirs="-I/usr/local/include"<br>
> libdirs="-L/usr/local/lib"

to:

> incdirs="-I/usr/local/include -I/usr/local/include/lua51"<br>
> libdirs="-L/usr/local/lib -L/usr/local/lib/lua51"

Now run "./configure", then "make", then "make install", hopefully  without issue.

Set up the directory and configuration file.

> mkdir ~/.imapfilter ; touch ~/.imapfilter/config.lua

Add the relevant account information, and hack out your filters. Review the imapfilter_config manpage for the desired sorting variables. A basic configuration would be the following.

> account1 = IMAP {<br>
> server = 'mail.your-stupid-domain.com',<br>
> username = 'your.email@your-stupid-domain.com',<br>
> password = 'your email password',<br>
> ssl = 'ssl3'<br>
> }<br><br>
> deletespam = account1.INBOX:contain_subject('SPAM')<br>
> account1.INBOX:delete_messages(deletespam)<br><br>
> filter<br><br>
> another filter<br><br>
> another filter<br><br>
> and more filters.

You just committed an evil sin. Email password saved to a file in plain text. Never recommended, much less on a multi user system. Please confirm your file permissions, and keep in mind any computer powered on is not to be trusted.

All that is left is to install a cronjob to have imapfilter shuffle your messages every ten minutes.

> crontab -e

and install the cronjob:

> */10 * * * *  /usr/local/bin/imapfilter -c /home/username/.imapfilter/config.lua > /dev/null 2>&amp;1

Now we have a lovely functional console replacement for thunderbird that looks like this:

<a href="/img/mutt2.png"><img align="left" title="mutt" src="/img/mutt1.png" alt="mutt width="300" height="225" /></a>
