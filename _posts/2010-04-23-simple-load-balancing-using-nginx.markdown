---
layout: post
date: 2010-04-23
title: Simple Load Balancing Using Nginx
excerpt: Nginx is a lovely web server. It's also a pretty neat load balancer too.
---
While the concept of load balancing has been around for a while, using Nginx to do this is fairly new to me. Other common load balancers in use today are <a title="LVS" href="http://www.linuxvirtualserver.org/" target="_blank">LVS</a>, <a title="HAProxy" href="http://haproxy.1wt.eu/" target="_blank">HAProxy</a>, <a title="Perlbal" href="http://www.danga.com/perlbal/" target="_blank">Perlbal</a> and <a title="Pound" href="http://www.apsis.ch/pound/" target="_blank">Pound</a>. In this example, I am using 3 <a href="http://www.mediatemple.net/webhosting/ve/" target="_new">(ve) servers</a> from Media Temple, each running on Ubuntu 9.10. To get started, log into the server you want to set up as the load balancer and install Nginx:

<pre><code>aptitude install nginx</code></pre>

Then we'll need to make a new Nginx default virtual host, <em>/etc/nginx/sites-available/default</em>. In the two server directives under the upstream backend section, be sure to put in the IP addresses or hosts you are balancing:

<pre><code class="nginx">upstream backend  {
  server 123.123.123.123;
  server 123.123.123.124;
}

server {
  location / {
    proxy_pass  http://backend;
  }
}</code></pre>

This is the simplest configuration possible. After you've completed the proxy configuration, test and restart Nginx:

<pre><code>nginx -t
/etc/init.d/nginx restart</code></pre>

At this point, requests handled by the load balancer will serve equal requests from each upstream server. The really nice thing is that if one of the upstream servers is not responding, the load balancer will automagically stop routing requests to it. So although the configuration has the unavailable server loaded, Nginx sees that it is down and then routes to all other available upstream servers. If all upstreams are down, Nginx will halt the proxy, simply showing a 502 http error. This also makes it a very useful tool for balancing mongrels for those running rails.

It's also worth noting that you can add as many backend nodes as you want; I'm just using two as an example. This makes using a (ve) server an ideal choice; spin up a new (ve) and simply add it to the upstream pool. There are a few other configuration options that provide the ability to add weight to backends to force an uneven load distribution. You can also use <em>proxy_set_header</em> to make sure the user's IP address is logged instead of localhost:

<pre><code class="nginx">upstream backend  {
  server john.tjstein.com;
  server paul.tjstein.com;
  server ringo.tjstein.com;
  server george.tjstein.com weight=30; # George was always my favorite
}

server {
  server_name www.tjstein.com tjstein.com;
  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass  http://backend;
  }
}</code></pre>

Nginx provides many other configuration directives so I would recommend checking the official Nginx documentation on the <a href="http://wiki.nginx.org/NginxHttpUpstreamModule" target="_blank">NginxHttpUpstreamModule</a> for more information. In the next post, we'll look at using a few file synchronization tools like <a title="Unison" href="http://www.cis.upenn.edu/~bcpierce/unison/" target="_blank">Unison</a> and <a title="rsync" href="http://samba.anu.edu.au/rsync/" target="_blank">rsync</a> for data replication between backends.