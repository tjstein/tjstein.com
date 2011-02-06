---
layout: post
title: APC Plugin For Munin
excerpt: Monitoring APC cache with Munin.
comments: true
---

If you're like me, you want to monitor and graph every little piece of your server. Until recently, I've been relying on a simple PHP script to display APC statistics. Although the tool worked well, I wanted to integrate the statistics into Munin, as most all other server properties and system services are monitoring with it. The problem was that finding a suitable APC plugin for Munin proved to be very difficult. Fortunately I stumbled across <a href="http://geb.german-elite.net/blog.php?b=106" target="_new">this</a> German gem (Vielen Dank, dass Sie  meine deutsche Freundin!) that met my needs. To set this up on your system, just follow the directions below:

First, let's set up the apcinfo.php file in the document root of your web server and retrieve the file from a private gist:

{% highlight bash %}
cd /var/www/html
wget -O apcinfo.php http://goo.gl/Lgfmz
{% endhighlight %}

Now, make the file executable:

{% highlight bash %}
chmod +x apcinfo.php
{% endhighlight %}

The script itself is very simple. It simply ouputs the total cache memory, memory available and memory used:

{% highlight php %}
<?php
$mem = apc_sma_info();
$mem_size = $mem['num_seg']*$mem['seg_size'];
$mem_avail= $mem['avail_mem'];
$mem_used = $mem_size-$mem_avail;
$out = array(
    'size: ' . sprintf("%.2f", $mem_size),
    'used: ' . sprintf("%.2f", $mem_used),
    'free: ' . sprintf("%.2f", $mem_avail)
    );

echo implode(' ', $out);
{% endhighlight %}

To confirm that the script is working, check out /apcinfo.php on your site. You should see something like the following:

<blockquote>size: 31457200.00 used: 19478104.00 free: 11979096.00</blockquote>

Now, let's set up the Munin plugin that will use that information:

{% highlight bash %}
cd /usr/share/munin/plugins
wget -O apc http://goo.gl/gUgkj 
chmod +x apc
ln -s /usr/share/munin/plugins/apc /etc/munin/plugins/apc
{% endhighlight %}

What that does is create the apc plugin, makes it executable and then symlinks it to the proper directory. Now, we'll need to make sure Munin loads the new plugin. To do this, we'll edit the <em>/etc/munin/plugin-conf.d/munin-node</em> to add the following:

{% highlight bash%}
[apc*]
user root
env.url http://example.com/apcinfo.php
{% endhighlight %}

Obviously, you'll want to replace example.com with the host you intend to use. Once the file has been updated and saved, restart Munin:

{% highlight bash %}
/etc/init.d/munin-node restart
{% endhighlight %}

Once the node is running, it'll take a few minutes for Munin to update so don't worry if you don't see any new graphs right away. One thing to keep in mind is that you will need the <a href="http://search.cpan.org/~gaas/libwww-perl-5.836/lib/LWP/UserAgent.pm" target="_new">LWP::UserAgent</a> Perl module. If you are on a Debian/Ubuntu OS, just run the following to install it:

{% highlight bash %}
aptitude install libwww-perl
{% endhighlight %}

Now, go check your pretty graphs.