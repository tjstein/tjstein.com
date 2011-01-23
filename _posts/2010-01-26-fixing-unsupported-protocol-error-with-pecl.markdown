---
layout: post
title: Fixing Unsupported Protocol Error with PECL
excerpt: A quick how-to on fixing the 'unsupported protocol' issue with PHP 5.2.9 and 5.2.10.
comments: true
---
After setting up PHP 5.2.10, I wanted to install a few additional modules using PECL. I found soon after that I was unable to do so, receiving the following error:

<blockquote>pear.php.net is using a unsupported protocol - This should never happen. install failed</blockquote>

The problem seems to exhibit itself with corrupted PEAR installations in PHP 5.2.9 and 5.2.10. To fix this, just backup and update the channels:

<pre><code class="bash">mv /usr/local/lib/php/.channels /usr/local/lib/php/.channels.bak</code></pre>
<pre><code class="bash">pear update-channels</code></pre>

After that, you should be good to go.