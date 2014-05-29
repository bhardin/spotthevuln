---
layout: post
title: Expensive Carpet
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Vulnerable Source Code
  _headspace_description: Can you identify the security vulnerability in this code
    snippet?
  _sexybookmarks_permaHash: 474480f7385fee2e3f4c180f3682a543
  _sexybookmarks_shortUrl: http://bit.ly/dfkiBn
---
<blockquote><p>What if everything is an illusion and nothing exists? In that case, I definitely overpaid for my carpet.<br />
- Woody Allen</p></blockquote>
<p>[ccnLe_php]</p>
<p>if ( !function_exists('get_userdatabylogin') ) :<br />
function get_userdatabylogin($user_login) {<br />
global $wpdb;<br />
$user_login = sanitize_user( $user_login );</p>
<p>if ( empty( $user_login ) )<br />
return false;</p>
<p>$userdata = wp_cache_get($user_login, 'userlogins');<br />
if ( $userdata )<br />
return $userdata;</p>
<p>if ( !$user = $wpdb-&gt;get_row("SELECT * FROM $wpdb-&gt;users WHERE user_login = '$user_login'") )<br />
return false;</p>
<p>$wpdb-&gt;hide_errors();<br />
$metavalues = $wpdb-&gt;get_results("SELECT meta_key, meta_value FROM $wpdb-&gt;usermeta WHERE user_id = '$user-&gt;ID'");<br />
$wpdb-&gt;show_errors();</p>
<p>if ($metavalues) {<br />
foreach ( $metavalues as $meta ) {<br />
$value = maybe_unserialize($meta-&gt;meta_value);<br />
$user-&gt;{$meta-&gt;meta_key} = $value;</p>
<p>// We need to set user_level from meta, not row<br />
if ( $wpdb-&gt;prefix . 'user_level' == $meta-&gt;meta_key )<br />
$user-&gt;user_level = $meta-&gt;meta_value;<br />
}<br />
}</p>
<p>// For backwards compat.<br />
if ( isset($user-&gt;first_name) )<br />
$user-&gt;user_firstname = $user-&gt;first_name;<br />
if ( isset($user-&gt;last_name) )<br />
$user-&gt;user_lastname = $user-&gt;last_name;<br />
if ( isset($user-&gt;description) )<br />
$user-&gt;user_description = $user-&gt;description;</p>
<p>wp_cache_add($user-&gt;ID, $user, 'users');<br />
wp_cache_add($user-&gt;user_login, $user, 'userlogins');</p>
<p>return $user;</p>
<p>}</p>
<p>Where Sanitize_user is<br />
function sanitize_user( $username, $strict = false ) {<br />
$raw_username = $username;<br />
$username = strip_tags($username);<br />
// Kill octets<br />
$username = preg_replace('|%([a-fA-F0-9][a-fA-F0-9])|', '', $username);<br />
$username = preg_replace('/&amp;.+?;/', '', $username); // Kill entities</p>
<p>// If strict, reduce to ASCII for max portability.<br />
if ( $strict )<br />
$username = preg_replace('|[^a-z0-9 _.\-@]|i', '', $username);</p>
<p>return apply_filters('sanitize_user', $username, $raw_username, $strict);<br />
}</p>
<p>```</p>
