---
date: "2011-10-17T01:18:00-07:00"
title: "A brief analysis of a tor exit node"
slug: "a-brief-analysis-of-a-tor-exit-node"
---

There is no doubt of the intentions of the tor project, though I've always wondered who most commonly uses it, and what kinds of traffic are they shuffling through it. In the event you're are unfamiliar with the [tor project](http://www.torproject.org/), it is a relay of encrypted proxies you may pass your network traffic through with the hopes of maintaining anonymity. You may find an in depth explanation of [what tor is, how it works, and why it exist, here](http://www.torproject.org/about/overview.html.en).

I recently set up a [tor exit node](https://gitweb.torproject.org/tor.git/blob_plain/HEAD:/contrib/tor-exit-notice.html) for analysis of what typically happens on the network. This is in no way meant to be considered conclusive data of its typical activities. There are far too many variables in a distributed network to accept any of this data as the status quo. Geographic location, throughput, bandwidth, and political boundaries all influence the shape of an exit node. A user of tor may configure what country their traffic stays in, and where it may exit. A tor user in the united states who only wishes to avoid [dmca](http://en.wikipedia.org/wiki/Digital_Millennium_Copyright_Act) complaints from their isp for seeding torrents of pirated movies are  more likely to configure their client to only pass through relays within the same country for the fastest network performance. A typical user in a country who is censored by their government and has no [freedom of speech or rights to information](http://en.wikipedia.org/wiki/Censorship_in_the_People's_Republic_of_China) is more likely to configure their client to pass through unrestricted countries.

My original intention was to log traffic for a period of time, then provide basic results of what the source was passing to the destination on exit, and provide various statistics on traffic relayed to other nodes. After reviewing the packet captures it became clear this could be damaging, providing data for hostile entities to attempt to correlate time and address information. Unfortunately tor doesn't protect your data's privacy from exit node operators or restrictive governments, as it's not an end to end solution.  I've posted a minimal amount of raw communication data to maintain confidentiality, and not undermine the tor project.

This was a loose project. The tor service was periodically restarted to adjust bandwidth limits. I used tcpdump to collect full packet captures. I logged in and manually restarted every other day or so in attempt to maintain manageable file sizes and confirm service functionality. These files were later consolidated with editcap to be broken back down by time. Keeping solid statistics on bandwidth or connection time was outside the scope of my interest.


**Network data and statistics**


<a href="/img/master-protocol1.png"><img class="alignleft size-medium wp-image-658" title="master-protocol1" src="/img/master-protocol1-300x225.png" alt="Tor traffic - protocol" width="300" height="225" /></a> Here is a basic plot of the traffic passed through the node displayed by protocol. A point on the graph was charted every 60 seconds for the full duration of the capture. As you can see, the majority of traffic is tcp bound, with a few noted events.

Capinfos provided us with information on the consolidated file we are working with:
<blockquote>
<pre>File name: master.pcap
File type: Wireshark/tcpdump/... - libpcap
File encapsulation: Ethernet
Packet size limit: file hdr: 96 bytes
Packet size limit: inferred: 96 bytes
Number of packets: 227808045
File size: 21911493703 bytes
Data size: 133490806403 bytes
Capture duration: 1725844 seconds
Start time: Thu Sep 15 21:53:02 2011
End time: Wed Oct 5 21:17:06 2011
Data byte rate: 77348.15 bytes/sec
Data bit rate: 618785.18 bits/sec
Average packet size: 585.98 bytes
Average packet rate: 132.00 packets/sec
SHA1: d60e2b0d04432873c18004a862e98ce94a610580
RIPEMD160: bb9a256b2a0c299a0757e0fbd51167f72d59db25
MD5: 4f4b779eca2f7b0919f311a6101a9f4d
Strict time order: True</pre>
</blockquote>

<a href="/img/through00.png"><img class="alignright size-medium wp-image-669" title="through00" src="/img/through00-300x225.png" alt="" width="300" height="225" /></a>A quick break down of the top 20 port destinations. Please note port 25 would have ranked at the top of the list had I had allowed outbound connections from the node. It was blocked after the first few hours to deny spam sources from dumping to open or compromised smtp relays. Several USA based "marketing companies" attempted to push large amounts of data through the node, only to give up days later, likely when their click through rates plummeted.
<blockquote>
<pre>1383530   4443  tor
993860      80  http
528338    3128  http-proxy
307267     445  microsoft-ds
243273    9050  tor/socks
199368   48478  assumed torrent
174804   11899  assumed torrent
166747    8080  http-alt
126980    6881  bittorrent
103494    8118  tor-privoxy
73892    51413  bittorrent
69423      443  https
65862     8909  ultrasurf-http-proxy
65188       25  smtp
60812    48032  assumed torrent
58425    45682  utorrent
53604    41302  torrent
53469     9001  tor
50491    41416  assumed torrent
48019       22  ssh</pre>
</blockquote>
Tor and http traffic beat out torrents for the number of connections made, though I did not do full bandwidth analysis by port. The majority of the traffic was being passed to another relay, despite being an exit node.

**IP address pool and geolocation**

251,634 unique ip addresses passed traffic through the tor node. Based on the 4.3 billion ip v4 address allocation, roughly 0.005858810618519783% of internet communicated through the server during the packet capture. The following list is the number of unique ip addresses, percentage of the total unique ip addresses the country provided, followed by the country name.
<blockquote>
<pre>51874 20.62% United_States
25563 10.16% China
16760  6.66% Japan
9709   3.86% Russian_Federation
9501   3.78% Korea_Republic_of
8885   3.53% United_Kingdom
8122   3.23% Canada
7834   3.11% France
7776   3.09% Germany
7395   2.94% Brazil
5671   2.25% Italy
5437   2.16% India
5397   2.14% Taiwan;_Republic_of_China_(ROC)
4060   1.61% Australia
4049   1.61% Spain
3847   1.53% Poland
3795   1.51% European_Union
3246   1.29% Netherlands
2613   1.04% Sweden
2506   1.00% Turkey
2412   0.96% Ukraine
2235   0.89% Mexico
2201   0.87% Argentina
1733   0.69% Romania
1638   0.65% Czech_Republic
1621   0.64% Hong_Kong
1475   0.59% Bulgaria
1453   0.58% Norway
1426   0.57% Thailand
1426   0.57% Denmark
1395   0.55% Indonesia
1355   0.54% Viet_Nam
1341   0.53% Philippines
1320   0.52% Finland
1296   0.52% Hungary
1264   0.50% Belgium
1255   0.50% Israel
1179   0.47% Portugal
1131   0.45% Colombia
1102   0.44% Malaysia
1076   0.43% Saudi_Arabia
1028   0.41% Greece
1014   0.40% Switzerland
1002   0.40% Chile
956    0.38% Austria
932    0.37% Iran_(ISLAMIC_Republic_Of)
883    0.35% Pakistan
866    0.34% South_Africa
842    0.33% Singapore
727    0.29% Venezuela
722    0.29% Serbia
716    0.28% Egypt
645    0.26% United_Arab_Emirates
636    0.25% Ireland
627    0.25% Lithuania
591    0.23% Reserved
575    0.23% New_Zealand
505    0.20% Croatia_(LOCAL_Name:_Hrvatska)
501    0.20% NOT IN DATABASE
475    0.19% Slovenia
473    0.19% Latvia
432    0.17% Morocco
422    0.17% Slovakia_(SLOVAK_Republic)
377    0.15% Kazakhstan
364    0.14% Algeria
311    0.12% Cyprus
311    0.12% Belarus
310    0.12% Kuwait
309    0.12% Estonia
304    0.12% Peru
303    0.12% Bosnia_and_Herzegowina
277    0.11% Puerto_Rico
270    0.11% Bangladesh
230    0.09% Tunisia
220    0.09% Costa_Rica
218    0.09% Jordan
212    0.08% Ecuador
211    0.08% Macedonia
08    0.08% Panama
203    0.08% Kenya
194    0.08% Uruguay
192    0.08% Georgia
182    0.07% Nigeria
173    0.07% Dominican_Republic
165    0.07% Sri_Lanka
162    0.06% Moldova_Republic_of
155    0.06% Bahrain
149    0.06% Luxembourg
144    0.06% Syrian_Arab_Republic
136    0.05% Cambodia
134    0.05% Armenia
133    0.05% Iceland
127    0.05% Malta
124    0.05% Guatemala
122    0.05% Trinidad_and_Tobago
114    0.05% Nepal
111    0.04% Jamaica
108    0.04% Lebanon
100    0.04% Mauritius
100    0.04% Ghana
100    0.04% Albania
99     0.04% Paraguay
99     0.04% Oman
94     0.04% Senegal
94     0.04% Iraq
93     0.04% Macau
93     0.04% Barbados
92     0.04% Azerbaijan
90     0.04% Sudan
88     0.03% Qatar
87     0.03% El_Salvador
86     0.03% Mongolia
86     0.03% Bolivia
83     0.03% Tanzania_United_Republic_of
80     0.03% Montenegro
79     0.03% Palestinian_Territory_Occupied
73     0.03% Maldives
66     0.03% Bahamas
63     0.03% New_Caledonia
57     0.02% Honduras
55     0.02% Uganda
55     0.02% Nicaragua
55     0.02% Netherlands_Antilles
53     0.02% Antigua_and_Barbuda
53     0.02% Angola
50     0.02% Guam
50     0.02% Brunei_Darussalam
49     0.02% Namibia
49     0.02% Afghanistan
48     0.02% Saint_Vincent_and_The_Grenadines
45     0.02% Cote_D'ivoire
45     0.02% Botswana
43     0.02% Libyan_Arab_Jamahiriya
41     0.02% Cayman_Islands
33     0.01% Cameroon
31     0.01% Lao_People's_Democratic_Republic
29     0.01% Kyrgyzstan
28     0.01% Monaco
27     0.01% Uzbekistan
26     0.01% Madagascar
26     0.01% Gibraltar
25     0.01% Mozambique
24     0.01% Jersey
24     0.01% Haiti
23     0.01% French_Polynesia
22     0.01% Suriname
22     0.01% Bermuda
22     0.01% Belize
21     0.01% Rwanda
19     0.01% Gabon
18     0.01% Fiji
17     0.01% Zambia
17     0.01% Guyana
17     0.01% Faroe_Islands
16     0.01% Mali
13     0.01% Ethiopia
13     0.01% Burkina_Faso
13     0.01% Aruba
12     0.00% Virgin_Islands_(U.S.)
12     0.00% Gambia
11     0.00% Yemen
10     0.00% Turks_and_Caicos_Islands
10     0.00% Togo
10     0.00% Saint_Kitts_and_Nevis
10     0.00% Congo_The_Democratic_Republic_of_The
9      0.00% Isle_of_Man
8      0.00% Virgin_Islands_(BRITISH)
8      0.00% Tajikistan
8      0.00% San_Marino
8      0.00% Guadeloupe
8      0.00% Andorra
7      0.00% Micronesia_Federated_States_of
7      0.00% Lesotho
5      0.00% Seychelles
5      0.00% Palau
4      0.00% Saint_Martin
4      0.00% Niger
3      0.00% Turkmenistan
3      0.00% Guernsey
3      0.00% Congo
2      0.00% Vanuatu
2      0.00% Tuvalu
2      0.00% Reunion
2      0.00% Marshall_Islands
1      0.00% Wallis_and_Futuna_Islands
1      0.00% Swaziland
1      0.00% Solomon_Islands
1      0.00% Samoa
1      0.00% Papua_New_Guinea
1      0.00% Myanmar
1      0.00% Montserrat
1      0.00% Liechtenstein
1      0.00% Greenland
1      0.00% Eritrea
1      0.00% Equatorial_Guinea
1      0.00% Cuba
1      0.00% Cape_Verde
1      0.00% Burundi
1      0.00% British_Indian_Ocean_Territory</pre>
</blockquote>


**Security notes and inbound attack data**


There 515 instances of credentials passed in clear text through this tor relay, 92 were unique. 79 were passed over http, 9 snmp, and 4 pop. There was also a wealth of sensitive data disclosed in url's and cookies. The following list if of web hosts the clear text credentials were forked to. The majority of which are torrent trackers. The frequency of yahoo is due to their web applications passing data in the clear, not a sign of its popularity.
<blockquote>
<pre>02.ah127.in
02.ahr3.in
10.rarbg.com
89.36.26.64
9.rarbg.com
a.adwolf.ru
adult-cinema-network.net
alleenretail.org
amateurallure.com
amazone-tracker.net
announce.torrent.lt
announce.torrentsmd.com
announce.x-torrents.nu
api.rapidshare.com
b.kavanga.ru
balancer.netdania.com
bilola.com
bit-torrent.bz
bitgamer.su
bollywoodtorrents.me
brt-share.com
buhaypirata.net
bwtorrents.com
checkmytorrentip.com
concen.org
dragon-torrents.biz
elbitz.net
exodus.desync.com
fast-limited.org
fatal-beauty.org
femdom-fetish-torrents.org
getiton.com
good73.net
greek-trackr.com
hans-johannes.nl
hdporn-torrent.com
hdworld.zapto.org
ihookup.com
ihookupmail.com
itoma.info
kickass.com
kinofilm.ws
l1.member.vip.in2.yahoo.com
l03.member.aue.yahoo.com
l04.member.ird.yahoo.com
l05.member.sp1.yahoo.com
l06.member.ird.yahoo.com
l10.member.mud.yahoo.com
l12.member.sg1.yahoo.com
l17.member.sp1.yahoo.com
l32.member.mud.yahoo.com
lavache.alinto.com
login.korea.yaho.com
manicomio-share.com
metal-zone.ru
minecraft.net
my-gamebox.com
o-o.preferred.iad09s14.v15.lscache6.c.youtube.com
odusseya.com
p2p.arenabg.ch
p2p.arenabg.com
passport.sohu.com
pow7.com
quebec-team.net
rek.wp.pl
s1.announceserver.com
s1.announceserver.t.net
sextor.org
softwarepaleis.org
soundclick.com
street-friends-partage.com
superseeds.org
t2.pow7.com
thefiles.ro
tj.tbxiangce.com
torrent.2ch.so
torrentpipeline.org
tr0.kinozal.tv
tr1.kinozal.tv
tr2.kinozal.tv
tracker.bitsoup.org
tracker.concen.org
traorrents.org
tracker.prq.to
tracker.publicbt.com
tracker.thailandtorrent.com
tracker.torrentbytes.net
tracker4.supertorrents.org
tractor.flro.org
tunisiagate.net
tv.tracker.prq.to
ultimate-tracker.com
vap1den1.lijit.com
ya.ru</pre>
</blockquote>
I receive s all for horrible content which should never be watched, much less purchased. Save your life. Go outside.
<blockquote>
<pre>NBC Universal Anti-Piracy Technical Operations
 Title: American Pie Presents: The Naked Mile
 Title: American Pie 2
 Title: AmericanayTSP, Inc.
 Infringed Work: How To Train Your Dragon
 Infringed Work: Le Mans
 Infringed Work: Flushed Away (DWA)
 Infringed Work: Transformers
 Infringed Work: Wrong Turn at Tahoe
 Infringed Work: The Last Airbender
 Infringed Work: Get Rich or Die Tryinrent</pre>
</blockquote>
The server hosted no websites besides the [provided html explaining what its function was](https://gitweb.torproject.org/tor.git/blob_plain/HEAD:/contrib/tor-exit-notice.html), therefore received zero legded granularity to view the relentless brute force attacks and various vulnerability and information gathering scans which happens on production servers, with minimal effort to filter out legitimate traffic.

The public facing services were ssh, dns, and h8 login attempts against ssh. These are the top 20, sorted by attempts, ip address and rdns if available.
<blockquote>
<pre>6043 115.41.239.142
5454 212.43.68.10   dns-2.witcom.de
4127 202.108.137.66
3970 115.238.60.16
1192 222.122.161.9  sciencetel.com
  aick.co.kr
                    aciexpress.co.kr
                    asialogis.com
                    pslogis.co.kr
                    emaxbiz.com
                    aciexpress.net
                    doora.co.kr
                    aicworld.co.kr

1086 -130-67.nyc.biz.rr.com
540 84.18.136.164   84-18-136-164.ip.bkom.it
540 61.19.85.154
522 211.118.104.11
424 219.94.154.235
419 83.143.40.122   host-40-122.spray.net.pl
226 60.10.58.156
215 219.235.1.105   host-219-235-1-105.iphost.gotonets.com
208 69.64.710.13.176.114
161 79.189.193.150  ihl150.internetdsl.tpnet.pl
141 1.85.2.72
105 200.117.250.99  host117250099.arnet.net.ar
94 58.18.172.206
86 24.232.23.26     mailer.mercovan.com.ar</pre>
</blockquote>
Apache received some scans for open and vulnerable webnd rdns if available, followed by the top 20 items checked.
<blockquote>
<pre>250 212.27.195.114
153 64.27.7.2      unassigned.calpop.com
 81 88.191.78.78   sd-13330.dedibox.fr
 55 182.48.60.230  www18192u.sakura.ne.jp
 30 93.190.141.145 customer.worldstre.203.62   ns207232.ovh.net
 10 77.221.159.18  77.221.159.18.addr.datapoint.ru
 10 63.247.194.2   mail.o-flynn.com
 10 212.29.174.34  smtp2.alphit.nl

 21 phpmyadmin
 17 deny2
 17 appConf.htm
 20 phpMyAdmin
 16 scripts
 15 pma
 13 mysql
  9 admin
  6 phpmyamyadmin
  5 web
  5 pr0x
  5 phpmyadmin2
  5 phpadmin
  5 myadmin
  5 dbadmin
  4 hp
  4 databaseadmin</pre>
</blockquote>
Reviewing the network graphs you may notice the spike in packets due to an inbound denial of service attack. It was from a single ip.to DoS a tor exit node from a single ip is beyond me.

**Conclusion**

The tor exit node was mostly used for the networks intended purpose. Maintaining anonymity on the internet. Despite relatively minor abuse, there was nothing else going on.r more boring than ever expected. I leave you with a list of government ip addresses taken after the packet capture. At least the though of some shady government performing some shady action, or some rebellious government employee performing a clandestine l is interesting. Unfortunately, they were just using tor to bypass their offices access control list, and hit facebook from work. A sad state of affairs.
<blockquote>
<pre>111.68.97.90.hec.gov.pk.
199.159.99.34.4k.usda.gov.
209-164-157-74.hartford.gov.
Gewait.swa.army.mil.
NS1.navy.mil.
PC140.mtr.noaa.gov.
PC184.mtr.noaa.gov.
alpaca-ii.it.anl.gov.
andromeda.serpro.gov.br.
antares.serpro.gov.br.
apc18.pifss.gov.kw.
app.mt.gov.
bc-cachesvr.gordon.army.mil.
bc.tamc.amedd.army.mil.
bcproxy-dmz-ca-01.ca.sandia.edlewo.impan.gov.pl.
br1-iso.larc.nasa.gov.
carana.tj.rr.gov.br.
centauro.serpro.gov.br.
chyron.house.gov.
clayton.state.gov.
dd1.chs.spawar.navy.mil.
dekalb-emip1.dekalbcountyga.gov.
dekalb-emip2.dekalbcountga.gov.
dekalb-emip2.dekalbcountyga.gov.
dns-s1.b.
dns.sra.gov.cn.
eastmailhub.treas.gov.
ecclerk.erie.gov.
eoffice.mic.gov.vn.
fiber.ameslab.gov.
fisepe.pe.gov.br.
franklin.net.ca.gov.
ftc-ns.fsc.usda.gov.
ftp.receita.fazenda.gov.br.
g203070.chs.spawar.navy.mil.
gate3-norfolk.nmci.navy.mil.
gateway-202l.amedd.army.mil.
generichost.lewis.army.mil.
gsnet.gsn.gov.tw.
h137-191-225-162.gn.gov.ie.
hermes.mpa.gov.sg.
hhh-ns02.ito.hhs.gov.
hide-102.maine.gov.
host-008.itec.al.gov.br.
host.159-142-110-243.gsa.gov.
host.jsc.nasa.gov.
host20.mns.gov.ua.
hq-cbs3.ccudgate.hud.gov.
internet9.irs.gov.
investment.gov.eg.
loadbalancer.honolulu.gov.
lsg.grafenwoehr.army.mil.
mail.camara-arq.sp.gov.br.
mail.fcgo.gov.np.
mail.incancerologia.gov.co.
mail.lmw.vic.gov.au.
mail.micse.gov.ec.
mail.parracity.nsw.gov.au.
mail.quan.bt.
mail2.vst.gov.vn.
mail3.mti.gov.eg.
maileast17.srvs.usps.gov.
mailin1.ssa.gov.
mincom.gov.bn.
mirror.net.cen.ct.gov.
mta.iti.gov.br.
mx.cbi.gov.ar.
mx1.timbo.sc.gov.br.
nat-adsl.azores.gov.pt.
nat.tambov.gov.ru.
natdw.nrel.gov.
nc01.mepcom.army.mil.
nsa.gov.
ns1.pi.gov.br.
ns1.prison.gov.my.
ns1.sc.gov.br.
ns1.ulakbim.gov.tr.
ns2.iti.gov.br.
ns2.pe.gov.br.
ns2.ulakbim.gov.tr.
ns21.belvoir.army.mil.
ns21.dix.army.mil.
ns21.irwin.army.mil.
ns3.ulakbim.gov.tr.
nsc2.nasa.gov.
nt0.rada.gov.ua.
ogero.gov.lb.petra.nic.gov.jo.
photojournal.jpl.nasa.gov.
pop3.riversideca.gov.
ppj-web-1.jpl.nasa.gov.
ppj-web-1x.jpl.nasa.gov.
railnetibwmum.railnet.gov.in.
railteldlimx.railnet.gov.in.
rede17-59.pmf.sc.gov.br.
rede192-58.pmf.sc.gov.br.
relay1.mecon.gov.ar.
reverse.d.org.
sgzero.llnl.gov.
smtp.police.gov.mv.
static-194-158-95-165.govern.ad.
static.greenwoodsc.gov.
stl-ns.fsc.usda.gov.
subnet71.idsc.gov.eg.
sumo-145-228.nitc.gov.np.
sumo-147-126.nitc.gov.np.
sumo-147-68.nitc.gov.np.
sw83131.crytic.mcye.gov.ar.
testeccl.nist.gov.
trifid.serpro.gov.br.
trl.santafe-conicet.gov.ar.
user.plano.gov.
venta.ch.govorit.ru.
vicce016.net.gov.bc.ca.
vicce018.net.gov.bc.ca.
vp51.aph.gov.au.
webproxy1.anl.gov.
webserver.namria.gov.ph.
webvpn.fiscal.ca.gov.
wlevs020.evs.anl.gov.
ws10-3-112.rcil.gov.in.
ws138-195-133-112.rcil.gov.in.
ws234-195-133-112.rcil.gov.in.
ws234-246-252-122.rcil.gov.in.
ws28-195-133-112.rcil.gov.in.
ws34-230-252-122.rcil.gov.in.
ws41-231-252-122.rcil.gov.in.
ws75-234-252-122.rcil.gov.in.
ws98-234-252-122.rcil.go-b.jpl.nasa.gov.
www.gender.gov.zm.
www.jpl.nasa.gov.
www.ncbi.nlm.nih.gov.
www.receita.fazenda.gov.br.
www.rfb.fazenda.gov.br.
www.sefaz.pe.gov.br.
www.upsc.gov.in.</pre>
</blockquote>

<table>
  <tr>
    <th><a href='/img/master-protocol0.png'><img width="150" height="150" src="/img/master-protocol0-150x150.png" class="attachment-thumbnail" alt="master-protocol0" /></a></th>
    <th><a href='/img/master-protocol1.png'><img width="150" height="150" src="/img/master-protocol1-150x150.png" class="attachment-thumbnail" alt="Tor traffic - protocol" /></a></th>
    <th><a href='/img/packet-cut0.png'><img width="150" height="150" src="/img/packet-cut0-150x150.png" class="attachment-thumbnail" alt="packet-cut0" /></a></th>
    <th><a href='/img/packet-cut0.png'><img width="150" height="150" src="/img/packet-cut1-150x150.png" class="attachment-thumbnail" alt="packet-cut1" /></a></th>
  </tr>
  <tr>
    <td><a href='/img/packet-cut2.png'><img width="150" height="150" src="/img/packet-cut2-150x150.png" class="attachment-thumbnail" alt="packet-cut2" /></a></td>
    <td><a href='/img/packet-cut3.png'><img width="150" height="150" src="/img/packet-cut3-150x150.png" class="attachment-thumbnail" alt="packet-cut3" /></a></td>
    <td><a href='/img/packet-cut4.png'><img width="150" height="150" src="/img/packet-cut4-150x150.png" class="attachment-thumbnail" alt="packet-cut4" /></a></td>
    <td><a href='/img/packet-cut5.png'><img width="150" height="150" src="/img/packet-cut5-150x150.png" class="attachment-thumbnail" alt="packet-cut5" /></a></td>
  </tr>
  <tr>
    <td><a href='/img/packet-cut6.png'><img width="150" height="150" src="/img/packet-cut6-150x150.png" class="attachment-thumbnail" alt="packet-cut6" /></a></td>
    <td><a href='/img/packet-cut7.png'><img width="150" height="150" src="/img/packet-cut7-150x150.png" class="attachment-thumbnail" alt="packet-cut7" /></a></td>
    <td><a href='/img/packet-cut8.png'><img width="150" height="150" src="/img/packet-cut8-150x150.png" class="attachment-thumbnail" alt="packet-cut8" /></a></td>
    <td><a href='/img/packet-cut9.png'><img width="150" height="150" src="/img/packet-cut9-150x150.png" class="attachment-thumbnail" alt="packet-cut9" /></a></td>
  </tr>
  <tr>
    <td><a href='/img/through00.png'><img width="150" height="150" src="/img/through00-150x150.png" class="attachment-thumbnail" alt="through00" /></a></td>
    <td><a href='/img/through01.png'><img width="150" height="150" src="/img/through01-150x150.png" class="attachment-thumbnail" alt="through01" /></a></td>
  </tr>
</table>
