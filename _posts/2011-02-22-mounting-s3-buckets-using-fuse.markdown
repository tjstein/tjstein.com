---
layout: post
title: Mounting S3 Buckets Using FUSE
excerpt: With the combination of FUSE and s3fs, you can expand your local filesystem into the cloud in minutes.
comments: true
---

Over the past few days, I've been playing around with <a href="http://fuse.sourceforge.net/" rel="external">FUSE</a> and a FUSE-based filesystem backed by Amazon S3, <a href="http://code.google.com/p/s3fs/" rel="external">s3fs</a>. Until recently, I've had a negative perception of FUSE that was pretty unfair, partly based on some of the lousy FUSE-based projects I had come across. I'm sure some of it also comes down to some partial ignorance on my part for not fully understanding what FUSE is and how it works.

<h4>What is FUSE?</h4>

FUSE is a loadable kernel module that lets you develop a user space filesystem framework without understanding filesystem internals or learning kernel module programming. This basically lets you develop a filesystem as executable binaries that are linked to the FUSE libraries. So, if you're not comfortable hacking on kernel code, FUSE might be a good option for you.

<h4>FUSE + Amazon S3</h4>

So, now that we have a basic understanding of FUSE, we can use this to extend the cloud-based storage service, S3. Using a tool like s3fs, you can now mount buckets to your local filesystem without much hassle. Although your reasons may vary for doing this, a few good scenarios come to mind:

* Your server is running low on disk space and you want to expand
* You want to give multiple servers read/write access to a single filesystem
* You want to access off-site backups on your local filesystem without ssh/rsync/ftp

To get started, we'll need to install some prerequisites. I've set this up successfully on Ubuntu 10.04 and 10.10 without any issues:

{% highlight bash %}
aptitude -y install build-essential libfuse-dev fuse-utils libcurl4-openssl-dev 
aptitude -y libxml2-dev mime-support pkg-config
{% endhighlight %}

Now you'll need to download and compile the s3fs source. As of 2/22/2011, the most recent release, supporting reduced redundancy storage, is 1.40. Check out the <a href="http://code.google.com/p/s3fs/downloads/list" rel="external">Google Code</a> page to be certain you're grabbing the most recent release.

{% highlight bash %}
cd /usr/local/src
wget http://s3fs.googlecode.com/files/s3fs-1.40.tar.gz
tar zxvf s3fs-1.40.tar.gz && cd s3fs-1.40
./configure
make && make install
{% endhighlight %}

This will install the s3fs binary in <em>/usr/local/bin/s3fs</em>. You can add it to your .bashrc if needed:

{% highlight bash %}
echo "export s3fs='/usr/local/bin/s3fs'" >> ~/.bashrc
source ~/.bashrc
{% endhighlight %}

Now we have to set the allow_other mount option for FUSE. Using the allow_other mount option works fine as root, but in order to have it work as other users, you need uncomment user_allow_other in the fuse configuration file:

{% highlight perl %}
perl -p -i -e 's|#user_allow_other|user_allow_other|g;' /etc/fuse.conf
{% endhighlight %}

To make sure the s3fs binary is working, run the following:

<pre class="terminal">
root@tjstein.com:~# s3fs
s3fs: missing BUCKET argument
Usage: s3fs BUCKET MOUNTPOINT [OPTION]...
</pre>

So before you can mount the bucket to your local filesystem, create the bucket in the AWS control panel or using a CLI toolset like <a href="http://s3tools.org" rel="external">s3cmd</a>. Then, create the mount directory on your local machine before mounting the bucket:

{% highlight bash %}
s3cmd mb s3://bucketname
mkdir -p /mnt/s3
s3fs bucketname -o use_cache=/tmp -o allow_other /mnt/s3
{% endhighlight %}

To allow access to the bucket, you must authenticate using your AWS secret access key and access key. You can either add the credentials in the s3fs command using flags or use a password file. Depending on what version of s3fs you are using, the location of the password file may differ -- it will most likely reside in your user's home directory or /etc.

To confirm the mount, run <em>mount -l</em> and look for /mnt/s3. 

Notes:
	Fuse performance
	Price of S3 using RRS
	use_cache option