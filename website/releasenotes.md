---
layout: default
title: Release Notes
---
# {{page.title}}

This is a list of release notes for various releases, you should review these before upgrading as any potential problems and backward incompatible changes will be highlighted here.

 * TOC Placeholder
  {:toc}

## 1.1.1 - 2011/02/02

This is a release in the development series of mcollective.  It features major new
features and numerous bug fixes.  Please pay careful attention to the upgrading
section as there is some changes that are not backward compatible.

This release is for early adopters, production users should consider the 1.0.x series.

### AES+RSA Security Plugin

A new security plugin that encrypts the payloads, uniquely identify senders and secure
replies from inspection by other people on the collective has been written.  The plugin
can re-use Puppet certificates and supports distributing of public keys if you wish.

This plugin and its deployment is very complex and it has a visible performance impact
but we felt it was a often requested feature and so decided to implement it.

Full documentation for this plugin can be found [in our docs](reference/plugins/security_aes.html), please read them very
carefully should you choose to deploy this plugin.

### Single Exeuctable Framework

In the past a lot of the CLI tools have behaved inconsistently as the mc scripts were
mostly just written to serve immediate needs, we are starting a process of improving
these scripts and making them more robust.

The first step is to create a new framework for CLI commands, we call these Single Executable
Applications.  A new executable called _mc_ is being distributed with this release:

{% highlight console %}
$ mc
The Marionette Collective version 1.1.1

/usr/sbin/mc: command (options)

Known commands: rpc filemgr inventory facts ping find help
{% endhighlight %}

{% highlight console %}
$ mc help
The Marionette Collection version 1.1.1

  facts           Reports on usage for a specific fact
  filemgr         Generic File Manager Client
  find            Find hosts matching criteria
  help            Application list and RPC agent help
  inventory       Shows an inventory for a given node
  ping            Ping all nodes
  rpc             Generic RPC agent client application
{% endhighlight %}

{% highlight console %}
$ mc rpc package status package=zsh
Determining the amount of hosts matching filter for 2 seconds .... 51

 * [ ============================================================> ] 51 / 51


 test.com:
    Properties:
       {:provider=>:yum,
	:release=>"3.el5",
	:arch=>"x86_64",
	:version=>"4.2.6",
	:epoch=>"0",
	:name=>"zsh",
	:ensure=>"4.2.6-3.el5"}
{% endhighlight %}

You can see these commands behave just like their older counter parts but is more integrated
and discovering available commands is much easier.

Agent help that was in the past available through _mc-rpc --ah agentname_ is now available through
_mc help agentname_ and error reporting is short single line reports by default but by adding
_-v_ to the command line you can get full Ruby backtraces.

We've maintained backward compatibility by creating wrappers for all the old mc scripts but these
might be deprecated in future.

These application live in the normal plugin directories and should make it much easier to distrubute
plugins in future.

We will port the scripts for plugins to this framework and encourage you to do the same when writing
new CLI tools.  An example of a ported CLI can be seen in the _filemgr_ agent.

Find the documentation for these plugins [here](reference/plugins/application.html).

### Miscelaneous Improvements

The logging system has been refactored to not use a Signleton, logging messages are now simply:

{% highlight ruby %}
MCollective::Log.notice("hello world")
{% endhighlight %}

A backwards compatible wrapper exist to prevent existing code from breaking.

In some cases - like when using MCollective from within Rails - the STOMP
gem would fail to decode the payloads.  We've worked with the authors and
a new release was made that makes this more robust but we've also enabled
Base64 encoding on the Stomp connector for those who can't upgrade the Gem
and who are running into this problem.

### Bug Fixes


 * Machines that do not pass security checks are handled as having not responded
   so that these are listed in the usual stat for non responsive hosts
 * The - character is now allowed in Fact names by the DDL for rpcutil
 * Version 1.1.0 introduced a bug with reloading agents from disks using USR1 and mc-controller

### Enhancements

 * New AES+RSA based security plugin was added
 * Create a new single executable framework and port several mc scripts
 * Security plugins have access to the callerid they are responding to
 * The logging methods have been improved by removing the use of Singletons
 * The STOMP connector can now Base64 encode all sent data to deal with en/decoding issues by the gem
 * The rpcutil agent has a new _ping_ action
 * the _mc ping_ client now supports standard filters
 * DDL documentation has been updated to show you can disable type validations in the DDL
 * Fact plugins can now force fact cache invalidation, the YAML plugin will immediately load new facts when mtime on the file change
 * Improve _1.0.0_ compatibility for _foo=/bar/_ style fact matches at the expense of _1.1.0_ compatibility

### Upgrading

