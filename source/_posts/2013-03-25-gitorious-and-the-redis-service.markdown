---
layout: post
title: "Gitorious and the Redis service"
date: 2013-03-25 15:03:09 -0500
comments: true
categories: [Git, Gitorious, Redhat, Redis, RHEL]
---
Gitorious has a very nice status command via /usr/bin/gitorious_status that quickly shows you if all your Gitorious services are up and running (see screenshot).

{% img left /images/health_check.png 300 250 %}

It's missing one very important service though, Redis!

Redis is now the default messaging service when you do a fresh install via the [Gitorious CE-Installer](http://getgitorious.com/installer) but it's not included in the status check, it's missing from the /admin/diagnostics page and it's missing a [Monit](http://mmonit.com/monit/) check file to restart it if it dies. Keeping it running is pretty important for a working Gitorious install because without it many of the web page operations like creating a new project or team will fail and it won't be very clear from the logs why it failed.

Lets fix some of those problems.

First off lets patch the gitorious_status script with the patch below which should work on any modern Linux variant.

Save the lines below as: **/tmp/my.patch**

```
*** gitorious_status 2013-03-25 16:11:04.475121039 -0500
--- gitorious_status_redis 2013-03-25 16:16:35.380135982 -0500
*************** unicorn_status() {
*** 26,31 ****
--- 26,35 ----
      check_process_and_report "ps -p $PID" "Unicorn"
  }

+ redis_status() {
+     check_process_and_report "/etc/init.d/redis status" "Redis"
+ }
+
  # Upstart's exit codes are a beast of its own
  resque_status() {
      check_process_and_report "/sbin/initctl status resque-worker" "Resque"
*************** sphinx_status
*** 80,82 ****
--- 84,87 ----
  memcached_status
  sshd_status
  mysqld_status
+ redis_status

```

Apply your patch to the status command:

     patch /usr/bin/gitorious_status < /tmp/my.patch

Your status command should now show the details of the Redis service as shown below.

{% img left /images/redis-status.png 300 250 %}

Next lets create a Monit config file for Redis which will watch the process and restart if needed.
I use Puppet and a custom Monit module I wrote for this but it's not required.
For now lets just manually create the file and you can integrate it into your configuration management tool later if you like. Note that the Monit file below is specific to Redhat, change the pidfile and start/stop lines as needed to match your OS.

Copy the contents below into:
   **/etc/monit.d/redis.monit**

```
check process redis-server with pidfile /var/run/redis/redis.pid
  start program = "/sbin/service redis start"
  stop program = "/sbin/service redis stop"
  if does not exist for 1 cycles then restart
  if 5 restarts within 5 cycles then alert
```

Now lets restart Monit so it picks up the change.
On Redhat that's done like so:

    service monit restart

Check that it's setup correctly in Monit:

    monit summary

OR

    monit status

So now you have Redis monitored by Monit and the gitorious_status command shows you if it's up or down but your /admin/diagnostics page is still missing any status about it. That last bit is not too hard to fix but for now I'm leaving that up to the Gitorious folks to patch along with the incorrect status about the gitorious-poller service which is not in use any longer.

