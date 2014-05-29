---
layout: post
title: Butterflies
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: c0180d133300105010535833b9791d7b
  _sexybookmarks_shortUrl: http://bit.ly/bBX7Uq
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '2'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
<blockquote><strong>It's so bizarre, I'm not scared of snakes or spiders. But I'm scared of butterflies. There is something eerie about them. Something weird!</strong>
- Nicole Kidman</blockquote>
```php
<?php

function print_loginlockdownAdminPage() {

global $wpdb;

$table_name = $wpdb-&gt;prefix . "lockdowns";

$loginlockdownAdminOptions = get_loginlockdownOptions();

if (isset($_POST['update_loginlockdownSettings'])) {

if (isset($_POST['ll_max_login_retries'])) {

$loginlockdownAdminOptions['max_login_retries'] = $_POST['ll_max_login_retries'];

}

if (isset($_POST['ll_retries_within'])) {

$loginlockdownAdminOptions['retries_within'] = $_POST['ll_retries_within'];

}

if (isset($_POST['ll_lockout_length'])) {

$loginlockdownAdminOptions['lockout_length'] = $_POST['ll_lockout_length'];

}

if (isset($_POST['ll_lockout_invalid_usernames'])) {

$loginlockdownAdminOptions['lockout_invalid_usernames'] = $_POST['ll_lockout_invalid_usernames'];

}

if (isset($_POST['ll_mask_login_errors'])) {

$loginlockdownAdminOptions['mask_login_errors'] = $_POST['ll_mask_login_errors'];

}

update_option("loginlockdownAdminOptions", $loginlockdownAdminOptions);

?&gt;

&lt;div&gt;&lt;p&gt;&lt;strong&gt;&lt;?php _e("Settings Updated.", "loginlockdown");?&gt;&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;

&lt;?php

}

if (isset($_POST['release_lockdowns'])) {

if (isset($_POST['releaseme'])) {

$released = $_POST['releaseme'];

foreach ( $released as $release_id ) {

$results = $wpdb-&gt;query("UPDATE $table_name SET release_date = now() " .

"WHERE lockdown_ID = $release_id");

}

}

update_option("loginlockdownAdminOptions", $loginlockdownAdminOptions);

?&gt;

&lt;div&gt;&lt;p&gt;&lt;strong&gt;&lt;?php _e("Lockdowns Released.", "loginlockdown");?&gt;&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;

&lt;?php

}

$dalist = listLockedDown();

?&gt;

&lt;div&gt;

&lt;form method="post" action="&lt;?php echo $_SERVER["REQUEST_URI"]; ?&gt;"&gt;

&lt;h2&gt;&lt;?php _e('Login LockDown Options', 'loginlockdown') ?&gt;&lt;/h2&gt;

&lt;h3&gt;&lt;?php _e('Max Login Retries', 'loginlockdown') ?&gt;&lt;/h3&gt;

&lt;input type="text" name="ll_max_login_retries" size="8" value="&lt;?php echo $loginlockdownAdminOptions['max_login_retries']; ?&gt;"&gt;

&lt;h3&gt;&lt;?php _e('Retry Time Period Restriction (minutes)', 'loginlockdown') ?&gt;&lt;/h3&gt;

&lt;input type="text" name="ll_retries_within" size="8" value="&lt;?php echo $loginlockdownAdminOptions['retries_within']; ?&gt;"&gt;

&lt;h3&gt;&lt;?php _e('Lockout Length (minutes)', 'loginlockdown') ?&gt;&lt;/h3&gt;

&lt;input type="text" name="ll_lockout_length" size="8" value="&lt;?php echo $loginlockdownAdminOptions['lockout_length']; ?&gt;"&gt;

```