Upgrading should be mostly painless as most things are backward compatible.

We discovered that we broke backward compatibility with _1.0.0_ and _0.4.x_ Fact filters.  A filter in the form
_foo=/bar/_ would be treated as an equality filter and not a regular expression.

This releases fixes this compatibility with older versions at the expense of compatibility with _1.1.0_.  If you
are upgrading from _1.1.0_ keep this in mind and plan accordingly, once you've upgraded a client its requests that
contain these filters will not be correctly parsed on servers running _1.1.0_.

The security plugins have changed slightly, if you wrote your own security plugin the interface to _encodereply_
has changed slightly.  All the bundled security plugins have been updated already and older ones will just
keep working.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2011/02/02|Load the DDL from disk once per printrpc call and not for every result|5958|
|2011/02/02|Include full Apache 2 license text|6113|
|2011/01/31|Create a new single executable application framework|5897|
|2011/01/30|Fix backward compatibility with old foo=/bar/ style fact searches|5985|
|2011/01/30|Documentation update to reflect correct default identity behavior|6073|
|2011/01/29|Let the YAML file force fact reloads when the files update|6057|
|2011/01/29|Add the ability for fact plugins to force fact invalidation|6057|
|2011/01/29|Document an approach to disable type validation in the DDL|6066|
|2011/01/19|Add basic filters to the mc-ping command|5933|
|2011/01/19|Add a ping action to the rpcutil agent|5937|
|2011/01/17|Allow MC::RPC#printrpc to print single results|5918|
|2011/01/16|Provide SimpleRPC style results when accessing the MC::Client results directly|5912|
|2011/01/11|Add an option to Base64 encode the STOMP payload|5815|
|2011/01/11|Fix a bug with forcing all facts to be strings|5832|
|2011/01/08|When using reload_agents or USR1 signal no agents would be reloaded|5808|
|2011/01/04|Use the LSB based init script on SUSE|5762|
|2011/01/04|Remove the use of a Singleton in the logging class|5749|
|2011/01/02|Add AES+RSA security plugin|5696|
|2010/12/31|Security plugins now have access to the callerid of the message they are replying to|5745|
|2010/12/30|Allow - in fact names|5727|
|2010/12/29|Treat machines that fail security validation like ones that did not respond|5700|

## 1.1.0 - 2010/12/29

This is the first in a new development series, as such there will be rapid changes
and new features.  We cannot guarantee the changes will be backward compatible but
we will as before try to keep these releases solid and production quality.

Production users who do not wish to have rapid change should use release 1.0.0.

This release focus mainly on getting all the community contributed code into a release
and adressing some issues I had but wasn't comfortable fixing them late in the
previous development series.

Please read these notes carefully we are **removing** some old functionality and changing
some internals, you need to carefully review the text below.

### Bug Fixes

 * The progress bar will now try hard to detect screen size and adjust itself,
   failing back to a dumb mode if it can't work it out.
 * rpcutil timeout was too short when considering slow facts and network latency

### Improvements

 * libdir can now be multiple directories specified with : seperation - Thanks to Richard Clamp
 * Logging is now pluggable, 3 logger types are supported - file, syslog and console.  Thanks to
   Nicolas Szalay for the initial Syslog code
 * A new experimental ssh agent based security system.  Thanks to Jordan Sissel
 * New fact matching operators <=, >=, <, >, !=, == and =~. Thanks to Mike Pountney
 * SimpleRPC fact_filter method can now take any valid fact string as input in addition to the old format
 * A message gets logged at startup showing mcollective version and logging level
 * The fact plugin format has been changed, simplified, made thread safe and global caching added.
   This breaks backward compatability with old fact sources
 * Creating options hashes has been simplified by adding a helper that creates them for you
 * Calculating the client timeout has been improved by including for latency and fact source slowness
 * Audit log lines are now on one line and include a ISO 8601 format date

### Removed Functionality

 * The old MCOLLECTIVE_TIMEOUT and MCOLLECTIVE_DTIMEOUT were removed, a new MCOLLECTIVE_EXTRA_OPTS
   was added which should allow much more flexibility.  Supply any command line options in this var

### Upgrading

Upgrading should be easy the only backward incompatible change is the Facts format.  If you only use
the included YAML plugin the upgrade will just work if you use the packages.  If you use either the
facter or ohai plugins you will need to download new plugins from the community plugin page.

If you wrote your own Facts plugin you will need to change it a bit:

  * The old get_facts method should now be load_facts_from_source
  * The class for facts have to be in the form MCollective::Facts::Foo_facts and the filename should match

This is all, your facts can now be much simpler as threading and caching is handled in the base class.

