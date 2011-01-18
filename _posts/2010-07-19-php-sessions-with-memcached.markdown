---
layout: post
date: 2009-07-19
title: PHP Sessions with Memcached
excerpt: Sessions, in many regards, would seem ideal to cache because they are transient, relatively small, and accessed frequently.
---
After setting up PHP 5.3.0 on a Debian virtual machine recently, I had some trouble implementing XCache, an opcode cacher for PHP. I've used XCache with Lighttpd in the past however now I am working with Nginx and using a compiled version of PHP instead of the Debian <a href="http://packages.debian.org/lenny/web/php5" target="_new">package</a>. The issue is that the current XCache release has not been adapted to support late static binding and namespaces of PHP 5.3.0. I've even compiled the <a href="http://xcache.lighttpd.net/pub/Releases/" target="_new">2.0</a> beta build of XCache however this was still problematic.

I decided instead to focus on session management first and build up a caching daemon to store PHP sessions. Sessions, in many regards, would seem ideal to cache because they are transient, relatively small, and accessed frequently. I'm not overly concerned with disk I/O but I'm always interested in finding new, or better, ways to improve server performance. Enter <a href="http://www.danga.com/memcached/" target="_new">memcached</a>.

Memcached is a general-purpose distributed memory caching system that was originally developed by <a href="http://www.danga.com/" target="_new">Danga Interactive</a> for LiveJournal, but is now used by many other sites. It is often used to speed up dynamic database-driven websites by caching data and objects in memory to reduce the number of times an external data source must be read. The system is used by several very large, well-known sites including YouTube, Wikipedia, Facebook, Digg, and Twitter. For a great article on how memcached was implemented into Facebook's architecture, check out <a href="http://www.facebook.com/note.php?note_id=39391378919" target="_new">Scaling memcached at Facebook</a>.

So, to get started, I'll outline how to install memcached from source and also the aptitude package manager. To install from source, use the following for reference:

<pre><code class="bash">wget http://pecl.php.net/get/memcache-3.0.4.tgz
tar xvfz memcache-3.0.4.tgz
cd memcache-3.0.4
phpize
./configure
make &amp;&amp; make test
make install</code></pre>

Note: When you install from source, it will provide the memcached extension directory, which is usually something like the following:

<pre><code>extension_dir = "/usr/local/lib/php/extensions/no-debug-non-zts-20090626/"</code></pre>

If you are on a Debian based system, you can also use aptitude instead of installing from source:

<pre><code class="bash">aptitude install memcached</code></pre>

Start the daemon:

<pre><code class="bash">/etc/init.d/memcached start</code></pre>

The default port is 11211 so if you want to verify that everything has been set up properly, use netstat:

<pre><code class="bash">netstat -an | grep :11211</code></pre>

The default configuration for memcached was adequate in my case however if you want to modify the connection port or memory limit, you can edit the memcached.conf -- /etc/memcached.conf. Now we need the memcached daemon to talk to PHP, and vice versa. For that we need the memcached PHP module. Again, this can be done with PECL or the package manager:

<pre><code class="bash">aptitude install php5-memcache</code></pre>

If the memcached extension hasn't already been loaded, make sure the following is in the php.ini:

<pre><code class="ini">display_startup_errors = On
extension = memcache.so
memcache.hash_strategy = &quot;consistent&quot;
memcache.max_failover_attempts = 100
memcache.allow_failover = 1
extension_dir = &quot;/usr/local/lib/php/extensions/no-debug-non-zts-20090626/&quot;
session.save_handler = memcache
session.save_path = &quot;tcp://127.0.0.1:11211?persistent=1&amp;weight=1&amp;timeout=1&amp;retry_interval=15&quot;</code></pre>

The <em>extension_dir</em> may vary depending on which version you are running so make sure the path is valid. You can also see we've defined the <em>session.save_handler</em> and <em>session.save_path</em> to use the memcached. Once you've saved the changes to the php.ini, restart your server and php if it runs as it's own process (php-fcgi or php-fpm):

<pre><code class="bash">/etc/init.d/nginx restart &amp;&amp; /etc/init.d/php-fpm restart</code></pre>

To verify that everything has been set up properly, load a <a href="http://us3.php.net/phpinfo" target="_new">phpinfo</a> page in the document root. You should have the memcached PHP module displayed. The session settings should be changed as well:

To test the new PHP session configuration, I've found <a href="http://www.forum.flashloaded.com/boards/showthread.php?t=15850">2 great scripts</a> that can be used to track PHP sessions. I'd also recommend checking out the official memcached <a href="http://code.google.com/p/memcached/wiki/Start" target="_new">documentation</a> as this is only a fraction of what memcached is capable of doing.