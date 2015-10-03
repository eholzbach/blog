---
title: "Writing modular puppet modules"
date: "2013-04-16T21:41:00-07:00"
slug: "writing-modular-puppet-modules"
---

A problem I quickly ran into was controlling module behavior dependent on hostname in a scalable fashion. I burned through several failed attempts at implementing this in the module manifest using various "if" statements on <a href="https://puppetlabs.com/puppet/related-projects/facter/" target="_blank">facter</a> variables. None of them were satisfactory. Luckily i floated back to puppet labs training documentation and found a modular approach was available. I then and smacked myself in the face for overlooking it from the start. Twice.

This example relies on having <a href="http://ericholzbach.net/blog/2012/09/things-not-to-do-with-puppet/">individual hosts listed as described in my previous post here</a>. Having a list of hosts does require a bit of maintenance, but text file manipulation using standard *nix user-land utilities makes it a rather painless process. It allows a modular approach to writing and controlling modules that's endlessly useful. After trial and failure, I was able to write a single module to define all types of hosts that are related as the building block of a specific role.

In this example lets pretend we have an application that requires a web server, a file server, a git repository, and a database server to run. I already have baseline configurations written for these common devices, but for this application they need to be altered in some way that does not fit in my normal mold. Defining these oddballs into the baseline configuration manifests is a failure. Changing the configuration on 100 database servers to meet the needs of a handful of systems is also a failure. Writing a puppet module for "custom application number one" wins.

The module structure looks like this:
<pre>squirrel_generator
├── files
│   ├── nuts_a
│   └── berrys_b
├── manifests
│   ├── dbserver.pp
│   ├── fileserver.pp
│   ├── gitserver.pp
│   ├── init.pp
│   └── webserver.pp
└── templates
    ├── gray_squirrel.erb
    └── red_squirrel.erb</pre>
We now have everything needed to generate a squirrel, that is unique to squirrel generation, within our module. You may also note that our init.pp is not monolithic. Our odd ball servers that have been altered to generate squirrels are defined in their own puppet pattern files. This allows our application to scale, with the added bonus of easier to read the squirrel blueprints.

The init.pp that you usually cram with far too much information is almost empty, thought it's a great place to store definitions that are common to all hosts within your module. Thank you, segmentation.
<pre>class squirrel_generator {
 \# We had all sorts of super secret crap to have servers fart out squirrels,
 \# but it turns out it belonged in other files. Sorry.</br>
 include half_assed_apology</br>
  file {'nuts_a':
    ensure => file,
    source => 'puppet:///squirrel-generator/nuts_a',
    path   => '/usr/home/squirrelmaster/nuts_a',
    owner  => root,
    group  => wheel,
    mode   => '0600',
  }
} </pre>

The module defaults to a half assed apology for filling a space with squirrels and providing nuts to the squirrel master on all nodes, though it alone performs no other function. We can safely include the squirrel_generator module on all associated hosts without worrying about undesirable actions.

Unfortunately we need functionality and we need squirrels.

With our servers listed, we can include the module and define the roll within the module the host should perform.
<pre>node baseline {
}
node 'squirrel-web.domain.com' inherits baseline {
     include squirrel_generator::webserver
}
node 'squirrel-db1.domain.com' inherits baseline {
     include squirrel_generator::dbserver
}
node 'squirrel-db2.domain.com' inherits baseline {
     include squirrel_generator::dbserver
}
node 'squirrel-git.domain.com' inherits baseline {
     include squirrel_generator::gitserver
}
node 'squirrel-fs.domain.com' inherits baseline {
     include squirrel_generator::fileserver
}</pre>
All these rolls are intertwined as unique pieces of the squirrel generating puzzle. We require two database servers of known sciuridae, a file server to provide squirrel generation blueprints, a git repository to maintain changes, and a web server.

Now you have an abundance of small forest creatures all built with a <a href="https://puppetlabs.com/puppet/what-is-puppet/" target="_blank">puppet</a> module. You're welcome.