You can place these new plugins into the plugindir before upgrading.  The old mcollective will not use
these plugins and the new one will not touch the old ones.  This will allow for a clean rollback.

Once the new version is deployed you will immediately have caching on all fact types at 3000 seconds
you can tune this using the fact_cache_time setting in the configuration file.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2010/12/28|Adjust the logfile audit format to include local time and all on one line|5694|
|2010/12/26|Improve the SimpleRPC fact_filter helper to support new fact operators|5678|
|2010/12/25|Increase the rpcutil timeout to allow for slow facts|5679|
|2010/12/25|Allow for network and fact source latency when calculating client timeout|5676|
|2010/12/25|Remove MCOLLECTIVE_TIMEOUT and MCOLLECTIVE_DTIMEOUT environment vars in favour of MCOLLECTIVE_EXTRA_OPTS|5675|
|2010/12/25|Refactor the creation of the options hash so other tools don't need to know the internal formats|5672|
|2010/12/21|The fact plugin format has been changed and simplified, the base now provides caching and thread safety|5083|
|2010/12/20|Add parameters <=, >=, <, >, !=, == and =~ to fact selection|5084|
|2010/12/14|Add experimental sshkey security plugin|5085|
|2010/12/13|Log a startup message showing version and log level|5538|
|2010/12/13|Add a console logger|5537|
|2010/12/13|Logging is now plugable and a syslog plugin was provided|5082|
|2010/12/13|Allow libdir to be an array of directories for agents and ddl files|5253|
|2010/12/13|The progress bar will now intelligently figure out the terminal dimentions|5524|

## 1.0.0 - 2010/12/13

### Release Focus and Notes

This is a bug fix and minor improvement release.

We will maintain the 1.0.x branch as a stable supported branch.  The features
currently in the branch will be frozen and we'll only do bug fixes.

A new 1.1.x series of releases will be done where we will introduce new features.
Once the 1.1.x code base reaches a mature point it will become the new stable
release and so forth.

### Bug Fixes

 * Settings like retry times were ignored in the Stomp connector
 * The default init script had incorrect LSB comments
 * The rpcutil DDL has better validation and will now match all facts

### New Features and Enhancements

 * You can now send RPC requests to a subset of discovered nodes
 * SimpleRPC custom_request can now be used to create fire and forget requests
 * Clients can now cleanly disconnect from the middleware.  Bundled clients have been
   updated.  This should cause fewer exceptions in ActiveMQ logs
 * Rather than big exceptions many clients will now log errors only
 * mc-facts has been reworked to be a SimpleRPC client, this speeds it up significantly
 * Add get_config_item to rpcutil to retrieve a running config value from a server
 * YAML facts are now forced to be all strings and is thread safe
 * On RedHat based systems the requirement for the LSB packages has been removed

The first feature is a major new feature in SimpleRPC.  If you expose a service redundantly
on your network using MCollective you wouldn't always want to send requests to all the
nodes providing the service.  You can now limit the requests to an arbitrary amount
using the new --limit-nodes option which will also take a percentage.  A shortcut -1 has
been added that is the equivelant to --limit-nodes 1

### Backwards Compatibility

This release will be backward compatible with older releases.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2010/12/04|Remove the LSB requirements for RedHat systems|5451|
|2010/11/23|Make the YAML fact source thread safe and force all facts to strings|5377|
|2010/11/23|Add get_config_item to rpcutil to retrieve a running config value from a server|5376|
|2010/11/20|Convert mc-facts into a SimpleRPC client|5371|
|2010/11/18|Added GPG signing to Rake packaging tasks (SIGNED=1)|5355|
|2010/11/17|Improve error messages from clients in the case of failure|5329|
|2010/11/17|Add helpers to disconnect from the middleware and update all bundled clients|5328|
|2010/11/16|Correct LSB provides and requires in default init script|5222|
|2010/11/16|Input validation on rpcutil has been improved to match all valid facts|5320|
|2010/11/16|Add the ability to limit the results to a subset of hosts|5306|
|2010/11/15|Add fire and forget mode to SimpleRPC custom_request|5305|
|2010/11/09|General connection settings to the Stomp connector was ignored|5245|

## 0.4.10 - 2010/10/18

### Release Focus and Notes

This is a bug fix and minor improvement release.

### Bug Fixes

 * Multiple RPC proxy classes in the same script would not all share the same command line options
 * Ruby 1.9.x compatibility has been improved
 * A major bug in registration has been fixed, any exception in the registration logic would have
   resulted in a high CPU consuming loop

The last bug is a major issue it will result in the _mcollectived_ consuming lots of CPU, updating to
this version of MCollective is strongly suggested.  Should you run into this problem on a large scale
you can use _mc-controller exit_ to exit all your _mcollectived_ processes at the same time.

