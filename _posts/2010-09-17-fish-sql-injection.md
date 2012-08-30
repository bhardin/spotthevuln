---
layout: post
title: Fish - SQL Injection
tags:
- All
- attacker
- dynamic SQL
- PHP
- prepared statement
- querystring
- Solution
- SQL Injection
- sql statement
- SQLi
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_shortUrl: http://bit.ly/abkc16
  _sexybookmarks_permaHash: f01951fd7a6e0a77d92fc1b256d4f6a9
---
## Details
__Affected Software:__ Members-only Wordpress plugin

__Fixed in Version:__  0.6.7

__Issue Type:__ SQL Injection

Original Code: <a title="Fish" href="http://spotthevuln.com/2010/09/fish/" target="_blank">Found    Here</a>
## Description
This week's vuln was an easy one :)  The vulnerability affected the "members-only" plug-in for Wordpress and was patched in version 0.6.7.  There are a couple changes in the diff, but the most important change (from a security standpoint) is the transition from a string-built, dynamic SQL statement to a prepared statement.  The following lines show the SQL injection bug:
<blockquote>$feedkey = $_GET['feedkey'];
... &lt;snip&gt;...
$find_feedkey = $wpdb-&gt;get_results("SELECT umeta_id FROM $wpdb-&gt;usermeta WHERE meta_value = '$feedkey'");</blockquote>
The members-only developers assigned a value for the $feedkey variable directly from the querystring.  They then used the attacker controlled value to build a SQL statement, using the $feedkey value to complete the SQL WHERE clause.  The symptoms presented above are classic SQL injection.  I'm happy to see that the members-only developers chose to use prepared statements to fix the SQL injection.

There are a few other fixes here, but the SQL injection was the most important security fix.
<h2>Developers Solution</h2>
[cce lang="diff"]
-if (empty($userdata-&gt;ID) &amp;&amp; $members_only_opt['feed_access'] != 'feednone')  //Check if user is logged in or Feed Keys is required
+if (empty($userdata-&gt;ID) || $members_only_opt['feed_access'] != 'feednone')  //Check if user is logged in or Feed Keys is required
{
$feedkey = $_GET['feedkey'];
if (!empty($feedkey))
{
// Check if Feed Key is in the Database
-$find_feedkey = $wpdb-&gt;get_results("SELECT umeta_id FROM $wpdb-&gt;usermeta WHERE meta_value = '$feedkey'");
+$find_feedkey = $wpdb-&gt;get_results( $wpdb-&gt;prepare(
+          "SELECT umeta_id FROM $wpdb-&gt;usermeta WHERE meta_value = %s",
+          $feedkey
+) );

if (!empty($find_feedkey) &amp;&amp; $members_only_opt['feed_access'] == 'feedkeys') //If Feed Key is found and using Feed Keys
{
$feedkey_valid = TRUE;
if (empty($feedkey) &amp;&amp; $members_only_opt['feed_access'] == 'feedkeys')
{
$feed = members_only_create_feed('No Feed Key Found', $errormsg['feedkey_missing']);
-header("Content-Type: application/xml; charset=ISO-8859-1");
+header("Content-Type: application/xml; charset=" . get_bloginfo( 'charset' ), true);
echo $feed;
exit;
}
elseif ($feedkey_valid == FALSE &amp;&amp; $members_only_opt['feed_access'] == 'feedkeys')
{
$feed = members_only_create_feed('Feed Key is Invalid', $errormsg['feedkey_invalid']);
-header("Content-Type: application/xml; charset=ISO-8859-1");
+header("Content-Type: application/xml; charset=" . get_bloginfo( 'charset' ), true);
echo $feed;
exit;
}

if (empty($feedkey) &amp;&amp; $members_only_opt['feed_access'] == 'feedkeys')
{
$feed = members_only_create_feed('No Feed Key Found', $errormsg['feedkey_missing']);
-header("Content-Type: application/xml; charset=ISO-8859-1");
+header("Content-Type: application/xml; charset=" . get_bloginfo( 'charset' ), true);
echo $feed;
exit;
}
[/cce] 
