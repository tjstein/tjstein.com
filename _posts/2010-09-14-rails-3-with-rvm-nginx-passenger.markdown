---
layout: post
date: 2010-09-14
title: Rails 3.0 with RVM, Nginx and Phusion Passenger
excerpt: An illustrated guide on setting up the latest rails release using RVM, Nginx and Phusion Passenger.
---

Rails 3.0 is ponies and rainbows! Well, kinda. I've never been into Rails much but 3.0 has brought on some much needed features over previous versions, some of which deserve bullet points:

<ul>
	<li>Brand new router with an emphasis on RESTful declarations</li>
	<li>Unobtrusive JavaScript helpers with drivers for Prototype, jQuery,  and more coming (end of inline JS)</li>
	<li>Explicit dependency management with <a href="http://gembundler.com/" target="_blank">Bundler</a></li>
</ul>

The newest version of Rails came out on August 29 so I decided to jump right in. Since Ruby 1.8.6 and earlier are no longer supported, we'll be using RVM in this article. <a href="http://rvm.beginrescueend.com/" target="_blank">RVM</a> is a command line tool which allows us to easily install, manage and  work with multiple ruby environments from interpreters to sets of gems.

Before we get started, we'll be using a fresh (ve) Server from <a href="http://mediatemple.net/" target="_blank">(mt) Media Temple</a> running Ubuntu 10.04 Lucid Lynx for the install. I should also note that we're not installing a full-featured application -- this is simply a tutorial on how to get the Rails 3.0 test application up and running with Nginx + Passenger.

<strong>Prerequisites</strong>: You'll need these to get through the entire install process for RVM, Nginx and <a href="http://www.modrails.com/">Phusion Passenger</a> (mod_rails).

<pre><code class="bash">aptitude -y install curl git-core build-essential zlib1g-dev libssl-dev</code></pre>
<pre><code class="bash">aptitude -y install libreadline5-dev libc6 libpcre3 libssl0.9.8 zlib1g</code></pre>

Now we'll need to do a system-wide install of RVM. For your convenience, the dude that wrote RVM provided a script you can run on most that does all the heavy lifting:

<pre><code class="bash">bash &lt; &lt;( curl -L http://bit.ly/rvm-install-system-wide )</code></pre>

Every piece of documentation and website I read on this had this particular part wrong. I've updated the correct path from the documentation for the purposes of this article:

<pre><code class="bash">echo "[[ -s '/usr/local/lib/rvm' ]] &amp;&amp; source '/usr/local/lib/rvm'" &gt;&gt; ~/.bashrc
source ~/.bashrc</code></pre>

Doing so ensures RVM is loaded as a function (versus as a binary), ensuring commands such as <em>rvm use</em> work as expected. Please note that you can confirm this worked correctly by opening a new shell and running:

<pre><code class="bash">type rvm | head -n1</code></pre>

If this was performed correctly, you should see:

<pre><code class="bash">rvm is a function</code></pre>

So if that looks good, we can continue installing Ruby:

<pre><code class="bash">rvm install 1.9.2</code></pre>

Now sit back and relax. This took about 8 minutes on my system. To make sure that the system uses this version, we can use RVM for this and install the rails and passenger gems:

<pre><code>rvm --default ruby-1.9.2
gem install rails passenger</code></pre>

Now comes the fun part of installing Nginx. This will grab the stable binary, bake in Phusion Passenger and compile it:

<pre><code>rvmsudo passenger-install-nginx-module</code></pre>

You will be presented with two options right from the start. Just choose Option 1 and let passenger download, compile and install Nginx for you. You'll also be prompted to choose a prefix directory. You can leave it as the default, /opt/nginx. When it completes, it should give you some Nginx configuration snippets for your rails apps. Save those as you'll need them later.

Now, you'll want a init script so you can stop/start/restart Nginx. I've provided mine here:

<pre><code>cd /etc/init.d
wget -O nginx http://bit.ly/8XU8Vl
chmod +x nginx
/usr/sbin/update-rc.d -f nginx defaults</code></pre>

We're almost done. Now we can create the test application:

<pre><code class="bash">cd /opt/nginx/html
rails new testapp</code></pre>

You'll need to update the Nginx configuration to make sure it's using the right document root now. The file will be located in <em>/opt/nginx/conf/nginx.conf</em>:

<pre><code class="bash">sed -i".bak" '47d' /opt/nginx/conf/nginx.conf
sed -i '47 a\
            root   /opt/nginx/html/testapp/public/;' /opt/nginx/conf/nginx.conf
/etc/init.d/nginx start</code></pre>

The spacing looks a little weird in that middle sed command for a purpose -- to keep the syntax of the nginx.conf file consistent. Now that we've restarted Nginx, we should see the 'Welcome aboard: You're riding Ruby on Rails!' image:

<center><img title="You're riding Ruby on Rails!" src="/images/rails3.jpg" alt="You're riding Ruby on Rails!" width="567" height="271" /></center>

If you're interested in an all-in-one installation, I've put together the steps above into a <a href="http://gist.github.com/raw/578736/0932236d03b18aff32361ec4718653c12b167209/rails3-nginx-passenger.sh" target="_new">simple bash script</a>. Enjoy!