### New Features and Enhancements

 * The PSK security plugin can now be configured to set the callerid to a few different values
   useful for cases where you want to do group based RPC Authorization for example.
 * Info logging has been minimised by demoting the 'not targetted at us' message to debug
 * Document the 'exit' option to mc-controller

### Backwards Compatibility

This release will be backward compatible with older releases.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2010/10/18|Document exit command to mc-controller|152|
|2010/10/13|Log messages that don't pass the filters at debug level|149|
|2010/10/03|Preserve options in cases where RPC::Client instances exist in the same program|148|
|2010/09/30|Add the ability to set different types of callerid in the PSK plugin|145|
|2010/09/30|Improve Ruby 1.9.x compatibility|142|
|2010/09/29|Improve error handling in registration to avoid high CPU usage loops|143|


## 0.4.9 - 2010/09/21

### Release Focus and Notes

This is a bug fix and minor improvement release.

### Bug Fixes

 * Internal data structure related to Agent meta data has been fixed, no user impact from this
 * When using per-user config files the _rpc-help.erb_ template could not be found
 * The log files will now rotate by default keeping 5 x 2MB files
 * The config were parsed multiple times in complex scripts, this has been eliminated
 * MCollective::RPC loaded a bunch of unneeded stuff into Object, this has been cleaned up
 * Various packaging related tweaks were done

### New Features

 * We ship a new agent called _rpcutil_ with the base system, you can use this agent to get inventory etc from your _mcollectived_.  _mc-inventory_ has been rewritten to use this agent and should serve as a good reference for what you can get from the agent.
 * The DDL now support :boolean style inputs, mc-rpc also turn true/false on the command line into booleans when needed

### Backwards Compatibility

This release will be backward compatible with older releases.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2010/09/20|Improve Debian packaging task|140|
|2010/09/20|Add :boolean type support to the DDL|138|
|2010/09/19|Refactor MCollective::RPC to add less unneeded stuff to Object|137|
|2010/09/18|Prevent duplicate config loading with multiple clients active|136|
|2010/09/18|Rotate the log file by default, keeping 5 x 2MB files|135|
|2010/09/18|Write a overview document detailing security of the collective|131|
|2010/09/18|Add MCollective.version, set it during packaging and include it in the rpcutil agent|134|
|2010/09/13|mc-inventory now use SimpleRPC and the rpcutil agent and display server stats|133|
|2010/09/13|Make the path to the rpc-help.erb configurable and set sane default|130|
|2010/09/13|Make the configfile used available in the Config class and add to rpcutil|132|
|2010/09/12|Rework internal statistics and add a rpcutil agent|129|
|2010/09/12|Fix internal memory structures related to agent meta data|128|
|2010/08/24|Update the OpenBSD port for changes in 0.4.8 tarball|125|
|2010/08/23|Fix indention/block error in M:R:Stats|124|
|2010/08/23|Fix permissions in the RPM for files in /etc|123|
|2010/08/23|Fix language in two error messages|122|

## 0.4.8 - 2010/08/20

### Release Focus and Notes

This is a bug fix and minor improvement release.

### Bug Fixes

 * The RPM packages now require redhat-lsb since our RC scripts need it
 * The rake tasks do not attempt to build rpms on all platforms
 * Some plugin missing related exceptions are now handled gracefully
 * The Rakefile had a few warnings cleaned up

### Notable New Features

 * Users can now have a _~/.mcollective_ file which will be preferred over over _/etc/mcollective/client.cfg_ if it exists.  You can still use _--config_ to override.

 * The SSL Security plugin can now use "either YAML or Marshal for serialization":/reference/plugins/security_ssl.html#serialization_method, this means other programming languages can be used as clients.  A sample Perl client is included in the ext directory.  Marshal remains the default for backwards compatibilty

 * _mc-inventory_ can now be used to create "custom reports using a small reporting DSL":/reference/ui/nodereports.html, this enable you to build custom reports listing all your machines and gives you access to facts, agents and classes lists.

 * The log level for the _mcollectived_ can be adjusted during run time using the _USR2_ unix process signal.

### Backwards Compatibility

