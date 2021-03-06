---
layout: post
title: Expensive Carpet - SQL Injection
tags:
- Solution
- SQL Injection
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '2'
  _headspace_page_title: SQL injection Vulnerability Code Example
  _headspace_description: SQL Injection Vulnerability Code Example
  _sexybookmarks_permaHash: c7bfa02d0bde4958186d667ddaf7a658
  _sexybookmarks_shortUrl: http://bit.ly/9DlsIz
---
## Details
<strong>__Affected Software:__ WordPress</strong>

<strong>__Fixed in Version:__  2.0.11</strong>

<strong>__Issue Type:__ SQL Injection</strong>

<strong>Original Code: </strong><a href="http://spotthevuln.com/2009/12/expensive-carpet/">Found Here</a>
## Description
This was a SQL injection bug that affected WordPress versions up until 2.0.11. Looking at the vulnerable code, we see the classic string building of a SQL statement. In this case, the $user_login variable is fully controlled by the attacker, allowing the attacker to control the string built SQL query. Surprisingly, a sanitize_user() function was called in order to sanitize the attacker controlled $user_login value. Looking at sanitize_user() function we see two routines which sanitize the user name, one routine to "kill octets" and one routine to "kill entities", unfortunately the sanitize_user() function failed to sanitize the $user_login variable against SQL injection. Another interesting fact about sanitize_user(), is it actually contained logic to sanitize $user_login to only ASCII characters, however this was disabled by default, leaving WordPress installations vulnerable to blind SQL Injection attacks.

After discovering fully weapoinzed exploit code for this issue, the WordPress developers addressed this issue by escaping the $user_login variable before passing it to the string built SQL query.
## Developer's Solution
```diff

if ( !function_exists('get_userdatabylogin') ) :
function get_userdatabylogin($user_login) {
global $wpdb;
$user_login = sanitize_user( $user_login );

if ( empty( $user_login ) )
return false;

$userdata = wp_cache_get($user_login, 'userlogins');
if ( $userdata )
return $userdata;
+       $user_login = $wpdb-&gt;escape($user_login);

if ( !$user = $wpdb-&gt;get_row("SELECT * FROM $wpdb-&gt;users WHERE user_login = '$user_login'") )
return false;

$wpdb-&gt;hide_errors();
$metavalues = $wpdb-&gt;get_results("SELECT meta_key, meta_value FROM $wpdb-&gt;usermeta WHERE user_id = '$user-&gt;ID'");
$wpdb-&gt;show_errors();

if ($metavalues) {
foreach ( $metavalues as $meta ) {
$value = maybe_unserialize($meta-&gt;meta_value);
$user-&gt;{$meta-&gt;meta_key} = $value;

// We need to set user_level from meta, not row
if ( $wpdb-&gt;prefix . 'user_level' == $meta-&gt;meta_key )
$user-&gt;user_level = $meta-&gt;meta_value;
}
}

// For backwards compat.
if ( isset($user-&gt;first_name) )
$user-&gt;user_firstname = $user-&gt;first_name;
if ( isset($user-&gt;last_name) )
$user-&gt;user_lastname = $user-&gt;last_name;
if ( isset($user-&gt;description) )
$user-&gt;user_description = $user-&gt;description;

wp_cache_add($user-&gt;ID, $user, 'users');
wp_cache_add($user-&gt;user_login, $user, 'userlogins');

return $user;

}

Where Sanitize_user is
function sanitize_user( $username, $strict = false ) {
$raw_username = $username;
$username = strip_tags($username);
// Kill octets
$username = preg_replace('|%([a-fA-F0-9][a-fA-F0-9])|', '', $username);
$username = preg_replace('/&amp;.+?;/', '', $username); // Kill entities

// If strict, reduce to ASCII for max portability.
if ( $strict )
$username = preg_replace('|[^a-z0-9 _.\-@]|i', '', $username);

return apply_filters('sanitize_user', $username, $raw_username, $strict);
}

```
