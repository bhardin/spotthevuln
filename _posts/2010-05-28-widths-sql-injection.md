---
layout: post
title: Widths - SQL Injection
tags:
- All Vulnerabilities
- bug fixes
- bugs
- Code Snippet
- Cross-Site Scripting (XSS)
- CSRF
- dynamic SQL
- framework level
- html markup
- inclusion
- logic
- nonce
- PHP
- sanitization
- security flaws
- security software
- Solution
- SQL Injection
- SQLi
- token validation
- type sql
- validation code
- vulnerabilities
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/b9LTd0
  _sexybookmarks_permaHash: e0c8845107f87359b4b1d7af053b11db
---
## Details
__Affected Software:__ Login LockDown for Wordpress

__Fixed in Version:__  1.5

__Issue Type:__ SQL Injection

Original Code: <a title="Widths" href="http://spotthevuln.com/2010/05/widths/" target="_blank">Found Here</a>
## Description
This week's code sample comes from the "Login Lockdown" plug-in for WordPress. It's always interesting when "security" software ends up having serious security flaws....

This patch contained several bug fixes. The first bug fix we see in the patch is the inclusion of nonce checking to prevent CSRF. It's difficult to detect CSRF vulnerabilities by looking at individual function logic and it's ok if the reader missed these bugs. CSRF token validation should be done at the framework level and including CSRF nonce validation in the logic of every function can quickly become unwieldy. If an application wide CSRF solution cannot be implemented at the framework level, then auditing for CSRF must be done at a function by function level. Personally, I prefer to check for CSRF vulnerabilities by identifying any function that performs a Create, Update, or Delete operation, mapping those functions back to the HTML markup and checking the markup to see if a nonce is passed as part of the POST or GET request. This is of course is done after an extensive audit of the nonce validation code.

The bugs that should have been spotted by the spotthevuln reader are the SQL injection and the XSS vulnerabilities in the code. The SQL Injection is pretty straight forward. The "releaseme" POST parameter is taken and is eventually passed directly to a dynamically built SQL statement without any sanitization. The developers fixed the vulnerability by utilizing the WordPress escape logic.

Finally, the last line of the code snippet actually contained an XSS vulnerability, echoing a $_SERVER variable without any sanitization.
## Developer's Solution
```diff

function print_loginlockdownAdminPage() {
        global $wpdb;
        $table_name = $wpdb-&gt;prefix . "lockdowns";
        $loginlockdownAdminOptions = get_loginlockdownOptions();

        if (isset($_POST['update_loginlockdownSettings'])) {

+  //wp_nonce check
+  check_admin_referer('login-lockdown_update-options');

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

+  //wp_nonce check
+  check_admin_referer('login-lockdown_release-lockdowns');

  if (isset($_POST['releaseme'])) {
                        $released = $_POST['releaseme'];
                        foreach ( $released as $release_id ) {
                                $results = $wpdb-&gt;query("UPDATE $table_name SET release_date = now() " .
-                                                       "WHERE lockdown_ID = $release_id");
+                                                      "WHERE lockdown_ID = " . $wpdb-&gt;escape($release_id) . "");
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
-&lt;form method="post" action="&lt;?php echo $_SERVER["REQUEST_URI"]; ?&gt;"&gt;
+&lt;form method="post" action="&lt;?php echo esc_attr($_SERVER["REQUEST_URI"]); ?&gt;"&gt;

```
