---
layout: post
title: Live Data Replication Using lsyncd
excerpt: One of my favorite Linux utilities I've discovered recently is lsyncd, a live syncing (mirror) daemon. Following the traditional Unix philosophy, it does data replication simply and it does it very well.
comments: true
---

One of my favorite Linux utilities I've discovered recently is <a href="http://code.google.com/p/lsyncd/" title="lsyncd - Live Syncing (Mirror) Daemon" rel="external">lsyncd</a>, a live syncing (mirror) daemon. Following the traditional Unix philosophy, it does data replication simply and it does it very well. Using some fancy inotify magic, lsyncd will spawn one or more processes to synchronize the targets after changes have been made.

After determining that a client would need multiple web-servers running in sync, I evaluated a few different tools that perform data replication and live syncing. This is one of the larger problems I've encountered with scaling web applications horizontally -- storage management among multiple web services. In traditional deployments, it might make sense to use something like NFS or even DRBD, operating on the block device level. While this works for write-heavy systems under high load, it isn't practical mirroring a whole block device for a high-traffic WordPress site -- I just needed a basic client-server model.

This is where lsyncd really shines. The lsyncd configuration file is written in Lua and super easy to set up. Below is the `lsyncd.conf` that duplicates `/var/www/tjstein.com` on the master server to the 4 targets with the same path:

{% highlight lua %}
settings = {
   delay        = 1,
   maxProcesses = 5,
   statusFile   = "/tmp/lsyncd.status",
   logfile      = "/var/log/lsyncd.log",
}

targetlist = {
 "10.0.1.23:/var/www/tjstein.com",
 "10.0.1.24:/var/www/tjstein.com",
 "10.0.1.25:/var/www/tjstein.com",
 "10.0.1.26:/var/www/tjstein.com"
}

for _, server in ipairs(targetlist) do
  sync{ default.rsync,
    source="/var/www/tjstein.com",
    rsyncOpts="-rltvupgo",
    target=server
  }
end
{% endhighlight %}

To make the rsyncing seamless, you'll need to send the public key generated on the master server to all of the duplicated nodes for easy access. With the client I was working with, we were also using Nginx to load balance across the web nodes. Since we only wanted writes going to the primary master server, we redirected all requests to /wp-admin to the primary node. Then, any changes made through the WordPress backend were synced over to the targets in less than 1 second. Here is the Nginx configuration:

{% highlight nginx %}
upstream backend  {
  server 10.0.1.20; #master
  server 10.0.1.21; #www-01
  server 10.0.1.22; #www-02
  server 10.0.1.23; #www-03
}

upstream admin {
  server 10.0.1.20;
}

server {
  server_name www.tjstein.com tjstein.com;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass  http://backend;
  }

  location ~ /wp-admin/* {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass  http://admin;
  }
}
{% endhighlight %}

<h4>Notes</h4>

You'll want to install lsyncd from source on most distributions. Older versions (pre 2.x) contained gross XML style configuration; the most recent version, as of 8/30/2011, is 2.0.5.