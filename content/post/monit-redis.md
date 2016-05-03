+++
date = "2014-03-20T01:11:51-04:00"
draft = false
title = "Monit: The Quick Fix"

+++

Ideally, in a production system, everything works perfectly. Services never mysteriously crash, free memory is constantly available, and CPU load rarely spikes above 50%. Unfortunately, this is not always the case.

<!--more-->

Recently, a client was having issues with their site going down, consistently. They knew it was due to the redis service stopping, but didn't know how to fix it. As a result, one person was perpetually on call, waiting for the site to go down, to run a simple `sudo service redis restart`.

While it won't fix all of your problems, monit will buy you time, and your sysadmins sleep.

### Installing Monit

On CentOS

```bash
# First, install the EPEL repository

wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm

sudo yum install monit
```

On Ubuntu

```bash
sudo apt-get install monit
```

This process creates a configuration file, and adds the command `monit` to the path.

### Configuring Monit

Monit config files are located in different places depending on the system. Ubuntu's config file is located at `/etc/monit/monitrc`, whereas CentOS uses `/etc/monitrc`.

Monit config files follow a simple structure, of the form

```bash
check process $processName with pidfile $pidfilePath
  start program = $commandToStart
  stop  program = $commandToStop
  if failed host localhost port $portNumber then restart
  $timeoutConditions
```

As an example, below is how we have configured monit to watch redis on a CentOS box that was having memory issues, causing redis to force quit.

```bash
check process redis-server with pidfile /var/run/redis/redis.pid
  start program = "/etc/init.d/redis start"
  stop  program = "/etc/init.d/redis stop"
  if failed host localhost port 6379 then restart
  if 5 restarts within 5 cycles then timeout
```

In general, monit is a good tool to have in place. Though hopefully it is rarely needed, it prevents issues where someone is called at 3am in order to run a simple command, due to a service crash.

Originally Posted at http://spantree.net/blog/2014/03/20/monit.html
