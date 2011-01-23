---
layout: post
title: Using WebFont Loader
excerpt: Fighting FOUT with WebFont Loader.
comments: true
---

In the process of designing this site, I decided to use some custom fonts using the @font-face CSS attribute. In comparison to some of the other web font options (Cufon, sIFR, FLIR), the @font-face CSS method is simple, easy to implement and well supported in most modern browsers. Although the <a href="http://code.google.com/apis/webfonts/" title="Google Font API" rel="external">Google Font API</a> is probably the most well-known web font library, I decided to roll my own kit from Font Squirrel and self-host the fonts. The really nice part about <a href="http://www.fontsquirrel.com/" title="Font Squirrel" rel="external">Font Squirrel</a> is that they provide all of the different font formats (TTF, EOT, OTF, and SVG), compatible with every browser on the market.

For all of my H1-H6 headings, I use the following markup:

<pre><code class="css">@font-face {
	font-family: 'YanoneKaffeesatzRegular';
	src: url('YanoneKaffeesatz-Regular-webfont.eot');
	src: local('☺'), url('/fonts/YanoneKaffeesatz-Regular-webfont.woff') format('woff'), url('/fonts/YanoneKaffeesatz-Regular-webfont.ttf') format('truetype'), url('/fonts/YanoneKaffeesatz-Regular-webfont.svg#webfont1BSMunJa') format('svg');
	font-weight: normal;
	font-style: normal;
}</code></pre>

I then load the font in like I would any other family:

<pre><code class="css">h1, h2, h3, h4, h5, h6{font-family:YanoneKaffeesatzRegular, Arial, Helvetica, sans-serif;font-weight:normal;color:#111;}</code></pre>

Everything looked great besides one small, yet extremely annoying, caveat.

I noticed in Firefox and Opera that for a split second, just before the page finished rendering, unstyled text would be displayed. This drove me crazy. Some subsequent Googling let me know that this was common, often referred to as FOUT (Flash of Unstyled Text). The fix was pretty trivial. What you can do is use the WebFont Loader from Google & Typekit.

The <a href="http://code.google.com/apis/webfonts/docs/webfont_loader.html" title="WebFont Loader - Google Font API - Google Code" rel="external">WebFont Loader</a> is a JavaScript library that gives you more control over font loading than the Google Font API provides. The key is using the events system to hide the font until it's ready to be shown. There are quite a few implementation options so I'd suggest checking out some of the following articles for help:

<ul>
	<li><a href="http://24ways.org/2010/using-the-webfont-loader-to-make-browsers-behave-the-same" title="Using the WebFont Loader to Make Browsers Behave the Same" rel="external">Using the WebFont Loader to Make Browsers Behave the Same</a>
	</li>
	<li><a href="http://sixrevisions.com/css/font-face-guide/" title="The Essential Guide to @font-face" rel="external">The Essential Guide to @font-face</a>
	</li>
	<li><a href="http://code.google.com/apis/webfonts/docs/webfont_loader.html" title="WebFont Loader - Google Font API - Google Code" rel="external">WebFont Loader - Google Font API - Google Code</a>
	</li>
</ul>

**Note**: If you see a horizontal scroll-bar after implementing the WebFont Loader on body text, try adding overflow attributes:

<pre><code class="css">body {font-size: 75%; font-family:DroidSansRegular; overflow: -moz-scrollbars-vertical; overflow-x: hidden; overflow-y: scroll;}
</code></pre>