---
layout: post
title: Burnout - SQL Injection
tags:
- addslashes
- bad idea
- double quotes
- dynamic SQL
- Encode
- insert sql
- PHP
- plugins
- query sql
- Solution
- SQL Injection
- Wordpress
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: c2a96b6cb1c8ccfa4cebd8b950824485
  _sexybookmarks_shortUrl: http://bit.ly/aKg7UQ
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
## Details
__Affected Software:__ TimesToCome Stop Bot Registration (Wordpress Plugin)

__Fixed in Version:__  1.9

__Issue Type:__ SQL Injection

Original Code: <a title="Burnout" href="http://spotthevuln.com/2010/08/burnout/" target="_blank">Found    Here</a>
## Description
Straight up SQL Injection vulnerability. The following attacker controlled values are taken via the following lines:
<blockquote>$request_time = $_SERVER['REQUEST_TIME'];

$http_accept = $_SERVER['HTTP_ACCEPT'];

$http_user_agent = $_SERVER['HTTP_USER_AGENT'];

$http_remote_addr = $_SERVER['REMOTE_ADDR'];</blockquote>
Then, a few lines down the attacker controlled values are used as part of a dynamic INSERT SQL statement.
<blockquote>$sql = "INSERT INTO " . $registration_log_table_name . " ( ip, email, problem, accept, agent, day )

VALUES ( '$http_remote_addr', '$user', '$error', '$http_accept', '$http_user_agent', NOW() )";

$result = $wpdb-&gt;query( $sql );</blockquote>
Understanding that taking attacker controlled data and using it as part of a dynamic SQL statement is a bad idea, the plug-in developers checked in a patch to sanitize the data before using in the SQL statement. The path is shown below:
<blockquote>$http_accept = htmlentities($http_accept);

$http_user_agent = htmlentities($http_user_agent);

$http_remote_addr = htmlentities($http_remote_addr);

$http_request_uri = htmlentities($html_request_uri);</blockquote>
There are a couple curious things about this patch. First, instead of transitioning away from dynamic SQL building (which can be difficult to get just right), the authors decided to sanitize the input before passing it to the dynamic SQL. Second (and more interesting), is the authors used htmlentities() to escape the data. Htmlentities() is typically used to escape data to prevent XSS attacks, not SQL Injection. Htmlentities() takes a few parameters, one of which is the optional $quote_style parameter. The $quote_style parameter defines how strings with double and single quotes will be escaped. According to the PHP documentation the three options are:
<blockquote>ENT_COMPAT  - Will convert double-quotes and leave single-quotes alone.

ENT_QUOTES  - Will convert both double and single quotes.

ENT_NOQUOTES - Will leave both double and single quotes unconverted</blockquote>
If the $quote_style is not specified, PHP will default to ENT_COMPAT. Do you think this patch will hold up to the test of time?
## Developers Solution
[cce lang="diff"]

// log all requests to register on our blog
function ttc_add_to_log( $user, $error)
{

global $wpdb;
$registration_log_table_name = $wpdb-&gt;prefix . "ttc_user_registration_log";
$request_time = $_SERVER['REQUEST_TIME'];
$http_accept = $_SERVER['HTTP_ACCEPT'];
$http_user_agent = $_SERVER['HTTP_USER_AGENT'];
$http_remote_addr = $_SERVER['REMOTE_ADDR'];

if($wpdb-&gt;get_var("show tables like  '$registration_log_table_name'") != $registration_log_table_name)  {ttc_wp_user_registration_install();
}

// wtf? accept statements coming in at over 255 chars?  Prevent sql errors and any funny business
// by shortening anything from user to 200 chars if over 255
if ( strlen($email) &gt; 200 ){ $email = substr ($email, 0, 200 ); }
if ( strlen($http_accept ) &gt; 200 ) { $http_accept = substr ( $http_accept, 0, 200 ); }
if ( strlen($http_user_agent ) &gt; 200 ) { $http_user_agent = substr ( $http_user_agent, 0, 200 ); }

+// clean input for database
+$http_accept = htmlentities($http_accept);
+$http_user_agent = htmlentities($http_user_agent);
+$http_remote_addr = htmlentities($http_remote_addr);
+$http_request_uri = htmlentities($html_request_uri);

$sql = "INSERT INTO " . $registration_log_table_name . " ( ip, email, problem, accept, agent, day )
VALUES ( '$http_remote_addr', '$user', '$error', '$http_accept', '$http_user_agent', NOW() )";
$result = $wpdb-&gt;query( $sql );
}

// add an email to our email blacklist if we decide it is an bot
function ttc_add_to_blacklist( $email )
{
global $wpdb;
$blacklist_table_name = $wpdb-&gt;prefix . "ttc_user_registration_blacklist";

if($wpdb-&gt;get_var("show tables like '$blacklist_table_name'") != $blacklist_table_name) {
ttc_wp_user_registration_install();
}

if ( strlen($email) &gt; 200 ){ $email = substr ($email, 0, 200 ); }

$sql = "INSERT INTO " . $blacklist_table_name . " ( blacklisted ) VALUES ( '$email' )";
$result = $wpdb-&gt;query( $sql );

}

// add an ip to our ip blacklist if we decide it is a bot
function ttc_add_to_ip_blacklist( $ip )
{
global $wpdb;
$ip_table_name = $wpdb-&gt;prefix . "ttc_ip_blacklist";

if($wpdb-&gt;get_var("show tables like '$ip_table_name'") != $ip_table_name) {
ttc_wp_user_registration_install();
}

$sql = "INSERT INTO " . $ip_table_name . " ( ip ) VALUES ( '$ip' )";
$result = $wpdb-&gt;query( $sql );
}

[/cce]
