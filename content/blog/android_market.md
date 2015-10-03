---
title: "Android Market. Holy f*ck its APT!"
date: "2011-05-21T18:09:14-07:00"
slug: "android-market-holy-fck-its-apt"
---

That's right. The [android OS market](https://market.android.com/), now known as ["advanced persistent threat"](http://en.wikipedia.org/wiki/Advanced_Persistent_Threat).

The buzz word from five years ago that basically defines "the internet". This can also be defined as "an "intelligent" threat that is not going away, no matter how many safeguards we put in place". One could also define this as "the way things have always been, and the way they will always be". Unfortunately "everything is normal" does not get enough media attention to spur larger budgets to buy things you don't need to pretend your mitigating threats you can't actually reduce.

By definition in the information security world, a threat is the malicious act, a vulnerability is a flaw without a safeguard in place, and risk is the possibility of said threat exploiting said vulnerability. Right. Infosec one zero one. Yet people have gone crazy posting all types of nonsense about malicious applications on the android marketplace. A market where any developer can post binaries without being audited before they are made publicly available.

What do you expect to happen? What could go wrong? People abusing the system  by posting applications that steal your financial information and reinvent your identity for someone on the other side of the world to take out a hefty bank loan? Jacked credentials to gain access into your company? Make your device take part in an array of fraud schemes to rack up your phone bill and siphon money from your pockets?

No. It can't be.

Oh wait, it's already been happening on all the major smartphone platforms for years. Symbian is a mess, applications have been routinely removed from the fruit companies product store, and malware authors didn't even have to try to be creative for windows mobile devices. Maybe I shouldn't be surprised.

The use of android based devices has grown exponentially. Malware writers target popular devices. [No news there](http://www.informationweek.com/news/229500572). A robust platform with the money and development community behind it to tackle the fruit phone fad which exploded years ago. Yes, I'm biased, but I will give them due credit for the innovation of changing the way we interact with web browsers on smartphone platforms. Thank you, fruit company.

While google is fast to remove reported malicious applications, this problem is never going to ease up. The implementation of an automated sandbox testing environment could prevent the arrogance of initial exploitation and information leakage immediately at install, though it will do nothing long term. A payload that [doesn't still contain the strings](http://www.androidpolice.com/2011/03/01/the-mother-of-all-android-malware-has-arrived-stolen-apps-released-to-the-market-that-root-your-phone-steal-your-data-and-open-backdoor/) of the [root exploit](http://dtors.org/2010/08/25/reversing-latest-exploid-release/) that waits a few weeks to run, will go undetected. 

Google has responded to rouge applications in a similar manor to how the [FBI took down the coreflood botnet](http://perimeterusa.com/blog/fbi-takes-down-coreflood-botnet-but-many-companies-remain-vulnerable/). They pinged out the infected devices, and [removed the malicous applications themselves](http://techcrunch.com/2011/03/05/android-malware-rootkit-google-response/), without notice, warning, or permission from the device owners. While it may be for the greater good, it brings up a nightmare of ethics, privacy, digital rights, big brother, and various control debates.

The more recent posting which struck me as down right stupid is an article titled [99% of Android Devices Harbor Authentication Flaw on Open WiFi Networks](http://www.eweek.com/c/a/Security/99-of-Android-Devices-Harbor-Authentication-Flaw-on-Open-WiFi-Networks-362697/). The title alone should induce at least a hardy laugh. Any traffic that passes on any open network is vulnerable. Any traffic on encrypted networks using the common protocols is [usually vulnerable to the same types of man in the middle attacks](http://ericholzbach.net/blog/2011/01/https-password-sniffing-on-freebsd/).

To quote the article with the only information which almost makes it relevant, "The tokens aren't bound to a session or a device, making it very easy for the attacker to impersonate the user with a different handset". You steal the token, you authenticate from whatever device you like without the need for credentials. No token manipulation or session id required.

Unfortunately most cookies are not bound to anything, and the ones that are can be easily manipulated. This old attack vector of session hijacking was very popular in the news with the publication of [firesheep](http://codebutler.com/firesheep), an automated tool in the form of a firefox plugin. Point and click exploitation always seems to draw a crowd.

Googles fix for this is to [disable plain text authentication](http://www.eweek.com/c/a/Security/Google-Silently-Patches-Android-Authentication-Flaw-837349/), requiring the use of [ssl/tls](http://en.wikipedia.org/wiki/Transport_Layer_Security). This s/could/should have been common practice for the s/internet/cloud/ with its adoption in the late 1990's thanks to [netscape](http://en.wikipedia.org/wiki/Netscape), yet fifteen years later progress has been slow. The majority of the most popular sites on the internet which firesheep takes advantage of are exploitable due to the legacy concept of using cryptographic authentication, then reverting the session back to clear text, to reduce the overhead of processing power required. This is no longer necessary. Gmail switched to https by default, with [little impact on their network](http://www.imperialviolet.org/2010/06/25/overclocking-ssl.html).

All this aside, it's a moot point if you roam around allowing your mobile device to connect to untrusted networks. Most people leave applications running which automagically check your email, get your stock quotes, or grab the latests updates from social networking sites, spewing your credentials and personal information through unknown hosts.

The problem is the fundamental structure of the current protocols in place. You may trust the server at google where your gmail account is housed, though you can't predict if something on the route between your device and the destination has been compromised.

Everything is flawed on an untrusted network, and networks are not to be trusted.