This release will be backward compatible with older releases.  If you choose to use YAML in the SSL plugin you need matching versions on the client.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2010/08/19|Fix missing help template in debian packages|90|
|2010/08/18|Clean up some hardlink warnings in the Rakefile|117|
|2010/08/18|Include the website in the main repo and add a simple Rake task|118|
|2010/08/17|Handle exceptions for missing plugins better|115|
|2010/08/17|Add support for ~/.mcollective as a config file|114|
|2010/08/07|SSL security plugin can use either YAML or Marshal|94|
|2010/08/06|Multiple YAML files can now be used as fact source|112|
|2010/08/06|Allow log level to be adjusted at run time with USR2|113|
|2010/07/31|Add basic report scripting support to mc-inventory|111|
|2010/07/06|Removed 'rpm' from the default rake task|109|
|2010/07/06|Add redhat-lsb to the server RPM dependencies|108|

## 0.4.7 - 2010/06/29

### Release Focus and Notes

This is a bug fix and incremental improvement release focusing on small improvements in the DDL mostly.

### Data Definition Language

We've extended the use of the DDL in the RPC client.  We've integrated the DDL into _printrpc_ helper.  The output is dynamic showing field names in human readable format rather than hash dumps.

We're also using color to improve the display of the results, the color display can be disabled with the new _color_ configuration option.

A "screencast of the DDL integration":http://mcollective.blip.tv/file/3799653 and color usage has been recorded.

### Bug Fixes

A serious issue has been fixed wrt complex agents.  If you attempted to use multiple agents from the same script errors such as duplicate discovery results or simply not working.

The default fact source has been changed to Yaml, it was inadvertently set to Facter int he past.

Some previously unhandled exceptions are now being handled correctly and passed onto the clients as failed requests rather than no responses at all.

### Backwards Compatibility

This release will be backward compatible with older releases.  The change to Yaml fact source by default might impact you if you did not previously specify a fact source in the configuration files.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/06/27 | Change default factsource to Yaml|106|
| 2010/06/27 | Added VIM snippets to create DDLs and Agents|102|
| 2010/06/26 | DDL based help now works better with Symbols in in/output|105|
| 2010/06/23 | Whitespace at the end of config lines are now stripped|100|
| 2010/06/22 | printrpc will now inject some colors into results|99|
| 2010/06/22 | Recover from syntax and other errors in agents|98|
| 2010/06/17 | The agent a MC::RPC::Client is working on is now available|97|
| 2010/06/17 | Integrate the DDL with data display helpers like printrpc|92|
| 2010/06/15 | Avoid duplicate topic subscribes in complex clients|95|
| 2010/06/15 | Catch some unhandled exceptions in RPC Agents|96|
| 2010/06/15 | Fix missing help template file from RPM|90|

## 0.4.6 - 2010/06/14

### Release Focus and Notes

This release is a major feature release.

We're focusing mainly on the Stomp connector and on the SimpleRPC agents in this release though a few smaller additions were made.

### Stomp Connector

We've historically been stuck just using Rubygem Stomp 1.1 due to multi threading bugs in the newer releases.  All attempts to contact the authors failed.  Recently though I had some luck and these issues are resolved in the RubyGem Stomp 1.1.6 release.

This means we can take advantage of a lot of new features such as connection pooling for failover/ha and also SSL TLS between nodes and ActiveMQ server.

See "Stomp Connector":/reference/plugins/connector_stomp.html for details.

### RPC Agent Data Description Language

I've been working since around February on building introspection, automatically generated documentation and the ability for user interfaces to be auto generated for agents, even ones you write your self.

This feature is documented in "DDL":/simplerpc/ddl.html but a quick example of a DDL document might help make it clear:

### CLI Utilities changes

  * _mc-facts_ now take all the standard filters so you can make reports for just subsets of machines
  * A new utility _mc-inventory_ has been added, it will show agents, facts and classes for a node
  * _mc-rpc_ has a new option _--agent-help_ that will use the DDL and display auto generated documentation for an agent.
  * _mc-facts_ output is sorted for better readability

### Backwards Compatibility

This release will be backward compatible with older releases, the new Stomp features though require the newer Ruby Gem.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/06/12 | Qualify the Process class to avoid clashes in the discovery agent|88|
| 2010/06/12 | Add mc-inventory which shows agents, classes and facts for a node|87|
| 2010/06/11 | mc-facts now supports standard filters|86|
| 2010/06/11 | Add connection pool retry options and ssl for connection|85|
| 2010/06/11 | Add support for specifying multiple stomp hosts for failover|84|
| 2010/06/10 | Tighten up handling of filters to avoid nil's getting into them|83|
| 2010/06/09 | Sort the mc-facts output to be more readable|82|
| 2010/06/08 | Fix deprecation warnings in newer Stomp gems|81|

## 0.4.5 - 2010/06/03

### Release Focus and Notes

This release is a major feature release.

The focus of this release is to finish up some of the more enterprise like features, we now have fine grained Authorization and Authentication and a new security model that uses SSL keys.

### Security Plugin

