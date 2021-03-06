Backupninja Module
-------------------

[![Puppet Forge](http://img.shields.io/puppetforge/v/gnubilafrance/backupninja.svg)](https://forge.puppetlabs.com/gnubilafrance/backupninja)
[![Build Status](https://travis-ci.org/gnubila-france/puppet-backupninja.png?branch=master)](https://travis-ci.org/gnubila-france/puppet-backupninja)

Fork of the official riseuplabs/backupninja module as it is inactive.

This module helps you configure all of your backups with puppet, using 
backupninja!

! Upgrade notice !

If you were previously using this module, some pieces have changed,
and you need to carefully change your use of them, or you will find
your backups to be duplicated on your backup server. The important
part that changed has to do with the rdiff-backup handler, if you
weren't using that, you don't need to worry.

If you were, you will need to make sure you change all of your
"$directory" parameters to be "$home" instead, and on your
backupserver you will need to move all of your backups into
"$home"/rdiff-backup. Previously, they were put in "$directory", which
doubled as the home for the user that was created. This caused
problems with rdiff-backup because of dot files and other things which
were not part of any rdiff-backup.

Getting started
---------------

First you will need to import the module:

  import "backupninja"

Configure your backup server
----------------------------

Now you will need to configure a backup server by adding the following
to your node definition for that server:
  
  include backupninja::server

By configuring a backupninja::server, this module will automatically
create sandboxed users on the server for each client for their
backups. 

You may also want to set some variables on your backup server, such as:

  $backupdir = "/backups"


Configure your backup clients
-----------------------------

The backupninja package and the necessary backup software will be
installed automatically when you include any of the different handlers
(as long as you are not handling it elsewhere in your manifests), for
example:

include backupninja::client::rdiff_backup

In this case, the module will make sure that the backupninja package
and the required rdiff-backup package are 'installed'/'present' (using
puppet's ensure parameter language). If you need to specify a specific
version of either backupninja itself, or the specific programs that
the handler class installs, you can specify the version you need
installed by providing a variable, for example:

$backupninja_ensure_version = "0.9.7~bpo50+1"
$rdiff_backup_ensure_version = "1.2.5-1~bpo40+1"
$rsync_ensure_version = "3.0.6-1~bpo50+1"
$duplicity_ensure_version = "0.6.04-1~bpo50+1"
$debconf_utils_ensure_version = "1.5.28"
$hwinfo_ensure_version = "16.0-2"

If you do not specify these variables the default 'installed/present'
version will be installed when you include this class.

Configuring handlers
--------------------

Depending on which backup method you want to use on your client, you
can simply specify some configuration options for that handler that are
necessary for your client.

Each handler has its own configuration options necessary to make it
work, each of those are available as puppet parameters. You can see
the handler documentation, or look at the handler puppet files
included in this module to see your different options.

Included below are some configuration examples for different handlers.

* An example mysql handler configuration:

backupninja::mysql { all_databases:
	user => root,
	backupdir => '/var/backups',
	compress => true,
	sqldump => true
}

* An example rdiff-backup handler configuration:

backupninja::rdiff { backup_all:
	directory => '/media/backupdisk',
	include => ['/var/backups', '/home', '/var/lib/dpkg/status'],
	exclude => '/home/*/.gnupg'
}

* A remote rdiff-backup handler:

    backupninja::rdiff { "main":
        host => "backup.example.com",
        type => "remote",
        directory => "/backup/$fqdn",
        user => "backup-$hostname",
    }


Configuring backupninja itself
------------------------------

You may wish to configure backupninja itself. You can do that by doing
the following, and the /etc/backupninja.conf will be managed by
puppet, all the backupninja configuration options are available, you
can find them inside this module as well.

For example:

backupninja::config { conf:
	loglvl => 3,
	usecolors => false,
	reportsuccess => false,
	reportwarning => true;
}


Nagios alerts about backup freshness
------------------------------------

If you set the $nagios_server variable to be the name of your nagios
server, then a passive nagios service gets setup so that the backup
server pushes checks, via a cronjob that calls
/usr/local/bin/checkbackups.pl, to the nagios server to alert about
relative backup freshness.

To use this feature a few pre-requisites are necessary:

 . configure nsca on your backup server (not done via puppet yet)
 . configure nsca on your nagios server (not done via puppet yet)
 . server backup directories are named after their $fqdn
 . using nagios2 module, nagios/nagios3 modules/nativetypes not supported yet
 . using a nagios puppet module that can create passive service checks
 . backups must be under $home/dup, $home/rdiff-backup depending on method
 . $nagios_server must be set before the class is included


