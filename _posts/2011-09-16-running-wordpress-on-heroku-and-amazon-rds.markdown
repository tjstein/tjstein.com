---
layout: post
title: Running WordPress on Heroku + Amazon RDS
excerpt: With the announcement of the Heroku and Facebook partnership yesterday, Heroku quietly confirmed support for two new languages, Python and PHP. The self-proclaimed polyglot platform, Heroku's Celadon Cedar Stack now has a considerable advantage over newcomers like PHP Fog, and Orchestra.io by avoiding mostly separate, language-specific products.
comments: true
---

<img src="/images/heroku-logo.png" class="alignleft">With the <a href="http://blog.heroku.com/archives/2011/9/15/facebook/" target="_new" rel="external">announcement</a> of the Heroku and Facebook partnership yesterday, Heroku quietly confirmed support for two new languages, Python and PHP. The self-proclaimed polyglot platform, Heroku's <a href="http://devcenter.heroku.com/articles/cedar" target="_new" rel="external">Celadon Cedar Stack</a> now has a considerable advantage over newcomers like <a href="https://phpfog.com/" target="_new" rel="external">PHP Fog</a>, and <a href="http://orchestra.io/" target="_new" rel="external">Orchestra.io</a> by avoiding mostly separate, language-specific products.

To test out Heroku's PHP support (version 5.3.6), I deployed WordPress. One of the current limitations is the lack of native MySQL support, outside of hooking into the <a href="http://xeround.com/" target="_new" rel="external">Xeround Cloud DB</a> or <a href="http://aws.amazon.com/rds/" target="_new" rel="external">Amazon RDS</a>. I've found one blog post that suggested using the default PostgreSQL backend that Heroku provides with the 'PG4WP' WordPress plugin, enabling WordPress to be used with a PostgreSQL database. While this may work, it lacks most plugin support and is more of a bandaid for the platform limitations. Instead, you can use Amazon's Relational Database Service (RDS) addon.

Amazon RDS is a service that allows you to set up, operate and scale a dedicated MySQL database server on top of EC2. In addition to standard MySQL features, RDS offers the following functionality:

* Automated backups
* Point-in-time recovery
* Seamless vertical scaling between instance types

The free Amazon RDS add-on lets you connect your Heroku app to an RDS instance and seamlessly use it in place of the standard, Heroku-provided PostgreSQL database. To get started, you should configure the RDS <a href="http://docs.amazonwebservices.com/AmazonRDS/latest/CommandLineReference/" target="_new" rel="external">command line toolkit</a> and <a href="http://devcenter.heroku.com/articles/heroku-command" target="_new" rel="external">Heroku gem</a> if you haven't already. Let's start by creating the RDS database instance on your local machine:

{% highlight bash %}
rds-create-db-instance --db-instance-identifier [name]\
  --allocated-storage 5 \
  --db-instance-class db.m1.small  \
  --engine MySQL5.1 \
  --master-username [user] \
  --master-user-password [pw] \
  --db-name [name] \
  --headers
{% endhighlight %}

This will take a few minutes. Once the database is provisioned, add your local IP address to the security group -- assuming your workstation’s public IP is 1.1.1.1:

{% highlight bash %}
rds-authorize-db-security-group-ingress default --cidr-ip 1.1.1.1/32
{% endhighlight %}

Heroku also needs to be able to access your RDS instance. To allow Heroku’s cloud through the RDS firewall, run the following command:

{% highlight bash %}
rds-authorize-db-security-group-ingress default \
    --ec2-security-group-name default \
    --ec2-security-group-owner-id 098166147350
{% endhighlight %}

Now we can begin building the application layer. Since we're going to be using Git for version control, I'd suggest cloning WordPress from Mark Jaquith's <a href="https://github.com/markjaquith/WordPress" target="_new">GitHub</a> repository; it is synced from Automattic's SVN repository every 30 minutes, including branches and tags:

{% highlight bash %}
git clone git://github.com/markjaquith/WordPress.git
cd WordPress
{% endhighlight %}

Before we start making any changes to file structure, we should make our own Git repository and start committing. Note here that you'll want to populate the wp-config.php with the MySQL credentials from the `rds-create-db-instance` command above:

{% highlight bash %}
git init
mv wp-config-sample.php wp-config.php
git add .
git commit -m 'initial commit'
{% endhighlight %}

Now, create the stack and enable the RDS addon with your MySQL credentials. Once the stack has been created, you can deploy:

{% highlight bash %}
heroku create --stack cedar
heroku addons:add amazon_rds url=mysql://user:pass@rdshostname.amazonaws.com/databasename
git push heroku master
{% endhighlight %}

The output should look something like this:

<pre class="terminal">
➜ wp-heroku-test git:(master) git push heroku master
Counting objects: 985, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (965/965), done.
Writing objects: 100% (985/985), 3.65 MiB | 221 KiB/s, done.
Total 985 (delta 66), reused 0 (delta 0)

-----> Heroku receiving push
-----> PHP app detected
-----> Bundling Apache v2.2.19
-----> Bundling PHP v5.3.6
-----> Discovering process types
       Procfile declares types -> (none)
       Default types for PHP   -> web
-----> Compiled slug size is 24.9MB
-----> Launching... done, v4
       http://evening-waterfall-3372.herokuapp.com deployed to Heroku

To git@heroku.com:evening-waterfall-3977.git
 * [new branch]      master -> master
</pre>

<h4>Notes</h4>

* A commenter on HackerNews noted that the slug is still read-only, but the ephemeral filesystem is writable. The slug is what gets deployed on each new dyno spawned. The ephemeral filesystem is the individual file system on each dyno. So a plugin like WP Super Cache would be able to write to the file system, but that cache would only exist for the individual dyno that wrote it.

* Because of the usage concerns of media and content uploads, I'd suggest using a CDN or Amazon S3 for storing images and attachments.

* The zlib extension for PHP is not compiled on the Celadon Cedar Stack. Theme and plugin uploads through the WordPress admin panel will fail. To get around this, set up your themes and plugins on your local workstation first, commit and deploy.

* Don't have a phpinfo page in the document root as it will contain your database crendentials in plain text.

* If you want to have pretty permalinks, create the .htaccess file on your local machine and populate the mod_rewrite rules prior to deploying.