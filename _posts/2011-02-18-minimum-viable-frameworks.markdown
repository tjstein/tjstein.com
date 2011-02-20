---
layout: post
title:  Minimum Viable Frameworks
excerpt: During the process of redesigning my personal blog, I evaluated several pieces of software including Jekyll, Flask and Toto among a few others.
comments: true
---
During the process of redesigning my personal blog, I evaluated several pieces of software including <a href="http://jekyllrb.com/" rel="external">Jekyll</a>, <a href="http://flask.pocoo.org" rel="external">Flask</a> (a Python micro-framework) and <a href="http://cloudhead.io/toto" rel="external">Toto</a> among a few others. I decided to go with Jekyll as it seemed to meet most of my requirements:

* No database; posts are stored in plain text, markdown or some other equivalent
* No server-side language requirements
* Can integrate easily with Git
* Can be themed and extended
* Has well-written documentation and good community adoption

Since the redesign, I've really embraced this type of site-building platform and I think many others are starting to see it's appeal. The popularity of these micro-frameworks has grown tremendously over the last year or two. I've read numerous blog posts from users leaving larger CMS-based blogging platforms like WordPress in favor of more minimalist, git-powered projects. Their reasons for switching vary, but one common theme is apparent: **It's simple.**

>> "I really wanted something simpler than WordPress. I didn’t need a CMS. I barely need a blogging engine. I update so infrequently. I want something that creates well formed html (hah), static content and is easy to use."

**Source: <a href="http://www.nata2.org/2010/08/17/jekyl-for-the-win/" rel="external">Harper Reed</a>**

<h4>Freemium Hosting</h4>

A few web service providers are also catching on, offering extremely affordable (and sometimes free) hosting. Amazon <a href="http://aws.typepad.com/aws/2011/02/host-your-static-website-on-amazon-s3.html" rel="external">recently announced</a> the availability of website hosting on S3, their storage service. Heroku has offered a free, albeit somewhat diminutive, tier for rapid-prototyping, staging, and testing purposes, as well as actually running lightweight apps. <a href="http://code.google.com/appengine/" rel="external">Google App Engine</a> also offers cloud-based solutions, which can be pushed further with tools like <a href="http://drydrop.binaryage.com/" rel="external">DryDrop</a>. DryDrop enables you to host your static site on Google App Engine and update it by pushing to GitHub. That's pretty rad.

If you're looking into micro-frameworks, check out the following for more information:

* <a href="http://fadeyev.net/2010/05/10/getting-started-with-toto/" rel="external">
Getting Started With Toto, a Tiny WordPress Killer</a>
* <a href="http://paulstamatiou.com/how-to-wordpress-to-jekyll" rel="external">How To: WordPress to Jekyll</a>
* <a href="http://www.nata2.org/2010/08/17/jekyl-for-the-win/" rel="external">I migrated this blog to Jekyll on App Engine. So long Wordpress</a>
* <a href="http://recursive-design.com/blog/2010/10/12/static-blogging-the-jekyll-way/" rel="external">Static blogging the Jekyll way</a>
* <a href="http://codebeef.com/migrating-to-jekyll/">An Improved Interface for Migrating to Jekyll</a>