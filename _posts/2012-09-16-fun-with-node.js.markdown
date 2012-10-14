---
layout: post
title: Generating Passwords with node.js
excerpt: Diving into node.js to solve the ancient problem of generating random passwords in the most difficult way possible.
comments: true
---

For a few years now, I've been using the same silly random password generator from a coworker of mine. Since I use 1Password for password management, I'm frequently creating new random passwords. While a feature exists within 1Password for generating new passwords, I usually find comfort in doing this from Terminal. When my coworker's server was down for a few days, I temporarily reverted to some bash-foo:

{% highlight bash %}
date +%s | sha256sum | base64 | head -c 13 ; echo
{% endhighlight %}

{% highlight bash %}
tr -cd '[:alnum:]' < /dev/urandom | fold -w13 | head -n1
{% endhighlight %}

{% highlight bash %}
< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c 13 ; echo
{% endhighlight %}

While these examples are super handy, I wanted to use this as an opportunity to play around with <a href="http://nodejs.org/" target="_new" rel="external">node.js</a>. This simple utility can generate 13 character random passwords using 2 node modules, <a href="https://github.com/visionmedia/express" target="_new" rel="external">express</a> and <a href="https://github.com/mikeal/request" target="_new" rel="external">request</a>. Here is the meat of it:

{% highlight javascript %}
var express, http, make_passwd;

express = require('express');

http = express();
http.use(express.logger());

make_passwd = function(n, a) {
  var index = (Math.random() * (a.length - 1)).toFixed(0);
  return n > 0 ? a[index] + make_passwd(n - 1, a) : '';
};

http.get('/', function(request, response) {
  var password;

  password = make_passwd(13, 'qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM1234567890');

  response.type('text/plain');
  response.send(password);
});

http.listen(process.env.PORT);
{% endhighlight %}

I've published a working version to <a href="http://passwd.tjstein.com" target="_new" rel="external">Heroku</a>. I've also published the repository on <a href="https://github.com/tjstein/passwd" target="_new" rel="external">GitHub</a>. Give it a shot:

{% highlight bash %}
curl –s http://passwd.tjstein.com 2>/dev/null | pbcopy
{% endhighlight %}