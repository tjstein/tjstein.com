---
layout: post
date: 2010-09-25
title: PHP-FPM Plugin Suite For Munin
excerpt: This is the first release of PHP-FPM plugins for Munin, available now on GitHub.
---
Since switching from a spawn-fcgi implementation several months ago, I’ve been really pleased with PHP-FPM. Given some of the new statistical features included in newer versions (5.3.2+), I put together a plugin suite for Munin. I am not proficient in Perl so I encourage feedback and suggestions to make these better. This suite contains plugins to measure average process size, total memory usage, connection count, process count and most importantly, pool status.

Before installing, make sure you have the most recent version of PHP-FPM:

<pre class="terminal">
[www-data@lenny:/var/www]# php-fpm -v
PHP 5.3.2-1ubuntu4.5ppa5~lucid1 (fpm-fcgi) (built: Sep 22 2010 08:04:01)
Copyright (c) 1997-2009 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2010 Zend Technologies
    with Suhosin v0.9.29, Copyright (c) 2007, by SektionEins GmbH
</pre>

Now we can get into our plugin directory, clone the plugins from my Git repo and make them executable:

{% highlight bash %}
cd /usr/share/munin/plugins
git clone git://github.com/tjstein/php5-fpm-munin-plugins.git
chmod +x PHP5-FPM-Munin-Plugins/phpfpm_*
{% endhighlight %}

To enable the plugins, you’ll need to create these symlinks:

{% highlight bash %}
ln -s /usr/share/munin/plugins/PHP5-FPM-Munin-Plugins/phpfpm_average /etc/munin/plugins/phpfpm_average
ln -s /usr/share/munin/plugins/PHP5-FPM-Munin-Plugins/phpfpm_connections /etc/munin/plugins/phpfpm_connections
ln -s /usr/share/munin/plugins/PHP5-FPM-Munin-Plugins/phpfpm_memory /etc/munin/plugins/phpfpm_memory
ln -s /usr/share/munin/plugins/PHP5-FPM-Munin-Plugins/phpfpm_status /etc/munin/plugins/phpfpm_status
ln -s /usr/share/munin/plugins/PHP5-FPM-Munin-Plugins/phpfpm_processes /etc/munin/plugins/phpfpm_processes
{% endhighlight %}

**Note**: For the phpfpm_status and phpfpm_connections  plugins, you’ll need to enable the status feature included in newer versions (5.3.2+) of PHP-FPM. Open up the php5-fpm.conf, in /etc/php5/fpm, and uncomment line 186 for the pm.status_path directive:

{% highlight bash %}
pm.status_path = /status
{% endhighlight %}

Jérôme Loyet from the Nginx forums provided some useful insight on how to get this working with Nginx. You’ll essentially set up the status location directive like this:

{% highlight nginx %}
location ~ ^/(status|ping)$ {
    include fastcgi_params;
    fastcgi_pass backend;
    fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
    allow 127.0.0.1;
    allow stats_collector.localdomain;
    allow watchdog.localdomain;
    deny all;
}
{% endhighlight %}

You’ll need to make sure that from within your box, you can curl /status with # curl http://localhost/status. You should get a response similar to this:

{% highlight bash %}
accepted conn: 40163
pool: www
process manager: dynamic
idle processes: 6
active processes: 0
total processes: 6
{% endhighlight %}

Once the plugins have been set up, you run any of the plugins manually using the munin-run command line tool:

{% highlight bash %}
cd /etc/munin/plugins && munin-run phpfpm_status
{% endhighlight %}

The output should look like this:

{% highlight bash %}
idle.value 6
active.value 0
total.value 6
{% endhighlight %}

**Note**: The phpfpm_status plugin is particularly useful if you’re using dynamic process management. You can choose static or dynamic in the php5-fpm.conf.

**Requirements**: libwww-perl