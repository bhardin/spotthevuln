---
layout: post
title: Butterflies - SQL Injection / XSS
tags:
- Cross-Site Scripting (XSS)
- PHP
- Solution
- SQL Injection
- Wordpress
status: publish
type: post
published: true
meta:
  _sexybookmarks_shortUrl: http://bit.ly/cqruBa
  _sexybookmarks_permaHash: 960a56a0349a78cdf3d44bd84fcd1a52
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: Butterflies - SQL Injection / XSS
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Butterflies - SQL Injection / XSS
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ WordPress Plugin Login LockDown

__Fixed in Version:__  1.5

__Issue Type:__ SQL Injection and XSS

Original Code: <a title="Butterflies" href="http://spotthevuln.com/2010/04/butterflies/" target="_blank">Found Here</a>
## Description
In a SpotTheVuln first, today's example actually contains two different vulnerabilities. The sample comes from a WordPress plugin called "Login Lockdown". The first vulnerability patched by the Login Lockdown developers is a SQL Injection vulnerability. In this particular example, we see that the "$release_id" variable is passed directly to a SQL statement without being sanitized. The Login Lockdown developers corrected this vulnerability by simply escaping the variable before including it in a dynamically built SQL statement.

The other vulnerabilities are XSS vulnerabilities. The beginning of the code sample contains several "isset" statements which eventually assign the values from various POST parameters to php variables. These variables are then used to build the HTML markup later in the PHP code. The Login Lockdown developers corrected this vulnerability by using the "esc_attr" before using the user controlled data in the HTML markup.
## Developer's Solution
```diff
<div id="_mcePaste">

&lt;?php

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

&lt;div class="updated"&gt;&lt;p&gt;&lt;strong&gt;&lt;?php _e("Settings Updated.", "loginlockdown");?&gt;&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;

&lt;?php

}

if (isset($_POST['release_lockdowns'])) {

if (isset($_POST['releaseme'])) {

$released = $_POST['releaseme'];

foreach ( $released as $release_id ) {

$results = $wpdb-&gt;query("UPDATE $table_name SET release_date = now() " .

- "WHERE lockdown_ID = $release_id");

+ "WHERE lockdown_ID = " . $wpdb-&gt;escape($release_id) . "");

}

}

update_option("loginlockdownAdminOptions", $loginlockdownAdminOptions);

?&gt;

&lt;div class="updated"&gt;&lt;p&gt;&lt;strong&gt;&lt;?php _e("Lockdowns Released.", "loginlockdown");?&gt;&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;

&lt;?php

}

$dalist = listLockedDown();

?&gt;

&lt;div&gt;

&lt;form method="post" action="&lt;?php echo $_SERVER["REQUEST_URI"]; ?&gt;"&gt;

&lt;h2&gt;&lt;?php _e('Login LockDown Options', 'loginlockdown') ?&gt;&lt;/h2&gt;

&lt;h3&gt;&lt;?php _e('Max Login Retries', 'loginlockdown') ?&gt;&lt;/h3&gt;

-&lt;input type="text" name="ll_max_login_retries" size="8" value="&lt;?php echo $loginlockdownAdminOptions['max_login_retries']; ?&gt;"&gt;

+&lt;input type="text" name="ll_max_login_retries" size="8" value="&lt;?php echo esc_attr($loginlockdownAdminOptions['max_login_retries']); ?&gt;"&gt;

&lt;h3&gt;&lt;?php _e('Retry Time Period Restriction (minutes)', 'loginlockdown') ?&gt;&lt;/h3&gt;

-&lt;input type="text" name="ll_retries_within" size="8" value="&lt;?php echo $loginlockdownAdminOptions['retries_within']; ?&gt;"&gt;

+&lt;input type="text" name="ll_retries_within" size="8" value="&lt;?php echo esc_attr($loginlockdownAdminOptions['retries_within']); ?&gt;"&gt;

&lt;h3&gt;&lt;?php _e('Lockout Length (minutes)', 'loginlockdown') ?&gt;&lt;/h3&gt;

-&lt;input type="text" name="ll_lockout_length" size="8" value="&lt;?php echo $loginlockdownAdminOptions['lockout_length']; ?&gt;"&gt;

+&lt;input type="text" name="ll_lockout_length" size="8" value="&lt;?php echo esc_attr($loginlockdownAdminOptions['lockout_length']); ?&gt;"&gt;

</div>
```
