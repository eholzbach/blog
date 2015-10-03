---
title: "Who do we trust and why do we trust them?"
date: "2011-09-15T00:21:00-07:00"
slug: "who-do-we-trust-and-why-do-we-trust-them"
---

<a href="http://en.wikipedia.org/wiki/Certificate_authority" target="_blank">Certificate authorities</a>, and because it's currently out best option, despite it being a poor one.

Everyone who uses the internet relies on certificate authorities to <a href="http://en.wikipedia.org/wiki/Authentication" target="_blank">authenticate</a> somewhat important sites they visit. This is done with a model of <a href="http://en.wikipedia.org/wiki/Public-key_cryptography" target="_blank">public key cryptography</a>, combined with a trusted-ish(ed) 3rd party entity acting as an <a href="http://en.wikipedia.org/wiki/Escrow" target="_blank">escrow</a> to delegate, maintain, and revoke cryptographic certificates. The commonly used browsers ship with a list of approved certificate authorities, which is not standardized, but chosen by the software vendor.

For simplicity, I'll provide the following example.

When you visit your banks website through <a href="http://en.wikipedia.org/wiki/HTTP_Secure" target="_blank">https</a> the, server provides your web browser with a <a href="http://en.wikipedia.org/wiki/Security_token" target="_blank">cryptographic token</a>. Your web browser does some magic mathematical wizardry (similar to magnets) to confirm the certificate has been signed by the certificate authority,. If it's approved, your browser will accept it's talking to who it should be talking to, and start negotiating what <a href="http://en.wikipedia.org/wiki/Cipher" target="_blank">cipher</a> it would like to use for communication.

In the event the mathematical wizardry nobody understands (similar to magnets), produces an incorrect signature, your browser should kindly present you with a pop up window asking to review, then accept or deny the ssl certificate. Unfortunately nobody does this as people generally don't understand what it is, why it's there, or how it can help them. Those who do understand are lazy and almost always just accept it anyway.

The authentication provided by ssl certificates and CA vendors are in place so you know your providing your banking credentials to your banks website, and not an impostor who will then sell them, or use them to transfer your balance to someone else's account, then to the local western union. By ignoring a warning from something you typically trust, you are inviting the possibility of a classic <a href="http://en.wikipedia.org/wiki/Man-in-the-middle_attack" target="_blank">man in the middle attack</a>, and the unfortunate ordeal of getting your bank to return the fraudulent transaction(s).

There have been several instances recently of certificate authorities erroneously selling and/or issuing without their knowledge, certificates for fortune 500 companies to customers who are not affiliate with said fortune 500 company. The plus side is the <a href="http://en.wikipedia.org/wiki/Public_key_infrastructure" target="_blank">public key infrastructure</a> model allows keys to be revoked, so we are not stuck with a random entity who owns a 256 byte string of ascii characters that cryptographically says they are google.com. The down side is we are all vulnerable until the breach is discovered as we (the whole internet) relies on these vendors for authentication.

There are reports that the <a href="http://blogs.comodo.com/it-security/data-security/the-recent-ra-compromise/" target="_blank">past</a> few <a href="http://www.comodo.com/Comodo-Fraud-Incident-2011-03-23.html" target="_blank">discovered breaches</a> at the major CA vendors were <a href="http://nakedsecurity.sophos.com/2011/09/05/operation-black-tulip-fox-its-report-on-the-diginotar-breach/" target="_blank">possibly funded by the Iranian government</a>. There is no doubt of desire by governments to monitor it's citizens and spy on the world abroad. A valid ssl certificate for a popular domain demands a large price on the black market due to potential profit generation through exploitation. Particular governments can/will provide funds to obtain illicit certificates to collect intelligence. I'm sure it's just another drop in the bucket of a countries defense budget.

There is a simple explanation of why we are all royally screwed.

The inherent complication is the vendors which our software have chosen to include as valid certificate authorities, are literally just vendors. There is no magic behind the curtains. Despite being a corporate entity which is based on selling a security model and providing associated products and services, they are fundamentally just as flawed and have to mitigate the same broad range of attack vectors and vulnerabilities every corporation is faced with on a daily basis. If we can't keep our own infrastructure secure, how can we feasibly expect someone else to do it for us.

The solution is.. currently none. The problem with any trust implementation is there has to be something that is trusted. Anything that has to be trusted is susceptible to exploitation, causing the entire model to crumble. You have two choices. Rely on a CA for authentication, or rely on yourself. If you use the current model you are faced with the previously stated issues. On the other hand we could place the certificate authority as an "in house" verification process. You could provide the url hosting the other half of the certificate in the same manor as <a href="http://en.wikipedia.org/wiki/Domain_Name_System" target="_blank">domain name servers</a>, though would still be plagued with the same problems. Plus now your putting all your eggs in one basket, aiding the avoidance of discovery of a compromise. Bottom line, you can't win.

The chain is only as strong as it's weakest link. The weakest link we currently rely on is just as weak as the rest of us.

