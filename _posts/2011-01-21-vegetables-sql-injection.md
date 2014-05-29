---
layout: post
title: Vegetables - SQL Injection
tags:
- All
- dynamic SQL
- dynamic sql statements
- PHP
- short url
- Solution
- SQL Injection
- SQLi
- Wordpress
- wordpress plugin
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/eT91HK
  _sexybookmarks_permaHash: ff5d64e46f2dc80235ac4913fa821b1c
---
<h3>Details</h3>
__Affected Software:__ Short URL Plugin

__Fixed in Version:__  Changeset 55280

__Issue Type:__ SQL Injection

Original Code: <a title="Vegetables" href="https://spotthevuln.com/2011/01/vegetables/" target="_blank">Found    Here</a>
<h3>Description</h3>
This weeks' vulnerabilities were a couple of SQL injection bugs in the Short URL Plugin for Wordpress. The symptoms for the issues indicate classic SQL injection, let's have a quick look at the code. First, looking over the code sample, we see a couple of dynamically built SQL statements. It would probably make sense to spend a bit of time and convert these dynamic SQL statements into prepared statements, that way you won't have to worry about a code change inadvertently re-introducing a SQL injection flaw or an escaping filter bypass. With dynamically built SQL statements we'll also have to trace each variable until we can determine whether the value can be controlled by an attacker. Lucky for us, the variable assignments are very close to the SQL statements. In the vulnerable sample, we see that the author is taking values directly from a POST request and using those tainted values to build SQL statements. Looking at the check-in, we see that the developer chose to use Wordpress' built-in escaping function for escaping user/attacker controlled data before passing it to a SQL statement.

Although the checked-in fixes were straightforward, I was surprised to see that the developers missed an obvious SQL injection on line 56. Same classic SQL injection symptoms, the only difference is the dynamic SQL being built is a DELETE SQL statement as opposed to an INSERT or UPDATE. For those that are wondering... YES, this SQL injection is still present in the latest version of the plug-in!  If you happen to be using this plug-in on your website, I would recommend you escape $delete_id before passing it to a SQL statement!  I notified the plug-in author, hopefully they'll be a patch soon.

Is this the first Spot-The-Vuln.com 0day?


