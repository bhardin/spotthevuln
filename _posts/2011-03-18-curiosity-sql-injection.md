---
layout: post
title: Curiosity - SQL Injection
tags:
- dynamic SQL
- PHP
- Solution
- SQL Injection
- sql query
- SQLi
- Wordpress
- wordpress plugin
status: publish
type: post
published: true
meta:
  _aktt_hash_meta: ! '#secure #code #dev'
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Comment-Rating Plugin

__Fixed in Version:__  2.9.24

__Issue Type:__ SQL Injection (SQLi)

Original Code: <a href="http://spotthevuln.com/2011/03/curiosity/">Found Here</a>
## Details
This week's vulnerability was a tricky one. The bug patched in this change list affected the Comment-Rating plugin for WordPress (fixed in 2.9.24). Let's take the bug step by step. First, the application takes a user/attacker supplied value and runs it through an escaping function here (line 9):

$k_id = strip_tags($wpdb->escape($_GET['id']));

So, $k_id is now tainted and contains an escaped value provided by the attacker. A few lines later, we see the following code:

if($k_id && $k_action && $k_path) {
    //Check to see if the comment id exists and grab the rating
    $query = "SELECT * FROM `$table_name` WHERE ck_comment_id = $k_id";
    $result = mysql_query($query);

The code above checks for a specific condition (which is a condition controllable by the attacker) then proceeds to build and execute a SQL query. On line 22 we see $k_id is used to build a dynamic SQL statement. Variables usage within stings are valid in PHP (http://php.net/manual/en/language.types.string.php - see Variable parsing). $k_id is escaped so we should be ok here...right?  Actually, in this case escaping isn't sufficient to prevent SQL injection. Escaping functions typically work by preventing a variable value from breaking out of quotes, unfortunately in this case there are no quotes to break out of. $k_id is designed to be a numeric value not a string, so there is no need to encapsulate the $k_id value in quotes. Although $k_id is designed to be numeric, there was nothing that would prevent an attacker from providing an arbitrary value for $k_id. For example, an attacker could provide a value like this for $k_id:

99999 union select uname, passwd from users

As you can see, there are no special characters (double quotes, single quotes, or database escape characters) in the string above that would have been escaped by a database escaping function. When used to build the $query variable, we end up with:

$query = "SELECT * FROM `$table_name` WHERE ck_comment_id = <span style="color: #ff0000;">99999 union select uname, passwd from users</span>";

The developers addressed this vulnerability by validating that $k_id is indeed numeric before using the value to build a dynamic SQL statement.
## Developer's Solution
[sourcecode language="diff" highlight="14,15"]
&lt;?php

require_once('../../../wp-config.php');
require_once('../../../wp-includes/functions.php');

// CSRF attack protection. Check the Referal field to be the same
// domain of the script

$k_id = strip_tags($wpdb-&gt;escape($_GET['id']));
$k_action = strip_tags($wpdb-&gt;escape($_GET['action']));
$k_path = strip_tags($wpdb-&gt;escape($_GET['path']));
$k_imgIndex = strip_tags($wpdb-&gt;escape($_GET['imgIndex']));

+// prevent SQL injection
+if (!is_numeric($k_id)) die('error|Query error');

$table_name = $wpdb-&gt;prefix . 'comment_rating';
$comment_table_name = $wpdb-&gt;prefix . 'comments';

if($k_id &amp;&amp; $k_action &amp;&amp; $k_path) {
    //Check to see if the comment id exists and grab the rating
    $query = &quot;SELECT * FROM `$table_name` WHERE ck_comment_id = $k_id&quot;;
    $result = mysql_query($query);

	if(!$result) { die('error|mysql: '.mysql_error()); }

   if(mysql_num_rows($result))
	{
      $duplicated = 0;  // used as a counter to off set duplicated votes
      if($row = @mysql_fetch_assoc($result))
      {
			if(strstr($row['ck_ips'], getenv(&quot;REMOTE_ADDR&quot;))) {
            // die('error|You have already voted on this item!');
            // Just don't count duplicated votes
            $duplicated = 1;
            $ck_ips = $row['ck_ips'];
         }
         else {
            $ck_ips = $row['ck_ips'] . ',' . getenv(&quot;REMOTE_ADDR&quot;); // IPs are separated by ','
         }
      }

      $total = $row['ck_rating_up'] - $row['ck_rating_down'];
      if($k_action == 'add') {
         $rating = $row['ck_rating_up'] + 1 - $duplicated;
         $direction = 'up';
         $total = $total + 1 - $duplicated;
      }
      elseif($k_action == 'subtract')
      {
         $rating = $row['ck_rating_down'] + 1 - $duplicated;
         $direction = 'down';
         $total = $total - 1 + $duplicated;
      } else {
            die('error|Try again later'); //No action given.
      }

      if (!$duplicated)
      {
         $query = &quot;UPDATE `$table_name` SET ck_rating_$direction = '$rating', ck_ips = '&quot; . $ck_ips  . &quot;' WHERE ck_comment_id = $k_id&quot;;
         $result = mysql_query($query);
         if(!$result)
         {
            // die('error|query '.$query);
            die('error|Query error');
         }

         // Now duplicated votes will not
         if(!mysql_affected_rows())
         {
            die('error|affected '. $rating);
         }

         $karma_modified = 0;
         if (get_option('ckrating_karma_type') == 'likes' &amp;&amp; $k_action == 'add') {
            $karma_modified = 1; $karma = $rating;
         }
         if (get_option('ckrating_karma_type') == 'dislikes' &amp;&amp; $k_action == 'subtract') {
            $karma_modified = 1; $karma = $rating;
         }
         if (get_option('ckrating_karma_type') == 'both') {
            $karma_modified = 1; $karma = $total;
         }

         if ($karma_modified) {
            $query = &quot;UPDATE `$comment_table_name` SET comment_karma = '$karma' WHERE comment_ID = $k_id&quot;;
            $result = mysql_query($query);
            if(!$result) die('error|Comment Query error');
         }
      }
   } else {
        die('error|Comment doesnt exist'); //Comment id not found in db, something wrong ?
   }
} else {
    die('error|Fatal: html format error');
}

// Add the + sign,
if ($total &gt; 0) { $total = &quot;+$total&quot;; }

//This sends the data back to the js to process and show on the page
// The dummy field will separate out any potential garbage that
// WP-superCache may attached to the end of the return.
echo(&quot;done|$k_id|$rating|$k_path|$direction|$total|$k_imgIndex|dummy&quot;);
?&gt;
[/sourcecode]
