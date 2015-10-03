---
title: "Automating Nagios with Puppet and Puppetdb"
date: "2014-09-02T20:22:00-07:00"
slug: "automating-nagios-with-puppet-and-puppetdb"
---

Monitoring automation should be a core component of any mission critical infrastructure. You can't rely on servers and software. You can't rely on people to correctly add servers and software to your monitoring system either. The only thing you can count on is that everything will fail at some point. Computers will always do a better job than humans, so outsource all you can to a machine.

For better or for worse [Nagios](http://www.nagios.org/) has been affecting (controlling) my life for years. If you have spent any time looking at the [puppetlabs type reference](https://docs.puppetlabs.com/references/latest/type.html) it's hard to overlook that roughly a third of the types are for nagios objects. A clear indicator of how important this was to so many people to be migrated in as a core component of the product.

[Puppetdb](https://docs.puppetlabs.com/puppetdb/latest/#what-data) is a [clojure](http://clojure.org/) application that stores puppet agent facts, catalog, run reports,
and any other data exported and plugs it in a [postgresql](http://www.postgresql.org/) back end. The data is then available via sql query or [rest api](http://en.wikipedi
a.org/wiki/Representational_state_transfer) that responds in [JSON](http://json.org/). While this data can be helpful in endless ways, the relevant function here is that
it can store data exported by agents. Having an automated authoritative data store is the golden ticket to automating nagios monitoring.

Knowing this, like so may others, I first tried to find a module someone else had written that would work in my environment. I never found a solution that was flexible or put together in a logical fashion. Repeatedly I've started to write a puppet module with the intentions of public consumption, only to hit the wall of "how is this going to be flexible enough to work in someone else's puppet code base".

So far, the answer is "It won't be". I'm giving up on a solution and instead offering a rough overview of my implementation. 

My solution was to use a mix of exported data and manual configuration from a [hiera](https://docs.puppetlabs.com/hiera/1/) data set. My nagios module structure looks like this:
<pre>nagios
├── files
│   └── templates.cfg 
├── manifests
│   ├── checks.pp 
│   ├── commands.pp 
│   ├── contactgroups.pp
│   ├── contacts.pp
│   ├── export.pp
│   ├── hostgroups.pp
│   ├── hosts.pp
│   ├── import.pp
│   ├── init.pp
│   ├── install.pp
│   └── service.pp 
└── templates
    ├── cgi.cfg.erb
    └── nagios.cfg.erb</pre>


The main class is to only be included in the node definition of a nagios server. It looks like this:
<pre>class nagios ( ) {<br>
  include nagios::install
  include nagios::service
  include nagios::import<br>
  Class['nagios::install'] -> Class['nagios::service', 'nagios::import']<br>
  $nagios_checks        = hiera('nagios::checks')
  $nagios_commands      = hiera('nagios::commands')
  $nagios_contactgroups = hiera('nagios::contactgroups')
  $nagios_contacts      = hiera('nagios::contacts')
  $nagios_hostgroups    = hiera('nagios::hostgroups')
  $nagios_hosts         = hiera('nagios::hosts')<br>
  create_resources('nagios::checks', $nagios_checks)
  create_resources('nagios::commands', $nagios_commands)
  create_resources('nagios::contactgroups', $nagios_contactgroups)
  create_resources('nagios::contacts', $nagios_contacts)
  create_resources('nagios::hostgroups', $nagios_hostgroups)
  create_resources('nagios::hosts', $nagios_hosts)<br>
  resources { [ 'nagios_command', 'nagios_contact', 'nagios_contactgroup',
                'nagios_host', 'nagios_hostgroup', 'nagios_service' ]:
    purge  => true,
  }
}</pre>

Yes, I know about automatic lookups, but I often find variable declarations easier to read because I can see where the data is coming from and I know where to [grep](http://en.wikipedia.org/wiki/Grep) for answers. People like to store data in odd places.

I include nagios::export on all nodes in the fleet. It resembles this: 
<pre>class nagios::export {<br>
  @@nagios_host {"${::hostname}.${::network}.${::datacenter}":
    ensure     => present,
    address    => $::ipaddress,
    group      => nagios,
    hostgroups => "${::datacenter}, ${::kernel}, ${::network}, ${::virtual}",
    mode       => '0644',
    owner      => root,
    use        => 'generic-server',
  }
}</pre>

So now we have agents with the nagios::export class sending data to puppetdb identifying themselves as unique nagios objects and offering a few host groups they belong to via native and custom facts. The nagios::import class that is included in the nagios class (only applied to the nagios server) is used to poll this data from puppetdb and clobber out a new nagios configuration file. It does so by using the [exported resource collector operator](https://docs.puppetlabs.com/puppet/latest/reference/lang_collectors.html#exported-resource-collectors). It looks like this:
<pre>class nagios::import {<br>
  Nagios_command <<||>> {
    require => Class['nagios::install'],
    notify  => Class['nagios::service'],
  }<br>
  Nagios_host <<||>> {
    require => Class['nagios::install'],
    notify  => Class['nagios::service'],
  }<br>
  Nagios_hostgroup <<||>> {
    require => Class['nagios::install'],
    notify  => Class['nagios::service'],
  }<br>
  Nagios_service <<||>> {
    require => Class['nagios::install'],
    notify  => Class['nagios::service'],
  }<br>
}</pre>

Some nagios commands I left defined in hiera. I will most likely break them all off to be exported through modules. Leaving this as a manual process long term would result in 1000 nagios command checks defined and 100 in use. Again, outsource to the machines if you can.
<pre>---
nagios::commands
  'check_ping':
    command_line: /usr/local/nagios/libexec/check_ping -H $HOSTADDRESS$ -w 100.0,90% -c 200.0,60%
  'check_snowflake':
    command_line: /usr/local/nagios/libexec/check_nrpe -H $HOSTADDRESS$ -c check_snowflake</pre>
 
Our nagios::commands resource (only applied to the nagios server) polls the data providers (hiera and puppetdb) and clobbers in the required definition to the nagios configuration file. It looks like this:
<pre>define nagios::commands (
  $ensure        = undef,
  $command_line  = undef,
) {<br>
  nagios_command { $name:
    ensure       => $ensure,
    command_line => $command_line,
    group        => nagios,
    mode         => '0644',
    notify       => Class['nagios::service'],
    owner        => root,
  }
}</pre>

Things I cannot easily automate (yet) within my current environment such as contacts and contactgroups are also stored in a hiera data set. Once there is a reliable source (I'm lookin at you, ldap) that can tell me who is and who isn't a systems administrator for what systems I'll be able to write some garbage to export the data for me. Until then I'm using the same formula above. While the above is a poor example, as you build out classes for contacts, hostgroups, hosts, services, etc, I suggest you include all available parameters, set them as undefined, and include logic to test that the required values for nagios to function are filled in correctly. Expect everything, demand the bare minimum.
 
With this framework it's now trivial to automate unique nagios checks for puppet controlled services. Either build one stupid large module with a bunch of goofy "if class is applied then" type stanzas, or loop through and update all of your puppet modules to export the checks required to monitor whatever function the module provides. If your puppet module pukes out unicorns and you want to ensure the unicorns are running then include a snippet like this within the unicorn generation module: 
<pre>@@nagios_service { "check_unicorns-${::hostname}":
  ensure              => present,
  use                 => 'generic-service', 
  host_name           => $::hostname,
  service_description => 'Unicorn Status',
  check_command       => 'check_unicorn-${::hostname}",
}<br>
@@nagios_command { "check_unicorns-${::hostname}":
  command_line => "check_nrpe -H $HOSTADDRESS$ -c check_unicorns",
}
</pre>

Now we have the puppet agent on the nagios server auto-magically creating configuration files for us that are updated with each run. We have unique service and check names so puppet can manage them and we can tweak commands to act differently per host. Wonderful. Now how do we remove hosts from nagios?

When a host is decommissioned have the following issued on the puppetmaster to remove it's ssl certificate and purge (most of) it's data from puppetdb. You could also have this done automatically by setting node-ttl and node-purge-ttl in puppetdb's configuration. 
<pre>puppet cert --clean $hostname ; puppet node deactivate $hostname</pre> 

If you review our main nagios class again, you will see the [resources metatype](https://docs.puppetlabs.com/references/latest/type.html#resources) with an array of resources and purge set to true. This automatically purges resources that are not explicitly defined by puppet. Since we just purged a node from puppetdb, the host definition and its checks (remember all checks are host specific, even a simple ping) will be removed from the nagios configuration files the next time the puppet agent runs on our nagios server.  

There are a few caveats to this set up. The first is resource purging is half broken. Search the internet, you will find a lot of mentions of it for various things. Flip through puppetlabs jira instance and you will find a lot of related tickets. When I first built this out I had included target => '/etc/nagios/conf.d/${filename}' for all the managed resources so I didn't have to define the configuration files from within nagios.cfg. This resulted in resources failing to be purged. By default puppet is going to want to write files to /etc/nagios/nagios_hostname.cfg for example. Let it. Defining a file target and the [resources metatype](https://docs.puppetlabs.com/references/latest/type.html#resources) don't currently play well with one another. 

The other issue is required for my environment and most likely everyone else's. The module is polling for data from both puppetdb and hiera, providing the ability to manually include and remove host and service data. On one hand, flexibility is great. I have a way to include devices which can't run the puppet agent. On the other hand we have created another data set that will require endless editing and auditing, going against the grain of what we want to achieve. What is this again? Why is it here? Do we still need it? Why is it not automated? If you have servers that are not built and controlled by your change management software, you are doing it wrong. If you have unique snowflake servers with anything manually tweaked it's going to be forgotten about forever and nobody will know why it worked in production but fails everywhere else. You are doing it wrong. 

Now go rethink and rebuild your monitoring setup for the 10th time.
