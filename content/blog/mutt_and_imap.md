---
title: "Using mutt with imap and imapfilter on FreeBSD"
date: "2011-07-08T01:19:00-07:00"
slug: "using-mutt-with-imap-and-imapfilter-on-freebsd"
---

I generally prefer to use <a href="http://en.wikipedia.org/wiki/Console_application" target="_blank">console applications</a> over anything with a <a href="http://en.wikipedia.org/wiki/Graphical_user_interface" target="_blank">graphical user interface</a>. I picked up this habit years ago from the desire to stuff everything I use in a <a href="http://en.wikipedia.org/wiki/GNU_Screen" target="_blank">screen session</a>, accessible from a <a href="http://en.wikipedia.org/wiki/Secure_Shell" target="_blank">remote host</a>. It still holds true that the console equivalent usually works just as well, runs faster, is easier to manage once configured without the need of reaching for a mouse and shuffling around windows, and generally <a href="http://suckless.org/manifest/" target="_blank">sucks less</a>.

I've always used <a href="http://en.wikipedia.org/wiki/Pine_(e-mail_client)" target="_blank">pine</a>, and later <a href="http://en.wikipedia.org/wiki/Alpine_(e-mail_client)" target="_blank">alpine</a>, as a mail client due to its overwhelming popularity and ease of use. The downfall is its <a href="http://en.wikipedia.org/wiki/Internet_Message_Access_Protocol" target="_blank">imap</a> support is less than desirable, and <a href="http://en.wikipedia.org/wiki/Secure_Sockets_Layer" target="_blank">ssl certificates</a> require a manual install, no prompt to retain. Archaic. While alpine does include support for message filtering, they have to be run manually. Not an ideal solution for someone who requires sorted imap folders from remote devices. I'm a curmudgeon and resistant to change for things that are not necessarily broken. Since I was setting up <a href="https://github.com/lefcha/imapfilter" target="_blank">imapfilters</a> to handle the sorting, and finally took the time to try out <a href="http://www.mutt.org/" target="_blank">mutt</a>. I've always heard good things about the client, and it carries the author quoted philosophy I can relate to.

First things first, I still float between <a href="http://joe-editor.sourceforge.net/" target="_blank">joe</a>, <a href="http://en.wikipedia.org/wiki/Vi" target="_blank">vi</a>, and <a href="http://en.wikipedia.org/wiki/Vim_(text_editor)" target="_blank">vim</a> as text editors, though have been drifting to vim lately as it's the de facto editor in my current employers environment. It was only logical to set it as the default editor for mutt.

Install vim and correct its backspace to the expected behavior of delete character, instead of just moving the cursor.
<blockquote>pkg_add -r vim
echo "set backspace=start" &gt;&gt; ~/.vimrc</blockquote>
Now for the sake of insecurity and simplicity, recompile openssl from the ports with "Build with <a href="http://en.wikipedia.org/wiki/MD2_(cryptography)" target="_blank">MD2 hash (obsolete)</a>" legacy support as it will error without it. You will most likely need to reconfigure the installation options for the port as it has either already been configured without, or never configured at all from installing from a package. Also confirm you have cyrsus-sasl2 installed.
<blockquote>cd /usr/ports/security/openssl ; make config
make deinstall ; make install clean
pkg_add -r cyrus-sasl2</blockquote>
Install mutt and aspell, unless your a spelling wizard or like the clunky ispell. Configure mutt as you like, but I prefer the mutt-devel port with the sidebar patch, and cyrus-sasl2 enabled from the configuration menu.
<blockquote>cd /usr/ports/mail/mutt-devel ; make install clean
pkg_add -r aspell</blockquote>
Configure mutt to use your imap account and create cache directories. Please note my configuration may be horribly malformed, redundant, and stupid. Please also note I have only skimmed the manpages for the software, and this functions for me, though I am an idiot.
<blockquote>mkdir ~/.mutt ; mkdir ~/.mutt/cache</blockquote>
My possiably horriable .muttrc file. Correct as required/desired.
<blockquote>set from = "your.email@your-stupid-domain.com"
set realname = "John doe"
set imap_user = "your.email@your-stupid-domain.com"
set folder = "imaps://mail.your-stupid-domain.com:993"
set spoolfile = "+INBOX"
set postponed ="+[INBOX]/Drafts"
set copy = yes
set record="imaps://mail.your-stupid-domain.com/INBOX/Sent"
set header_cache =~/.mutt/cache/headers
set message_cachedir =~/.mutt/cache/bodies
set certificate_file =~/.mutt/certificates
set smtp_url = "smtps://your.email@your-stupid-domain.com@mail.your-stupid-domain.com:465"
set ssl_force_tls = yes
set sort = threads
set editor = vim
set ispell="aspell -e -c"
set timeout=15
set mail_check=600