<h3>Developers Solution</h3>
[sourcecode language="diff" highlight="27-30,41-46,56"]
&lt;?php
...snip...
function kd_admin_options_su(){
   global $table_prefix, $wpdb, $user_ID;

   $table_name = $table_prefix . &quot;short_url&quot;;

   if($wpdb-&gt;get_var(&quot;show tables like '$table_name'&quot;) != $table_name){

   $sql = &quot;CREATE TABLE &quot;.$table_name.&quot; (
   link_id int(11) NOT NULL auto_increment,
   link_url text NOT NULL,
   link_desc text NOT NULL,
   link_count int(11) NOT NULL default '0',
   PRIMARY KEY  (`link_id`)
   );&quot;;

   require_once(ABSPATH . 'wp-admin/upgrade-functions.php');
   dbDelta($sql);

   }


   if(isset($_POST['action'])) {
      $action = $_POST['action'];

if($action == &quot;create&quot;){
-  $add_url = $_POST['form_url'];
-  $add_desc = $_POST['form_desc'];
+  $add_url = $wpdb-&gt;escape($_POST['form_url']);
+  $add_desc = $wpdb-&gt;escape($_POST['form_desc']);

   if($add_url == &quot;http://&quot; || (!$add_url)){ $ERR = $ERR . &quot;&lt;br&gt;You must enter a URL to redirect to!&quot;; }
   if(!$ERR){
      $wpdb-&gt;query(&quot;INSERT INTO $table_name (link_url,link_desc) VALUES ('$add_url','$add_desc')&quot;);
         $new_url = get_option(&quot;siteurl&quot;) . &quot;/u/&quot; . mysql_insert_id();
         $MES = $MES . &quot;&lt;br&gt;The redirect URL has been added. Your new Short URL is: &quot; . $new_url;
         }
      }

if($action == &quot;edit&quot;){
-  $edit_id = $_POST['id'];
-  $edit_url = $_POST['form_url'];
-  $edit_desc = $_POST['form_desc'];
+  $edit_id = $wpdb-&gt;escape($_POST['id']);
+  $edit_url = $wpdb-&gt;escape($_POST['form_url']);
+  $edit_desc = $wpdb-&gt;escape($_POST['form_desc']);

   if($edit_url == &quot;http://&quot; || (!$edit_url)){ $ERR = $ERR . &quot;&lt;br&gt;You must enter a URL to redirect to!&quot;; }
   if(!$ERR){
      $wpdb-&gt;query(&quot;UPDATE $table_name SET link_url='$edit_url',link_desc='$edit_desc' WHERE link_id = $edit_id&quot;);
         $MES = $MES . &quot;&lt;br&gt;The redirect URL has been modified.&quot;;
         }
      }


if($action == &quot;delete&quot;){
   $delete_id = $_POST['id'];
   $wpdb-&gt;query(&quot;DELETE FROM $table_name WHERE link_id = '$delete_id'&quot;);
   $MES = $MES . &quot;&lt;br&gt;Redirect deleted!&quot;;
   }

if($action == &quot;clearall&quot;){
        $wpdb-&gt;query(&quot;UPDATE $table_name SET link_count='0' WHERE link_count &gt; 0&quot;);
   $MES = $MES . &quot;&lt;br&gt;Counts have been reset!&quot;;
   }
}
   ?&gt;
   &lt;div class=wrap&gt;
   &lt;form method=&quot;post&quot;&gt;
      &lt;h2&gt;Short URL Admin&lt;/h2&gt;
&lt;?php if($ERR){ echo &quot;&lt;p&gt;&quot; . $ERR . &quot;&lt;/p&gt;&quot;; }
if($MES){ echo &quot;&lt;p&gt;&quot; . $MES . &quot;&lt;/p&gt;&quot;; } ?&gt;
      &lt;p&gt;Short URL allows you to create shorter URL's and keeps track of how many
times a link has been clicked. It's useful for managing downloads, keeping track
of outbound links and for masking URL's. Clicking the Clear All Clicks button
will reset the count for each entry. Visit the &lt;a href=&quot;http://www.harleyquine.com/php-scripts/short-url-plugin/&quot;&gt;plugin page&lt;/a&gt; for more information about this plugin.&lt;/p&gt;

&lt;h2&gt;Current Redirects&lt;/h2&gt;
&lt;table class=&quot;widefat&quot;&gt;
   &lt;thead&gt;
   &lt;tr&gt;
   &lt;th scope=&quot;col&quot;&gt;Short URL (The URL to use)&lt;/th&gt;
   &lt;th scope=&quot;col&quot;&gt;Real URL (Where it redirects to)&lt;/th&gt;
   &lt;th scope=&quot;col&quot;&gt;Notes&lt;/th&gt;
   &lt;th scope=&quot;col&quot;&gt;Amount of Clicks&lt;/th&gt;
   &lt;th scope=&quot;col&quot;&gt;Manage&lt;/th&gt;
   &lt;/tr&gt;
      &lt;/thead&gt;
   &lt;tbody id=&quot;the-list&quot;&gt;
&lt;?php
   $rowdata = $wpdb-&gt;get_results(&quot;SELECT * FROM $table_name&quot;);

   foreach ($rowdata as $row) {
   $is_editing = $_POST['edit_id'];
   if($is_editing){
      if($is_editing == $row-&gt;link_id){ $EDIT = 1; $EDIT_ID = $row-&gt;link_id; $EDIT_URL = $row-&gt;link_url; $EDIT_DESC = $row-&gt;link_desc; }
      }
?&gt;
   &lt;tr class='&lt;?php echo $class; ?&gt;'&gt;

   &lt;th scope=&quot;row&quot;&gt;&lt;a href=&quot;&lt;? echo get_option(&quot;siteurl&quot;) . &quot;/u/&quot; . $row-&gt;link_id; ?&gt;&quot; target=&quot;_blank&quot;&gt;&lt;? echo get_option(&quot;siteurl&quot;) . &quot;/u/&quot; . $row-&gt;link_id; ?&gt;&lt;/a&gt;&lt;/th&gt;
   &lt;td&gt;&lt;? echo $row-&gt;link_url; ?&gt;&lt;/td&gt;
   &lt;td&gt;&lt;? echo $row-&gt;link_desc; ?&gt;&lt;/td&gt;
   &lt;td&gt;&lt;? echo $row-&gt;link_count; ?&gt;&lt;/td&gt;
   &lt;td&gt;&lt;form method=&quot;post&quot; name=&quot;delete&quot;&gt;&lt;input type=&quot;hidden&quot; name=&quot;action&quot; value=&quot;delete&quot;&gt;&lt;input type=&quot;hidden&quot; name=&quot;id&quot; value=&quot;&lt;? echo $row-&gt;link_id; ?&gt;&quot;&gt;&lt;input type=&quot;submit&quot; value=&quot;Delete&quot;&gt;&lt;/form&gt;&lt;form method=&quot;post&quot; name=&quot;edit&quot;&gt;&lt;input type=&quot;hidden&quot; name=&quot;edit_id&quot; value=&quot;&lt;? echo $row-&gt;link_id; ?&gt;&quot;&gt;&lt;input type=&quot;submit&quot; value=&quot;Edit&quot;&gt;&lt;/form&gt;&lt;/td&gt;
...snip...[/sourcecode]
