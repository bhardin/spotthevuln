---
layout: post
title: Monster Truck
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
  _sexybookmarks_shortUrl: http://bit.ly/aWDW5N
  _sexybookmarks_permaHash: 9e4c02593cc993ce0a457d262644afe1
---
<blockquote>There are two ways to pass a hurdle: leaping over or plowing through... There needs to be a monster truck option.
- Jeph Jacques, Questionable Content #1356, 03-10-09</blockquote>
```php
<?
function update_option($option_name, $newvalue) {
global $wpdb;

if ( is_string($newvalue) )
$newvalue = trim($newvalue);

// If the new and old values are the same, no need to update.
$oldvalue = get_option($option_name);
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
