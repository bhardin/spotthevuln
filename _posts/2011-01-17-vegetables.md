---
layout: post
title: Vegetables
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
  _sexybookmarks_shortUrl: http://bit.ly/h2NaXP
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: ddb45042072a8beeb5310bf675a4d8dc
---
<blockquote><strong>People need trouble -- a little frustration to sharpen the spirit on, toughen it. Artists do; I don't mean you need to live in a rat hole or gutter, but you have to learn fortitude, endurance. Only vegetables are happy.</strong> <br><strong> - William Faulkner
</strong>
[sourcecode language="php"]
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
   $add_url = $_POST['form_url'];
   $add_desc = $_POST['form_desc'];
   if($add_url == &quot;http://&quot; || (!$add_url)){ $ERR = $ERR . &quot;&lt;br&gt;You must enter a URL to redirect to!&quot;; }
   if(!$ERR){
      $wpdb-&gt;query(&quot;INSERT INTO $table_name (link_url,link_desc) VALUES ('$add_url','$add_desc')&quot;);
         $new_url = get_option(&quot;siteurl&quot;) . &quot;/u/&quot; . mysql_insert_id();
         $MES = $MES . &quot;&lt;br&gt;The redirect URL has been added. Your new Short URL is: &quot; . $new_url;
         }
      }

if($action == &quot;edit&quot;){
   $edit_id = $_POST['id'];
   $edit_url = $_POST['form_url'];
   $edit_desc = $_POST['form_desc'];
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
...snip...
[/sourcecode] 