Vladimir Vuksan contributed the base code of a new "SSL based security plugin":/reference/plugins/security_ssl.html .  This plugin builds on the old PSK plugin but gives each client a unique certificate pair.  The nodes all share a certificate and only store client public keys.  This means that should one node be compromised it cannot be used to control the rest of the network.

### Authorization Plugin

We've developed new hooks and plugins for SimpleRPC that enable you to write "fine grained authorization systems":/simplerpc/authorization.html .  You can do authorization on every aspect of the request and you'll have access to the caller userid as provided by the security plugin.  Combined with the above SSL plugin this can be used to build powerful and secure Authentication and Authorization systems.

A sample plugin can be found "here":http://code.google.com/p/mcollective-plugins/wiki/ActionPolicy

### Enhancements for Web Development

Web apps doesn't tie in well with our discover, request, wait model.  We've made it easier for web apps to maintain their own cached discovery data using the "Registration:/reference/plugins/registration.html system and then based on that do requests that would not require any discovery.

### Fire and Forget requests

It is often desirable to just send a request and not wait for any reply.  We've made it easy to do requests like this] with the addition of a new request parameter on the SimpleRPC client class.

Requests like this will not take any time to do discovery and you will not be able to get results back from the agents.

### Reloading Agents

To make it a bit easier to manage daemons and agents you can now send the _mcollectived_ a _USR1_ signal and it will re-read all it's agents from disk.

### Backwards Compatibility

This release when used with the old style PSK plugin should be perfectly backward compatible with your existing agents.  To use some of the newer features like authorization will require config and/or agent changes.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/06/01 | Improve the main discovery agent by adding facts and classes to its inventory action|79|
| 2010/05/30 | Add various helpers to get reports as text instead of printing them|43|
| 2010/05/30 | Add a custom_request method to call SimpleRPC agents with your own discovery|75|
| 2010/05/30 | Refactor RPC::Client to be more generic and easier to maintain|75|
| 2010/05/29 | Fix a small scoping issue in Security::Base|76|
| 2010/05/25 | Add option --no-progress to disable progress bar for SimpleRPC|74|
| 2010/05/23 | Add some missing dependencies to the RPMs|72|
| 2010/05/22 | Add an option _:process_results_ to the client|71|
| 2010/05/13 | Fix help output that still shows old branding|70|
| 2010/04/27 | The supplied generic stompclient now accepts STOMP_PORT in the environment|68|
| 2010/04/26 | Add a SimpleRPC Client helper to reset filters|64|
| 2010/04/26 | Listen for signal USR1 and reload all agents from disk|65|
| 2010/04/12 | Add SimpleRPC Authorization support|63|


## 0.4.4 - 2010/04/03

### Release Focus and Notes

This release is mostly a bug fix release.

We've cleaned up the logs a bit so you'll see fewer exceptions logged, also if you have Rails + Puppet on a node the logs will stay in the standard format.  We are handling some exceptions better which has improved stability of the registration system.

If you do some slow queries in your discovery agent you can bump the timeout up in the config using _plugin.discovery.timeout_.

All the scripts now use _/usr/bin/env ruby_ rather than hardcoded paths to deal better with Ruby's in weird places.

Several other small annoyances was fixed or improved.

### mc-controller

We've always had a tool that let you control a network of mcollective instances remotely, it lagged behind a bit with the core, we've fixed it up and documented it "here":/reference/basic/daemon.html .  You can use it to reload agents from disk without restarting the daemon for example or get stats or shut down the entire mcollective network.

### Backwards Compatibility

No changes that impacts backward compatibility has been made.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/03/27 | Make it easier to construct SimpleRPC requests to use with the standard client library|60|
| 2010/03/27 | Manipulating the filters via the helper methods will force rediscovery|59|
| 2010/03/23 | Prevent Activesupport when brought in by Facter from breaking our logs|57|
| 2010/03/23 | Clean up logging for messages not targeted at us|56|
| 2010/03/19 | Add exception handling to the registration base class|55|
| 2010/03/03 | Use /usr/bin/env ruby instead of hardcoded paths|54|
| 2010/02/17 | Improve mc-controller and document it|46|
| 2010/02/08 | Remove some close coupling with Stomp to easy creating of other connectors|49|
| 2010/02/01 | Made the discovery agent timeout configurable using plugin.discovery.timeout|48|
| 2010/01/25 | mc-controller now correctly loads/reloads agents.|45|
| 2010/01/25 | Building packages has been improved to ensure rdocs are always included|44|


## 0.4.3 - 2010/01/24

### Release Focus and Notes

This release fixes a few bugs and introduce a major new SimpleRPC feature for auditing requests.

