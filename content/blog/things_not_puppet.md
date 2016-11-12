---
title: "Things not to do with puppet"
date: "2012-09-25T16:47:00-07:00"
slug: "things-not-to-do-with-puppet"
---

I've recently had the pleasure to begin rolling out [Puppet](http://puppetlabs.com/puppet/what-is-puppet/) in various network enclaves. This can be both inviting and intimidating. On one hand, a [sysadmin](http://en.wikipedia.org/wiki/System_administrator) can see the light at the end of the tunnel, stepping closer to job automation. Puppet wears many hats, enabling easy change management for a large number of systems while functioning as an accurate time-line to resolve compliance audits at a moments notice. On the other hand, being tasked with writing [manifests](http://docs.puppetlabs.com/learning/manifests.html) to define systems and their configurations across your employers environment could easily go sour. A simple change that should not have been made uniform or a tweak to correct a previous blunder can break application work flows. It's a fine line between harnessing complexity, setting yourself adrift on an iceberg, or being thrown under the [devops](http://en.wikipedia.org/wiki/DevOps) bus.

Unfortunately I've always managed environments that configuration management was either not needed, needed and not used, or configuration automation fell outside of my job description. Flat environments, other software and patch management products, and miscellaneous unspeakable horrors have kept me years late to the party.

[Learning to use puppet](http://docs.puppetlabs.com/learning/) is a relativity painless procedure, assuming you understand a few caveats. While most sadmins can hack out anything to a functional state quickly and deal with the implementation issues later, puppet is not a tool to rush. It demands some trail and error, forcing one to look at problems as an engineer instead of an implementor. Manifest, class and module design flaws can come back to haunt you, hitting scalability walls or simply becoming just as unmanageable as the five hundred stand alone systems were before puppet.

While drafting a game plan with ease and scalability in mind, I ran through several wrongs before I hit the right [$PATH](http://en.wikipedia.org/wiki/PATH_(variable)) (shell humor). Luckily I'm not on my own and have peers to prevent me from destroying my future.

So lets look at some of my poor choices.

*Don't test until you have written a handful of complete manifests*.

My first run failed because I defined a baseline configuration as a module and tried to define nodes by inheriting the module. You can't inherit modules. A basic error which I would have walked on if I had tested from the start. Your organization should have a development environment. Use it, or use [virtualize](https://www.virtualbox.org/) your own.  I required access to resources available only through my hosts [vpn](http://en.wikipedia.org/wiki/Virtual_private_network) connection, yet I needed the machines to be able to talk to each other, and I want to be able to shell them. An easy solution is to create the machines with the first network interface natted, add in a virtual network in [VirtualBox](https://www.virtualbox.org/), add a second nic to each machine, configure the second nic as "host-only" and assign static ip's. Take a snapshot of the puppet client vm after it has the client side software running, and its certificate is signed in the server vm.

*Try to design monolithic manifests*.

I'm a simplest by nature. I drift towards clean solutions. I don't care if its a bit too heavy, lacking a few features, or a little behind the curve if it provides easy management and comprehension. The monolithic approach to defining objects to be managed by puppet would work in a landscape with A and B variations. Unfortunately this does not apply to the real world and is a giant waste of time. I cornered myself by having to write an entire manifest for every single server class and configuration, hacking the same manifest over and over. The key to not shooting yourself in the foot with manifest, class, and module design is flexibility.

*Write modules to define specific servers before you write a baseline configuration*.

Then go back and restructure everything an hour later so you can write a baseline manifest.

*Write a module to define an object and everything it uses*.

Then go rip out most of it to create more modules when you realize your ntp configuration can be reused in other manifests.

*Don't ask design questions until you have rewritten half of your stuff using a new blueprint*.

Asking questions is stupid.

*Don't use templates*.

Who wants to manage a single dynamic configuration with different variables for multiple servers when you can have 30 static configuration files to maintain.

*Use global package variables*.

Prevent yourself from ever using "ensure =&gt; absent" by using a global "ensure =&gt; installed". Removing software is counterproductive.

*Make modules for everything that doesn't carry a file or template*.

Spread love for endless nested directories.

Where did this trail (by fire) and error leave me?

With a simple framework built to withstand scalability challenges and endless requirement updates.

A classic /etc/puppet structure on the puppetmaster looks like this:
<pre>.
├── auth.conf
├── fileserver.conf
├── manifests
│   ├── baseline.pp
│   ├── classes
│   │   ├── accounts.pp
│   │   ├── dbserver.pp
│   │   ├── git.pp
│   │   ├── ircserver.pp
│   │   ├── kerberos.pp
│   │   ├── kvmserver.pp
│   │   ├── lamp.pp
│   │   ├── splunk.pp
│   │   ├── vm.pp
│   │   └── webserver.pp
│   ├── enclave1.pp
│   └── site.pp
├── modules
│   ├── motd
│   │   ├── files
│   │   │   └── motd
│   │   └── manifests
│   │       └── init.pp
│   ├── nrpe
│   │   ├── manifests
│   │   │   └── init.pp
│   │   └── templates
│   │       └── nrpe.conf.erb
│   └── ntp
│       ├── manifests
│       │   └── init.pp
│       └── templates
│           └── ntp.conf.erb
└── puppet.conf</pre>
We start with defining a baseline of universal traits all systems share. We write classes and modules for items that may not belong everywhere or if we just want to reuse it in other enclaves. An example baseline.pp file:
<pre>class baseline {
  include accounts
  include motd
  include nrpe
  include ntp</pre>

<pre>  package { 'rsync':
    ensure => 'installed'
  }</pre>

<pre>  package { 'sendmail':
    ensure => 'installed'
  }</pre>

<pre>  package { 'openssh-clients':
    ensure => 'installed'
  }</pre>

<pre>  package { 'bind-utils':
    ensure => 'installed'
   }</pre>

<pre>  service { 'puppet':
    enable => true,
    ensure => running,
  }</pre>

<pre>  service { 'ip6tables': 
    ensure => stopped,
    enable => false,
  }</pre>

An example site.pp file:
<pre>import "baseline"
import "classes/\*"
import "enclave1"

filebucket { main: server => puppet }
File { backup => main }</pre>
An example enclave1.pp file:
<pre>node baseline {
}
node 'server1.domain.com' inherits baseline {
}
node 'server2.domain.com' inherits baseline {
     include lamp
}
node 'server3.domain.com' inherits baseline {
     include ircserver
     include git
     include dbserver
}
node 'server4.domain.com' inherits baseline {
     include kvmserver
}</pre>
In this example everything inherits the traits of the baseline configuration, and we are free to include various module and classes to add functionality while maintaining flexibility. Not painted into a corner. Now go have fun creating modules, template configuration files, and write classes to define everything in your environment until you automate yourself out of a job.
