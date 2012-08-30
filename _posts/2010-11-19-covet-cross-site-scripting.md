---
layout: post
title: Covet - Cross Site Scripting
tags:
- All
- All Vulnerabilities
- attacker
- cross site scripting
- Cross-Site Scripting (XSS)
- Encode
- PHP
- querystring
- request uri
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  _oembed_6e7899438737758429f668ffc4f8bf93: ! '{{unknown}}'
  aktt_tweeted: '1'
  _oembed_54e49ebe6fc45058afde7de1c0982440: ! '{{unknown}}'
  _oembed_297d67a1875888a6cd685484344937de: ! '{{unknown}}'
  _sexybookmarks_shortUrl: http://bit.ly/c1fjQ9
  _sexybookmarks_permaHash: 09120d7377348673c7f83a71550e4a26
  _oembed_73799e56daf662700f44b5a7cb757b0b: ! '{{unknown}}'
  _oembed_86a20ff086320a1e5e54d1431c4c0075: ! '{{unknown}}'
  _oembed_fe268eaabf4a8919db6d22828471e10f: ! '{{unknown}}'
---
## Details
__Affected Software:__ Subscribe to Comments Plugin

__Fixed in Version:__  2.1

__Issue Type:__ Cross Site Scripting (XSS)

Original Code: <a title="Covet" href="http://spotthevuln.com/2010/11/covet/" target="_blank">Found    Here</a>
## Description
A familiar symptom here with the same ole result.  In the vulnerable code we see a call to $_SERVER['REQUEST_URI'].  For some reason, many developers assume REQUEST_URI cannot be tainted and used for XSS.  REQUEST_URI will not only include the path to current php file, it will also include any querystring parameters in the URI as well.  Here's a few examples of what REQUEST_URI will return:
<blockquote>http://spotthevuln.com/blah/
results in --&gt; /

http://spotthevuln.com/blah//index.php
results in --&gt; /blah/index.php

http://spotthevuln.com/blah/index.php?qs=value
results in --&gt; /blah/index.php?qs=value

http://spotthevuln.com/blah/index.php/qs1/qs2
results in --&gt; /blah/index.php/qs1/qs2</blockquote>
As you can see an attacker can easily taint the REQUEST_URI value, using it in XSS attacks.  The developers addressed this vulnerability by encoding calls to REQUEST_URI.
## Developers Solution
[sourcecode language="diff"]
&lt;?php

/* ================= */
/* Display Functions */
/* ================= */

/* -------------------------
What follows are the functions that display things in your comments form.
Feel free to customize them to your needs
------------------------- */

/* -------------------------
This is the code that is inserted into your comment form.  You may modify it, if you wish.
------------------------- */
function show_subscription_checkbox ($id='0') {
	global $sg_subscribe;
	sg_subscribe_start();

	if ( $sg_subscribe-&gt;checkbox_shown ) return $id;
	if ( !$email = $sg_subscribe-&gt;current_viewer_subscription_status() ) : ?&gt;

&lt;?php /* ------------------------------------------------------------------- */ ?&gt;
&lt;?php /* This is the text that is displayed for users who are NOT subscribed */ ?&gt;
&lt;?php /* ------------------------------------------------------------------- */ ?&gt;

	&lt;p &lt;?php if ($sg_subscribe-&gt;clear_both) echo 'style=&quot;clear: both;&quot; '; ?&gt;class=&quot;subscribe-to-comments&quot;&gt;
        &lt;input type=&quot;checkbox&quot; name=&quot;subscribe&quot; id=&quot;subscribe&quot; value=&quot;subscribe&quot; style=&quot;width: auto;&quot; &lt;?php if ($sg_subscribe-&gt;default_subscribed) echo 'checked=&quot;checked&quot; '; ?&gt;/&gt;
        &lt;label for=&quot;subscribe&quot;&gt;&lt;?php echo $sg_subscribe-&gt;not_subscribed_text; ?&gt;&lt;/label&gt;
	&lt;/p&gt;

&lt;?php /* ------------------------------------------------------------------- */ ?&gt;

&lt;?php elseif ( $email == 'admin' &amp;&amp; current_user_can('manage_options') ) : ?&gt;

&lt;?php /* ------------------------------------------------------------- */ ?&gt;
&lt;?php /* This is the text that is displayed for the author of the post */ ?&gt;
&lt;?php /* ------------------------------------------------------------- */ ?&gt;

	&lt;p &lt;?php if ($sg_subscribe-&gt;clear_both) echo 'style=&quot;clear: both;&quot; '; ?&gt;class=&quot;subscribe-to-comments&quot;&gt;
	&lt;?php echo str_replace('[manager_link]', $sg_subscribe-&gt;manage_link($email, true, false), $sg_subscribe-&gt;author_text); ?&gt;
	&lt;/p&gt;

&lt;?php else : ?&gt;

&lt;?php /* --------------------------------------------------------------- */ ?&gt;
&lt;?php /* This is the text that is displayed for users who ARE subscribed */ ?&gt;
&lt;?php /* --------------------------------------------------------------- */ ?&gt;

	&lt;p &lt;?php if ($sg_subscribe-&gt;clear_both) echo 'style=&quot;clear: both;&quot; '; ?&gt;class=&quot;subscribe-to-comments&quot;&gt;
	&lt;?php echo str_replace('[manager_link]', $sg_subscribe-&gt;manage_link($email, true, false), $sg_subscribe-&gt;subscribed_text); ?&gt;
	&lt;/p&gt;

&lt;?php /* --------------------------------------------------------------- */ ?&gt;

&lt;?php endif;

$sg_subscribe-&gt;checkbox_shown = true;
return $id;
}