### Auditing

We've created an "auditing framework for SimpleRPC":/simplerpc/auditing.html, each request gets passed to an audit plugin for processing.  We ship one that simply logs to a file on each node and there's a "community plugin":http://code.google.com/p/mcollective-plugins/wiki/AuditCentralRPCLog that logs everything on a central logging host.

In future we might add auditing to the client libraries so requests will be logged where they are sent as well as auditing of replies being sent, this will be driven by requests from the community though.

### New _fail!_ method for SimpleRPC

Till now while writing agents you can use the _fail_ method to set statusses in the reply, this however did not also raise exceptions and terminate execution of the agent immediately.

Often the existing behavior is required but it did lead to some awkward code when you did want to just exit the agent immediately as well as set a fail status.  We've added a _fail!_ method that works just like _fail_ except it stops execution of your agent immediately.

### Backwards Compatibility

No changes that impacts backward compatibility has been made.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/01/23 | Handle ctrl-c during discovery without showing exceptions to users|34|
| 2010/01/21 | Force all facts in the YAML fact source to be strings|41|
| 2010/01/19 | Add SimpleRPCAuditing audit logging to SimpleRPC clients and Agents| |
| 2010/01/18 | The SRPM we provide will now build outside of the Rake environment|40|
| 2010/01/18 | Add a _fail!_ method to RPC::Agent|37|
| 2010/01/18 | mc-rpc can now be used without supplying arguments|38|
| 2010/01/18 | Don't raise an error if no user/pass is given to the stomp connector, try unauthenticated mode|35|
| 2010/01/17 | Improve error message when Regex validation failed on SimpleRPC input|36|


## 0.4.2 - 2010/01/14

### Release Focus and Notes

This release fixes a few bugs, add some command line improvements and brings major changes to the Debian packaging.

### Packaging

Firstly we've had some amazing work done by Riccardo Setti to make us Debian packages that complies with Debian and Ubuntu policy, this release use these new packages.  It has some unfortunate changes to file layout detailed below but overall I think it's a big win to get us in line with Distribution policies and standards.

The only major change is that in the past we used _/usr/libexec/mcollective_ as the libdir, but Debian does not have this directory and it is not in the LFHS anymore so we now use _/usr/share/mcollective/plugins_ as the lib dir.  You need to move your plugins there and update both client and server configs.

The RedHat packages will move to this convention too in the next release since I think it's the better location and complies with LFHS.

### Command Line Improvements

We've streamlined the command line a bit, nothings changed we've just added some flags.

The _--with-class_, _--with-fact_, _--with-agent_ and _--with-identity_ now all have a short form _-C_, _-F_, _-A_ and _-I_ respectively.

We've added a new filter option _--with_ and a short form _-W_ that combines _--with-class_ and _--with-fact_ into one filter type, use case would be:

{% highlight console %}
  % mc-find-hosts -W "/centreon/ country=de roles::dev_server/"
{% endhighlight %}

This would find hosts with class regex matched _/centreon/_, class _roles::dev_server_ and fact matching _country=de_.  Hopefully this saves on some typing.

You can also now set the environment variables _MCOLLECTIVE_TIMEOUT_ and _MCOLLECTIVE_DTIMEOUT_ which saves you from typing _--timeout_ and _--discovery-timeout_ often, especially useful on very fast networks.

### Other fixes and improvements

 * We've added the COPYING file to all the packages
 * We've made the init script more LSB compliant
 * A bug related to discovery in SimpleRPC was fixed

### Backwards Compatibility

The only backwards issue is the Debian packages.  They've been tested to upgrade cleanly but you need to change the config as above.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/01/13 | New packaging for Debian provided by Riccardo Setti|29|
| 2010/01/07 | Improved LSB compliance of the init script - thanks Riccardo Setti|32|
| 2010/01/07 | Multiple calls to SimpleRPC client would reset discovered hosts|31|
| 2010/01/04 | Timeouts can now be changed with MCOLLECTIVE_TIMEOUT and MCOLLECTIVE_DTIMEOUT environment vars|25|
| 2010/01/04 | Specify class and fact filters easier with the new -W or --with option|27 |
| 2010/01/04 | Added COPYING file to RPMs and tarball|28|
| 2010/01/04 | Make shorter filter options -C, -I, -A and -F|26|

## 0.4.1 - 2010/01/02

### Release Focus and Notes

This is a bug fix release to address some shortcomings and issues found in Simple RPC.

The main issue is around handling of meta data in agents, the documented approach did not work, we've now solved this by adding a number of hooks into the processing of Simple RPC agents.

We've also made logging and config retrieval a bit easier in agents - and documented this.

