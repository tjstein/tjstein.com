---
layout: post
title: APC Cache Purge Plugin for WordPress
excerpt: After a recent WordPress update, I noticed that the admin panel did not correctly reflect the new version as indicated in the footer of the backend panel. I realized that the APC opcode cache was not flushed and therefore held on to cached versions of many updated files. I thought about what other ways this cache could be flushed without restarting the web server or PHP daemon (PHP-FPM) without compromising service availability.
comments: true
---

After a recent WordPress update, I noticed that the admin panel did not correctly reflect the new version as indicated in the footer of the backend panel. I realized that the <a href="http://php.net/manual/en/book.apc.php" title="APC" target="_new">APC opcode cache</a> was not flushed and therefore held on to cached versions of many updated files. I thought about what other ways this cache could be flushed without restarting the web server or PHP daemon (PHP-FPM) without compromising service availability.

Some subsequent googling later, I found a <a href="http://konstruktors.com/blog/wordpress/2382-clear-apc-cache-button-for-wordpress/
" title="‘Clear APC Cache’ Button for WordPress" target="_new">dirty hack</a> that, although helpful, was somewhat limited by design -- it suggests adding the script to the functions.php file of your theme. This means you'd need to manually insert the code for each theme. Also, `apc_clear_cache()` did not work by default with APC version 3.1.3p1. From this, I created a simple, single purpose WordPress plugin to flush the APC cache.

Once activated, you'll get a 'Purge APC' option under the tools menu. Once the option is clicked, the APC cache is flushed and the entries are displayed on the page:

Feel free to fork or submit pull requests to the <a href="https://github.com/bummercloud/apc-cache-purge" title="GitHub" target="_new">GitHub repository</a>.

{% highlight php %}
<?php
/**
 * @package APC Cache Purge
 * @version 0.1
 */
/*
Plugin Name: APC Cache Purge
Plugin URI: http://tjstein.com
Description: This is a simple, single purpose plugin to flush the APC cache.
Author: TJ Stein, inspired by Kaspars Dambis of konstruktors.com
Version: 0.1
Author URI: http://tjstein.com
License: GPLv2
*/
function apc_purge() {
	return apc_clear_cache('opcode');
}
// Add Purge APC menu under Tools menu
add_action('admin_menu', 'php_apc_info');
        
function php_apc_info() {
	add_submenu_page('tools.php', 'Purge APC', 'Purge APC', 'activate_plugins', 'flush_php_apc', 'php_apc_options');
}
        
function php_apc_options() {
	if (apc_purge() && apc_purge('user'))
		print '<p>Success!</p>';
	else
		print '<p>Clearing Failed!</p>';
	print '<pre>'; print_r(apc_cache_info()); print '</pre>';
}
// Add Purge APC in the favorite actions dropdown
add_filter('favorite_actions', 'clear_apc_link');
        
function clear_apc_link($actions) {
	$actions['tools.php?page=flush_php_apc'] = array('Purge APC', 'edit_posts');
	return $actions;
}
?>
{% endhighlight %}

<h4>Installation</h4>

* Upload <code>apc-cache-purge.php</code> to the `/wp-content/plugins/` directory
* Activate the plugin through the ‘Plugins’ menu in WordPress
* When needed, flush the cache by clicking on ‘Purge APC’ under the Tools section