---
title: "Centralized syslog and storage"
description: "Centralized syslog, elk, and long term storage"
date: "2016-11-09T11:24:00-07:00"
---

Logging data is good. Centralized logging is better. Making data available to a broad user base within your organization is critical. Several fantastic solutions exist (splunk, elk stack, graylog2), but implementing technology is not the end of the conversation. Policies and processes will dictate how logs are manipulated and archived. What type of data has value long term? Do you only need authentication logs for triage and incident response? Should kernel and custom application messages be stored longer to track timelines of overall degraded performance? 

There is no "one size fits all" answer. Every business, environment, and use case is different. Policies may not be in place or change at a later date. Rather than rebuilding every time business requirements change, architect systems that cover all the bases. Make it flexible enough to scale moving forward. Bonus points if it's simple and well documented.

While there is no simple open source solution that does everything out of the box, there are plenty of tools available to piecemeal a system that works. I see pleasant harmony between classic syslog daemons and modern ingest stacks. Some of the older daemons can handle a large volume of data without issue. These can be used to aggregate, buffer, and replicate while feeding to other ingests processes and writing raw logs to disk. This increases complexity, but I think the benefits outweigh the risk.

<a href="/img/syslog_l.png"><img align="left" title="syslog" src="/img/syslog_s.png" alt="syslog" width="360" height="159" /></a>I'm using rsyslog as a forwarder because it's the default in the fleet I currently manage, though any of the modern implementations should do the trick. Rsyslog is also the initial ingest process within my centralized logging system. It forwards a copy of all messages to logstash and writes logs to disk. I'm now ready to check a box for whenever the policy or compliance requirements trickle through. If logstash was to parse the incoming data for output to disk, it would write in json unless its passed through another codec. By storing raw logs elsewhere long term, I can keep a shorter period of log data in elasticsearch, speeding up queries and reducing the number of nodes in the cluster. 

With this setup I retained the ability to do longer maintenance downstream. I can issue rolling restarts of elasticsearch without dropping the cluster. I've configured rsyslogd clients with a local queue and forward with tcp. I prefer to put a little bit of pressure across the fleet instead of a single process having to buffer the collective queue. This enables me to stop the centralized rsyslog ingest process with  data buffered client side until availability is restored.

Flexiable solution implemented. Check the box. Walk away.