/* -------------------------------------------------------------------- */
/* This function outputs a &quot;subscribe without commenting&quot; form.         */
/* Place this somewhere within &quot;the loop&quot;, but NOT within another form  */
/* This is NOT inserted automaticallly... you must place it yourself    */
/* -------------------------------------------------------------------- */
function show_manual_subscription_form () {
	global $id, $sg_subscribe, $user_email;
	sg_subscribe_start();
	$sg_subscribe-&gt;show_errors('solo_subscribe', '&lt;div class=&quot;solo-subscribe-errors&quot;&gt;', '&lt;/div&gt;', __('&lt;strong&gt;Error: &lt;/strong&gt;', 'subscribe-to-comments'), '&lt;br /&gt;');

if ( !$sg_subscribe-&gt;current_viewer_subscription_status() ) :
	get_currentuserinfo(); ?&gt;

&lt;?php /* ------------------------------------------------------------------- */ ?&gt;
&lt;?php /* This is the text that is displayed for users who are NOT subscribed */ ?&gt;
&lt;?php /* ------------------------------------------------------------------- */ ?&gt;

-	&lt;form action=&quot;http://&lt;?php echo $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'] ?&gt;&quot; method=&quot;post&quot;&gt;
+	&lt;form action=&quot;http://&lt;?php echo $_SERVER['HTTP_HOST'] . wp_specialchars($_SERVER['REQUEST_URI']); ?&gt;&quot; method=&quot;post&quot;&gt;
	&lt;input type=&quot;hidden&quot; name=&quot;solo-comment-subscribe&quot; value=&quot;solo-comment-subscribe&quot; /&gt;
	&lt;input type=&quot;hidden&quot; name=&quot;postid&quot; value=&quot;&lt;?php echo $id; ?&gt;&quot; /&gt;
-	&lt;input type=&quot;hidden&quot; name=&quot;ref&quot; value=&quot;&lt;?php echo 'http://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI']; ?&gt;&quot; /&gt;
+	&lt;input type=&quot;hidden&quot; name=&quot;ref&quot; value=&quot;&lt;?php echo urlencode('http://' . $_SERVER['HTTP_HOST'] . wp_specialchars($_SERVER['REQUEST_URI'])); ?&gt;&quot; /&gt;

	&lt;p class=&quot;solo-subscribe-to-comments&quot;&gt;
	&lt;?php _e('Subscribe without commenting', 'subscribe-to-comments'); ?&gt;
	&lt;br /&gt;
	&lt;label for=&quot;solo-subscribe-email&quot;&gt;&lt;?php _e('E-Mail:', 'subscribe-to-comments'); ?&gt;
	&lt;input type=&quot;text&quot; name=&quot;email&quot; id=&quot;solo-subscribe-email&quot; size=&quot;22&quot; value=&quot;&lt;?php echo $user_email; ?&gt;&quot; /&gt;&lt;/label&gt;
	&lt;input type=&quot;submit&quot; name=&quot;submit&quot; value=&quot;&lt;?php _e('Subscribe', 'subscribe-to-comments'); ?&gt;&quot; /&gt;
	&lt;/p&gt;
	&lt;/form&gt;

&lt;?php /* ------------------------------------------------------------------- */ ?&gt;
[/sourcecode] 
