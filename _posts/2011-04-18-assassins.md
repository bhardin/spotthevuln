---
layout: post
title: Assassins
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  aktt_notify_twitter: 'yes'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
---
<blockquote><strong>I do not like assassins, or men of low character.
Gene Hackman</strong></blockquote>
[sourcecode language="php"]
&lt;?php
...snip...
function wp_original_referer_field() {
	echo '&lt;input type=&quot;hidden&quot; name=&quot;_wp_original_http_referer&quot; value=&quot;' . attribute_escape(stripslashes($_SERVER['REQUEST_URI'])) . '&quot; /&gt;';
}

function wp_get_referer() {
	foreach ( array($_REQUEST['_wp_http_referer'], $_SERVER['HTTP_REFERER']) as $ref )
		if ( !empty($ref) )
			return $ref;
	return false;
}

function wp_get_original_referer() {
	if ( !empty($_REQUEST['_wp_original_http_referer']) )
		return $_REQUEST['_wp_original_http_referer'];
	return false;
}

function wp_mkdir_p($target) {
	// from php.net/mkdir user contributed notes
	if (file_exists($target)) {
		if (! @ is_dir($target))
			return false;
		else
			return true;
	}

	// Attempting to create the directory may clutter up our display.
	if (@ mkdir($target)) {
		$stat = @ stat(dirname($target));
		$dir_perms = $stat['mode'] &amp; 0007777;  // Get the permission bits.
		@ chmod($target, $dir_perms);
		return true;
	} else {
		if ( is_dir(dirname($target)) )
			return false;
	}

	// If the above failed, attempt to create the parent node, then try again.
	if (wp_mkdir_p(dirname($target)))
		return wp_mkdir_p($target);

	return false;
}

...snip...

function wp_nonce_ays($action) {
	global $pagenow, $menu, $submenu, $parent_file, $submenu_file;

	$adminurl = get_option('siteurl') . '/wp-admin';
	if ( wp_get_referer() )
		$adminurl = wp_get_referer();

	$title = __('WordPress Confirmation');
	// Remove extra layer of slashes.
	$_POST   = stripslashes_deep($_POST  );
	if ( $_POST ) {
		$q = http_build_query($_POST);
		$q = explode( ini_get('arg_separator.output'), $q);
		$html .= &quot;\t&lt;form method='post' action='$pagenow'&gt;\n&quot;;
		foreach ( (array) $q as $a ) {
			$v = substr(strstr($a, '='), 1);
			$k = substr($a, 0, -(strlen($v)+1));
			$html .= &quot;\t\t&lt;input type='hidden' name='&quot; . attribute_escape(urldecode($k)) . &quot;' value='&quot; . attribute_escape(urldecode($v)) . &quot;' /&gt;\n&quot;;
		}
		$html .= &quot;\t\t&lt;input type='hidden' name='_wpnonce' value='&quot; . wp_create_nonce($action) . &quot;' /&gt;\n&quot;;
		$html .= &quot;\t\t&lt;div id='message' class='confirm fade'&gt;\n\t\t&lt;p&gt;&quot; . wp_specialchars(wp_explain_nonce($action)) . &quot;&lt;/p&gt;\n\t\t&lt;p&gt;&lt;a href='$adminurl'&gt;&quot; . __('No') . &quot;&lt;/a&gt; &lt;input type='submit' value='&quot; . __('Yes') . &quot;' /&gt;&lt;/p&gt;\n\t\t&lt;/div&gt;\n\t&lt;/form&gt;\n&quot;;
	} else {
		$html .= &quot;\t&lt;div id='message' class='confirm fade'&gt;\n\t&lt;p&gt;&quot; . wp_specialchars(wp_explain_nonce($action)) . &quot;&lt;/p&gt;\n\t&lt;p&gt;&lt;a href='$adminurl'&gt;&quot; . __('No') . &quot;&lt;/a&gt; &lt;a href='&quot; . add_query_arg( '_wpnonce', wp_create_nonce($action), $_SERVER['REQUEST_URI'] ) . &quot;'&gt;&quot; . __('Yes') . &quot;&lt;/a&gt;&lt;/p&gt;\n\t&lt;/div&gt;\n&quot;;
	}
	$html .= &quot;&lt;/body&gt;\n&lt;/html&gt;&quot;;
	wp_die($html, $title);
}

function wp_die($message, $title = '') {
	global $wp_locale;

	header('Content-Type: text/html; charset=utf-8');

	if ( empty($title) )
		$title = __('WordPress &amp;rsaquo; Error');

	if ( strstr($_SERVER['PHP_SELF'], 'wp-admin') )
		$admin_dir = '';
	else
		$admin_dir = 'wp-admin/';

?&gt;
&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Transitional//EN&quot; &quot;http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;&gt;
&lt;html xmlns=&quot;http://www.w3.org/1999/xhtml&quot; &lt;?php if ( function_exists('language_attributes') ) language_attributes(); ?&gt;&gt;
&lt;head&gt;
	&lt;title&gt;&lt;?php echo $title ?&gt;&lt;/title&gt;
	&lt;meta http-equiv=&quot;Content-Type&quot; content=&quot;text/html; charset=utf-8&quot; /&gt;
	&lt;link rel=&quot;stylesheet&quot; href=&quot;&lt;?php echo $admin_dir; ?&gt;install.css&quot; type=&quot;text/css&quot; /&gt;
&lt;?php if ( ('rtl' == $wp_locale-&gt;text_direction) ) : ?&gt;
	&lt;link rel=&quot;stylesheet&quot; href=&quot;&lt;?php echo $admin_dir; ?&gt;install-rtl.css&quot; type=&quot;text/css&quot; /&gt;
&lt;?php endif; ?&gt;
&lt;/head&gt;
&lt;body&gt;
	&lt;h1 id=&quot;logo&quot;&gt;&lt;img alt=&quot;WordPress&quot; src=&quot;&lt;?php echo $admin_dir; ?&gt;images/wordpress-logo.png&quot; /&gt;&lt;/h1&gt;
	&lt;p&gt;&lt;?php echo $message; ?&gt;&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;

...snip...
[/sourcecode] 