# set up the sidebar, default not visible
set sidebar_width=24
set sidebar_visible=yes
set sidebar_delim='|'
set sidebar_sort=yes

# which mailboxes to list in the sidebar
set imap_check_subscribed=yes
# color of folders with new mail
color sidebar_new yellow default

# ctrl-n, ctrl-p to select next, prev folder
# ctrl-o to open selected folder
bind index \CP sidebar-prev
bind index \CN sidebar-next
bind index \CO sidebar-open
bind pager \CP sidebar-prev
bind pager \CN sidebar-next
bind pager \CO sidebar-open

color hdrdefault yellow black
color quoted white black
color signature green black
color attachment red black
color message brightred black
color error brightred black
color indicator black red
color status brightgreen blue
color tree white black
color normal white black
color markers red black
color search white black
color tilde brightmagenta black
color index blue black ~F
color index red black "~N|~O"</blockquote>
Now on to <a href="https://github.com/lefcha/imapfilter" target="_blank">imapfilter</a>. It is a <a href="http://www.lua.org/" target="_blank">lua</a> application, which is a bit annoying. I think it's the only reason I have lua (a "fad" programming language that will fall to the side) installed on my system. Unfortunately there is no package or port available. Extract the tarball and edit the configuration file to reflect where the required libraries and includes headers reside on FreeBSD. Change:
<blockquote>incdirs="-I/usr/local/include"
libdirs="-L/usr/local/lib"</blockquote>
to:
<blockquote>incdirs="-I/usr/local/include -I/usr/local/include/lua51"
libdirs="-L/usr/local/lib -L/usr/local/lib/lua51"</blockquote>
Now run "./configure", then "make", then "make install", hopefully  without issue.

Set up the directory and configuration file.
<blockquote>mkdir ~/.imapfilter ; touch ~/.imapfilter/config.lua</blockquote>
Add the relevant account information, and hack out your filters. Review the imapfilter_config manpage for the desired sorting variables. A basic configuration would be the following.
<blockquote>account1 = IMAP {
server = 'mail.your-stupid-domain.com',
username = 'your.email@your-stupid-domain.com',
password = 'your email password',
ssl = 'ssl3'
}

deletespam = account1.INBOX:contain_subject('***SPAM***')
account1.INBOX:delete_messages(deletespam)

filter

another filter

another filter

and more filters.</blockquote>
You just committed an evil sin. Email password saved to a file in plain text. Never recommended, much less on a multi user system. Please confirm your file permissions, and keep in mind any computer powered on is not to be trusted.

All that is left is to install a cronjob to have imapfilter shuffle your messages every ten minutes.
<blockquote>crontab -e</blockquote>
and install the cronjob:
<blockquote>*/10 * * * *  /usr/local/bin/imapfilter -c /home/username/.imapfilter/config.lua &gt; /dev/null 2&gt;&amp;1</blockquote>
Now we have a lovely functional console replacement for thunderbird that looks like this:
<p style="text-align: center;"><a href='./images/mutt2.png'><img src="./images/mutt1.png"/></a>
</p>