You can now call the _mc-rpc_ command a bit easier:

{% highlight console %}
  % mc-rpc --agent helloworld --action echo --argument msg="hello world"
  % mc-rpc helloworld echo msg="hello world"
{% endhighlight %}

The 2 calls are the same, you can pass as many arguments in _key=val_ pairs as needed at the end.

### Backwards Compatibility

No issues with backward compatibility, should be a simple upgrade.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2010/01/02 | Added hooks to plug into the processing of requests, also enabled setting meta data and timeouts|14|
| 2010/01/02 | Created readers for @config and @logger in the SimpleRPC agent|23|
| 2009/12/30 | Don't send out any requests if no nodes were discovered|17|
| 2009/12/30 | Added :discovered and :discovered_nodes to client stats|20|
| 2009/12/30 | Add a empty_filter? helper to the RPC mixin|18|
| 2009/12/30 | Fix formatting bug with progress bar|21|
| 2009/12/29 | Simplify mc-rpc command line|16|
| 2009/12/29 | Fix layout issue when printing hosts that did not respond|15|


## 0.4.0 - 2009/12/29

### Release Focus and Notes

This release introduced a major new feature - Simple RPC - a framework for easily building clients and agents.  More than that it's a set of conventions and standards that will help us build generic clients like web based ones capable of talking to all agents.

We think this feature is ready for wide use, it's well documented and we've done extensive testing.  We'll be porting some of our own code over to it once this release is out and we do anticipate there might be some _0.4.x_ releases to round off a few issues that might remain.  We do not currently have any open tickets against Simple RPC.

We've also added the ability to create more complex queries such as:

{% highlight console %}
--with-class /dev_server/ --with-class /rails/
{% endhighlight %}

This does an _AND_ operation on the puppet classes on the node and finds only nodes with both _/dev_server/_ *AND* _/rails/_ classes.  This new functionality applies to all types of filter.

We've made the _--with-class_ filters more generic in comments, documentation etc with an eye to be more usable in Chef and other Configuration Management environments.

### Backwards Compatibility

Unfortunately introducing the new filtering methods has some backward compatibility issues, if you had clients/agents with code like:

{% highlight ruby %}
   options[:filter]["agent"] = "some agent"
{% endhighlight %}

You should now change that to:

{% highlight ruby %}
   options[:filter]["agent"] << "some agent"
{% endhighlight %}

As each filter is an array now.  If you do not change the code it will still work as before but you will not be able to use the compound filtering feature on filter types that you've forced to be a string and there might be some other undesired side effects.  We've tried though to at least not break old code, they just can't use the new features.

You were also able to test easily in the past if you're running unfiltered using
something like:

{% highlight ruby %}
   if options[:filter] == {}
{% endhighlight %}

Now that's much harder and we've added a helper to make this easier:

{% highlight ruby %}
   if MCollective::Util.empty_filter?(options[:filter])
{% endhighlight %}

This new method is compatible with both the old and new filter method so you can start using it before you finish the first issue mentioned here.

We've also made the class filter more generic, in the past you did class filters like this:

{% highlight ruby %}
   options[:filter]["puppet_class"] << /apache/
{% endhighlight %}

Now you have to adjust it to:

{% highlight ruby %}
   options[:filter]["cf_class"] << /apache/
{% endhighlight %}

Old code will keep working but you should change to this name for filters to be consistent with the rest of the code base.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
| 2009/12/28 | Add support for other configuration management systems like chef in the --with-class filters|13|
| 2009/12/28 | Add a _Util.empty`_`filter?_ to test for an empty filter| |
| 2009/12/27 | Create a new client framework SimpleRPCIntroduction|6|
| 2009/12/27 | Add support for multiple filters of the same type|3|

## 0.3.0 - 2009/12/17

### Release Focus and Notes

Primarily a bug fix release.  Only new feature is to allow the user to create _MCollective::Util::`*`_ classes and put those in the plugins directory.  This is useful for more complex agents and clients.

### Backwards Compatibility

This release should not break any older code, if it does it's a bug.

### Changes

|Date|Description|Ticket|
|----|-----------|------|
|2009/12/16|Improvements for newer versions of Ruby where TERM signal was not handled|7|
|2009/12/07|MCollective::Util is now a module and plugins can drop in util classes in the plugin dir| |
|2009/12/07|The Rakefile now works with rake provided on Debian 4 systems|2|
|2009/12/07|Improvements in the RC script for Debian and older Ubuntu systems|5|

## 0.2.0 - 2009/12/02

### Release Focus and Notes

First numbered release

### Backwards Compatibility

n/a

### Changes

n/a
