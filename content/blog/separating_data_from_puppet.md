---
title: "Separating data from code; using hiera in puppet"
date: "2013-11-12T23:53:00-07:00"
slug: "separating-data-from-code-using-hiera-in-puppet"
---

There are a few goals for writing puppet code that wont crush your soul at a later date. Scalability across operating systems and environments, ease of maintenance, and writing "one manifest to rule them all" type code are pretty high on that list. Every time you add

> if $::fqdn == 'unique-flower.yourcubefarm.com' {

to your puppet code, you are creating a condition where future administrators will want to murder you. Rightfully so. Over time your code base will grow from a handful of modules to a novel of state enforcement. The more [conditional statements](http://docs.puppetlabs.com/puppet/3/reference/lang_conditional.html) to support unique flowers, the harder it becomes to write or maintain manifests without similar conditional clauses, not to mention the increased time to spin up new hires. Seriously. Don't mention it.

Unfortunately I have a lot of unique flowers in multiple environments. I have a lot of trash puppet code to support them. I do not wish to be murdered.

It is time to address this.

One solution is to separate data from code by using [hierarchical data structures](http://docs.puppetlabs.com/hiera/1/puppet.html) to store data, stripping your puppet code back to "one size fits all". This is also known as "doing it right" or "winning".

As previously stated, a goal of writing non-murder inducing puppet code is to have your manifest just work, regardless if it's applied to a server in environment A, or data center B, or running operating system C, or if service D should be stopped on server E instead of running like it is everywhere else. This can be achieved by moving all node specific data out of your manifests, into [YAML](http://en.wikipedia.org/wiki/YAML) formatted data sets called hiera. Note that hiera also supports [JSON](http://en.wikipedia.org/wiki/JSON) out of the box, or you can [write your own backend provider](http://docs.puppetlabs.com/hiera/1/custom_backends.html) if you're into that sort of thing.

A simple way to begin using hiera is to consolidate the [case](http://docs.puppetlabs.com/puppet/2.7/reference/lang_conditional.html#case-statements) and [if](http://docs.puppetlabs.com/puppet/2.7/reference/lang_conditional.html#if-statements) conditional clauses you have buried within your manifests to support multiple operating systems. By doing so you have gone from a bunch of repeating code to support [deb](http://en.wikipedia.org/wiki/Deb_(file_format)), [tbz](http://www.freebsd.org/doc/en/books/handbook/packages-using.html) and [rpm](http://en.wikipedia.org/wiki/RPM_Package_Manager) providers and endless if/else statements, to having a one stop shop for dynamaic variables and puppet code that supports it all out of the box.

I run the 3.\* branch open source version of puppet, so hiera was already installed. On the puppet master, create /etc/puppet/hiera.yaml
<pre>  ---
  :hierarchy:
      - "node/%{::fqdn}"
      - %{operatingsystem}
      - common
  :backends:
      - yaml
  :yaml:
      :datadir: '/etc/puppet/hieradata'</pre>
Create a data directory to store additional yaml files. Puppetlabs recommends you keep it within the puppet configuration directory because that's were humans will look first. I'd like to agree.

Create the following yaml files [based on facter](http://puppetlabs.com/facter) to support your mishmash of operating systems.

/etc/puppet/hieradata/CentOS.yaml
<pre>  ---
  bindutils: 'bind-utils'
  cron_name: 'cronie'
  cron_daem: 'crond'
  manpages:  'man'
  motdpath:  '/etc/motd'
  netcat:    'nc'
  ssh_pkgs:
             - 'openssh'
             - 'openssh-clients'
             - 'openssh-server'
  ssh_sname: 'sshd'</pre>
/etc/puppet/hieradata/Ubuntu.yaml
<pre>  ---
  bindutils: 'bind9utils'
  cron_name: 'cron'
  cron_daem: 'cron'
  manpages:  'manpages'
  motdpath:  '/etc/motd.tail'
  netcat:    'netcat-openbsd'
  ssh_pkgs:
             - 'openssh-client'
             - 'openssh-server'
  ssh_sname: 'ssh'</pre>
Also create /etc/puppet/hieradata/node/unique-flower.yourdomain.com.yaml to support the one stupid CentOS 5 box you have for unknown reasons. Support it now, wage war at later date.
<pre>  ---
  cron_name: 'vixie-cron'</pre>
Now start your manifest by telling puppet to use hiera to resolve needed variables instead of have a ton of silly repeating conditional statements.
<pre>  $bindutils = hiera('bindutils')
  $cron_name = hiera('cron_name')
  $cron_daem = hiera('cron_daem')
  $manpages  = hiera('manpages')
  $motdpath  = hiera('motdpath')
  $netcat    = hiera('netcat')
  $ssh_pkgs  = hiera('ssh_pkgs')
  $ssh_sname = hiera('ssh_sname')</pre>
There are a few [different ways that hiera can fill in variables](http://docs.puppetlabs.com/hiera/1/lookup_types.html) in your code. If you look at the order of our hiera.yaml we go from unique flowers, to general operating systems, to traits that are common across platform. Hiera uses facter facts to resolve them. When puppet is compiling a manifest using hiera, it will attempt to resolve a variable in hierarchical order. If no result is found it will move on to the next data source. Our example also uses a group of packages for openssh, allowing us a single instance of package definition within our manifest to install multiple packages. Here is what our manifest now looks like:
<pre>  package { 'openssh':
    ensure => installed,
    name   => $ssh_pkgs,
  }
</pre>
<pre>  package { 'crond':
    ensure => installed,
    name   => $cron_name,
  }
</pre>
<pre>  service { 'cron_daem':
    ensure  => running,
    name    => $cron_daem,
    enable  => true,
  }
</pre>
The puppet code has now been un-f\*cked to support [s/all/most](http://en.wikipedia.org/wiki/Sed) of your environments. Hiera has made our manifest more portable, easier to read, easier to maintain, and has expanded the data set available to other modules.

Go pat yourself on the back for reducing complexity and bloat. Try not to get stabbed on the way out the door.
