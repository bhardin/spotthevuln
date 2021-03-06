---
layout: post
title: Country - Multiple Vulns
tags:
- All
- Code Snippet
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/cmdkA2
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: e712adbf81fb7f5e6cd072c89a208079
---
## Details
__Affected Software:__ Loginlockdown plugin for WordPress

__Fixed in Version:__  1.5

__Issue Type:__ CSRF, XSS, SQLi

Original Code: <a title="Country" href="http://spotthevuln.com/2010/10/country/" target="_blank">Found    Here</a>
## Description
There were a ton of issues addressed in this patch. Let's start from the top and work our way down. The first change we see is the addition of a nonce check (check_admin_referer()). The counterpart  to check_admin_referer() is a function named wp_nonce_field(), which can be found on lines 56-59 and 80-83. Lines 56-59 and 80-83 set the nonce/token in the HTML markup and check_admin_referer() later validates that the token is legitimate. This is done to prevent Cross Site Request Forgery (CSRF). Although no one symptom is a dead giveaway for CSRF, finding a FORM without a token/nonce as one of the INPUT fields is an indication that the request should be investigated. If the request introduces a state changing operations (changing of a user setting, modification of data, installation/removal of a new feature...etc) it will need to be protected by a CSRF nonce. This patch adds CSRF protections in a few different places.

The next issue is a classic SQL injection vulnerability. In line 38 we see that the developer has set $_POST['releaseme'] to a variable named $released. A few lines later, $released is used to build a dynamic SQL statement. What's interesting is $released is used at the end of a SQL statement and appears to be an integer, there could be other issues here if the escaping isn't done properly :)

Next up is a slew of XSS bugs. Lines 10-29 use various $_POST parameters to set a number of variables. These variables are then used to build HTML markup in lines 54-78. The developer addressed these XSS bugs by using the esc_attr() API before allowing PHP to echo the variable value. The developer also echoed the $_SERVER["REQUEST_URI"] value directly into HTML markup as well. This variable represents the URI given in order to access the page and is considered tainted/attacker controlled.
## Developer's Solution
[sourcecode language="diff"]

&lt;?php

...&lt;snip&gt;...

function print_loginlockdownAdminPage() {
 global $wpdb;
 $table_name = $wpdb-&gt;prefix . &quot;lockdowns&quot;;
 $loginlockdownAdminOptions = get_loginlockdownOptions();

 if (isset($_POST['update_loginlockdownSettings'])) {
+        //wp_nonce check
+        check_admin_referer('login-lockdown_update-options');
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
 update_option(&quot;loginlockdownAdminOptions&quot;, $loginlockdownAdminOptions);
 ?&gt;

&lt;div&gt;&lt;p&gt;&lt;strong&gt;&lt;?php _e(&quot;Settings Updated.&quot;, &quot;loginlockdown&quot;);?&gt;&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;
 &lt;?php
 }
 if (isset($_POST['release_lockdowns'])) {
+        //wp_nonce check
+        check_admin_referer('login-lockdown_release-lockdowns');
 if (isset($_POST['releaseme'])) {
 $released = $_POST['releaseme'];
 foreach ( $released as $release_id ) {
 $results = $wpdb-&gt;query(&quot;UPDATE $table_name SET release_date = now() &quot; .
-                            &quot;WHERE lockdown_ID = $release_id&quot;);
+                            &quot;WHERE lockdown_ID = &quot; . $wpdb-&gt;escape($release_id) . &quot;&quot;);
 }
 }
 update_option(&quot;loginlockdownAdminOptions&quot;, $loginlockdownAdminOptions);
 ?&gt;

&lt;div&gt;&lt;p&gt;&lt;strong&gt;&lt;?php _e(&quot;Lockdowns Released.&quot;, &quot;loginlockdown&quot;);?&gt;&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;
 &lt;?php
 }
 $dalist = listLockedDown();
