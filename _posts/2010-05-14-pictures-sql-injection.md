---
layout: post
title: Pictures - SQL Injection
tags:
- Consumption
- dynamic SQL
- PHP
- plugins
- Solution
- SQL Injection
- SQLi
- Wordpress
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 926263f7b8a9f45d70748112c9dd9aaf
  _sexybookmarks_shortUrl: http://bit.ly/axNpMI
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
---
## Details
__Affected Software:__ Short URL Plugin

__Fixed in Version:__  2.0

__Issue Type:__ SQL Injection

Original Code: <a title="Pictures" href="http://spotthevuln.com/2010/05/pictures/" target="_blank">Found Here</a>
## Description
This was a vulnerability that affected the Short URL Wordpress plugin. The vulnerability is very straightforward and should have been easily detected by a security code reviewer. The vulnerable code section takes attacker controlled data directly from $_POST['form_url'], $_POST['form_desc'], and $_POST['id'] and uses the tainted value immediately in dynamically built SQL statements. One interesting piece of this particular code fix is that the developers chose to implement the code fixes near the assignment of the variable (as opposed to near consumption, in the SQL statement).

Another interesting piece of the code fix is the logic for the following conditional:
<blockquote>if($action == "delete"){</blockquote>
looks like the devs may have forgotten something :)
## Developer's Solution
```diff

&lt;?php
require_once(ABSPATH . 'wp-admin/upgrade-functions.php');
   dbDelta($sql);

   }
   if(isset($_POST['action'])) {
      $action = $_POST['action'];

if($action == "create"){
-  $add_url = $_POST['form_url'];
-  $add_desc = $_POST['form_desc'];
+  $add_url = $wpdb-&gt;escape($_POST['form_url']);
+  $add_desc = $wpdb-&gt;escape($_POST['form_desc']);
   if($add_url == "http://" || (!$add_url)){ $ERR = $ERR . "&lt;br&gt;You must enter a URL to redirect to!"; }
   if(!$ERR){
      $wpdb-&gt;query("INSERT INTO $table_name (link_url,link_desc) VALUES ('$add_url','$add_desc')");
         $new_url = get_option("siteurl") . "/u/" . mysql_insert_id();
         $MES = $MES . "&lt;br&gt;The redirect URL has been added. Your new Short URL is: " . $new_url;
         }
      }

if($action == "edit"){
-  $edit_id = $_POST['id'];
-  $edit_url = $_POST['form_url'];
-  $edit_desc = $_POST['form_desc'];
+  $edit_id = $wpdb-&gt;escape($_POST['id']);
+  $edit_url = $wpdb-&gt;escape($_POST['form_url']);
+  $edit_desc = $wpdb-&gt;escape($_POST['form_desc']);
   if($edit_url == "http://" || (!$edit_url)){ $ERR = $ERR . "&lt;br&gt;You must enter a URL to redirect to!"; }
   if(!$ERR){
      $wpdb-&gt;query("UPDATE $table_name SET link_url='$edit_url',link_desc='$edit_desc' WHERE link_id = $edit_id");
         $MES = $MES . "&lt;br&gt;The redirect URL has been modified.";
         }
      }


if($action == "delete"){
   $delete_id = $_POST['id'];
   $wpdb-&gt;query("DELETE FROM $table_name WHERE link_id = '$delete_id'");
   $MES = $MES . "&lt;br&gt;Redirect deleted!";
   }

if($action == "clearall"){
        $wpdb-&gt;query("UPDATE $table_name SET link_count='0' WHERE link_count &gt; 0");
   $MES = $MES . "&lt;br&gt;Counts have been reset!";
   }
}
   ?&gt;
   &lt;div&gt;
   &lt;form method="post"&gt;
      &lt;h2&gt;Short URL Admin&lt;/h2&gt;
&lt;?php if($ERR){ echo "&lt;p&gt;" . $ERR . "&lt;/p&gt;"; }
if($MES){ echo "&lt;p&gt;" . $MES . "&lt;/p&gt;"; } ?&gt;
      &lt;p&gt;Short URL allows you to create shorter URL's and keeps track of how many
times a link has been clicked. It's useful for managing downloads, keeping track
of outbound links and for masking URL's. Clicking the Clear All Clicks button
will reset the count for each entry. Visit the &lt;a href="<a href="http://www.harleyquine.com/php-scripts/short-url-plugin/%22%3Eplugin">http://www.harleyquine.com/php-scripts/short-url-plugin/"&gt;plugin</a> page&lt;/a&gt; for more information about this plugin.&lt;/p&gt;

&lt;h2&gt;Current Redirects&lt;/h2&gt;
&lt;table&gt;
   &lt;thead&gt;
   &lt;tr&gt;
   &lt;th scope="col"&gt;Short URL (The URL to use)&lt;/th&gt;
   &lt;th scope="col"&gt;Real URL (Where it redirects to)&lt;/th&gt;
   &lt;th scope="col"&gt;Notes&lt;/th&gt;
   &lt;th scope="col"&gt;Amount of Clicks&lt;/th&gt;
   &lt;th scope="col"&gt;Manage&lt;/th&gt;
   &lt;/tr&gt;
      &lt;/thead&gt;
   &lt;tbody id="the-list"&gt;
?&gt;

```
