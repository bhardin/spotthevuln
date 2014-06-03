---
layout: post
title: Monster Truck - SQL Injection
tags:
- Solution
- SQL Injection
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: SQL injection Vulnerability Code Example
  _headspace_description: SQL Injection Vulnerability Code Example
  _sexybookmarks_permaHash: af3c11d4c04aa6a1d8131530dfa1990b
  _sexybookmarks_shortUrl: http://bit.ly/cQ6Fah
---
## Details
<strong>__Affected Software:__ WordPress</strong>

<strong>__Fixed in Version:__  2.0.11</strong>

<strong>__Issue Type:__ SQL Injection</strong>

<strong>Original Code: </strong><a href="http://spotthevuln.com/2009/11/vulnerable-code-monster-truck/">Found Here</a>
## Description
This SQL Injection vulnerability was fixed in WordPress 2.0.11. Functions utilizing "string building" techniques to build SQL queries should always undergo detailed security review. Properly sanitizing dynamic, string built SQL queries is difficult. In this case, we see that $option_name is passed to the get_option() function, which in turn passes the un-sanitized, used controlled value to a string built SQL statement. The WordPress developers chose to address this issue by first escaping the $option_name variable and passing the escaped value to get_option(). The patched code also adds a comment indicating that the update_option() function expects $option_name to <strong>NOT </strong>be SQL-escaped (line 1 of the code diff). It may be fruitful to see if other calls to the update_option() function escape the $option_name value :)

Newer versions of WordPress (2.5 and &gt;) have transitioned away from calling string building SQL queries and provide a $wpdb-&gt;prepare() function to help prevent SQL Injection attacks. Information related to the database functions supported by WordPress can be found <a href="http://codex.wordpress.org/Function_Reference/wpdb_Class" target="_blank">here</a>.
## Developer's Solution
```diff

+// expects $option_name to NOT be SQL-escaped

function update_option($option_name, $newvalue) {

global $wpdb;

+       $safe_option_name = $wpdb-&gt;escape($option_name);

if ( is_string($newvalue) )
$newvalue = trim($newvalue);

// If the new and old values are the same, no need to update.
-       $oldvalue = get_option($option_name);
+       $oldvalue = get_option($safe_option_name);
if ( $newvalue == $oldvalue ) {
return false;
}

if ( false === $oldvalue ) {
add_option($option_name, $newvalue);
return true;
}

$_newvalue = $newvalue;
$newvalue = maybe_serialize($newvalue);

wp_cache_set($option_name, $newvalue, 'options');

$newvalue = $wpdb-&gt;escape($newvalue);
$option_name = $wpdb-&gt;escape($option_name);
$wpdb-&gt;query("UPDATE $wpdb-&gt;options SET option_value = '$newvalue' WHERE option_name = '$option_name'");
if ( $wpdb-&gt;rows_affected == 1 ) {
do_action("update_option_{$option_name}", array('old'=&gt;$oldvalue, 'new'=&gt;$_newvalue));
return true;
}
return false;
}

Where get_option is the following:

function get_option($setting) {
global $wpdb;

// Allow plugins to short-circuit options.
$pre = apply_filters( 'pre_option_' . $setting, false );
if ( false !== $pre )
return $pre;

// prevent non-existent options from triggering multiple queries
$notoptions = wp_cache_get('notoptions', 'options');
if ( isset($notoptions[$setting]) )
return false;

$alloptions = wp_load_alloptions();

if ( isset($alloptions[$setting]) ) {
$value = $alloptions[$setting];
} else {
$value = wp_cache_get($setting, 'options');

if ( false === $value ) {
if ( defined('WP_INSTALLING') )
$wpdb-&gt;hide_errors();
$row = $wpdb-&gt;get_row("SELECT option_value FROM $wpdb-&gt;options WHERE option_name = '$setting' LIMIT 1");
if ( defined('WP_INSTALLING') )
$wpdb-&gt;show_errors();

if( is_object( $row) ) { // Has to be get_row instead of get_var because of funkiness with 0, false, null values
$value = $row-&gt;option_value;
wp_cache_add($setting, $value, 'options');
} else { // option does not exist, so we must cache its non-existence
$notoptions[$setting] = true;
wp_cache_set('notoptions', $notoptions, 'options');
return false;
}
}
```