?&gt;
&lt;div&gt;
-&lt;form method=&quot;post&quot; action=&quot;&lt;?php echo $_SERVER[&quot;REQUEST_URI&quot;]; ?&gt;&quot;&gt;
+&lt;form method=&quot;post&quot; action=&quot;&lt;?php echo esc_attr($_SERVER[&quot;REQUEST_URI&quot;]); ?&gt;&quot;&gt;
+&lt;?php
+if ( function_exists('wp_nonce_field') )
+    wp_nonce_field('login-lockdown_update-options');
+?&gt;
&lt;h2&gt;&lt;?php _e('Login LockDown Options', 'loginlockdown') ?&gt;&lt;/h2&gt;
&lt;h3&gt;&lt;?php _e('Max Login Retries', 'loginlockdown') ?&gt;&lt;/h3&gt;
-&lt;input type=&quot;text&quot; name=&quot;ll_max_login_retries&quot; size=&quot;8&quot; value=&quot;&lt;?php echo $loginlockdownAdminOptions['max_login_retries']; ?&gt;&quot;&gt;
+&lt;input type=&quot;text&quot; name=&quot;ll_max_login_retries&quot; size=&quot;8&quot; value=&quot;&lt;?php echo esc_attr($loginlockdownAdminOptions['max_login_retries']); ?&gt;&quot;&gt;
&lt;h3&gt;&lt;?php _e('Retry Time Period Restriction (minutes)', 'loginlockdown') ?&gt;&lt;/h3&gt;
-&lt;input type=&quot;text&quot; name=&quot;ll_retries_within&quot; size=&quot;8&quot; value=&quot;&lt;?php echo $loginlockdownAdminOptions['retries_within']; ?&gt;&quot;&gt;
+&lt;input type=&quot;text&quot; name=&quot;ll_retries_within&quot; size=&quot;8&quot; value=&quot;&lt;?php echo esc_attr($loginlockdownAdminOptions['retries_within']); ?&gt;&quot;
&lt;h3&gt;&lt;?php _e('Lockout Length (minutes)', 'loginlockdown') ?&gt;&lt;/h3&gt;
-&lt;input type=&quot;text&quot; name=&quot;ll_lockout_length&quot; size=&quot;8&quot; value=&quot;&lt;?php echo $loginlockdownAdminOptions['lockout_length']; ?&gt;&quot;&gt;
+&lt;input type=&quot;text&quot; name=&quot;ll_lockout_length&quot; size=&quot;8&quot; value=&quot;&lt;?php echo esc_attr($loginlockdownAdminOptions['lockout_length']); ?&gt;&quot;
&lt;h3&gt;&lt;?php _e('Lockout Invalid Usernames?', 'loginlockdown') ?&gt;&lt;/h3&gt;
&lt;input type=&quot;radio&quot; name=&quot;ll_lockout_invalid_usernames&quot; value=&quot;yes&quot; &lt;?php if( $loginlockdownAdminOptions['lockout_invalid_usernames'] == &quot;yes&quot; ) echo &quot;checked&quot;; ?&gt;&gt;&amp;nbsp;Yes&amp;nbsp;&amp;nbsp;&amp;nbsp;&lt;input type=&quot;radio&quot; name=&quot;ll_lockout_invalid_usernames&quot; value=&quot;no&quot; &lt;?php if( $loginlockdownAdminOptions['lockout_invalid_usernames'] == &quot;no&quot; ) echo &quot;checked&quot;; ?&gt;&gt;&amp;nbsp;No
&lt;h3&gt;&lt;?php _e('Mask Login Errors?', 'loginlockdown') ?&gt;&lt;/h3&gt;
&lt;input type=&quot;radio&quot; name=&quot;ll_mask_login_errors&quot; value=&quot;yes&quot; &lt;?php if( $loginlockdownAdminOptions['mask_login_errors'] == &quot;yes&quot; ) echo &quot;checked&quot;; ?&gt;&gt;&amp;nbsp;Yes&amp;nbsp;&amp;nbsp;&amp;nbsp;&lt;input type=&quot;radio&quot; name=&quot;ll_mask_login_errors&quot; value=&quot;no&quot; &lt;?php if( $loginlockdownAdminOptions['mask_login_errors'] == &quot;no&quot; ) echo &quot;checked&quot;; ?&gt;&gt;&amp;nbsp;No
&lt;div&gt;
&lt;input type=&quot;submit&quot; name=&quot;update_loginlockdownSettings&quot; value=&quot;&lt;?php _e('Update Settings', 'loginlockdown') ?&gt;&quot; /&gt;&lt;/div&gt;
&lt;/form&gt;
&lt;br /&gt;
-&lt;form method=&quot;post&quot; action=&quot;&lt;?php echo $_SERVER[&quot;REQUEST_URI&quot;]; ?&gt;&quot;&gt;
+&lt;form method=&quot;post&quot; action=&quot;&lt;?php echo esc_attr($_SERVER[&quot;REQUEST_URI&quot;]); ?&gt;&quot;&gt;
+&lt;?php
+if ( function_exists('wp_nonce_field') )
+    wp_nonce_field('login-lockdown_release-lockdowns');
+?&gt;
&lt;h3&gt;&lt;?php _e('Currently Locked Out', 'loginlockdown') ?&gt;&lt;/h3&gt;

[/sourcecode]
