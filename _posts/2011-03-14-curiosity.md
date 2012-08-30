---
layout: post
title: Curiosity
tags:
- Code Snippet
status: publish
type: post
quote: The cure for boredom is curiosity. There is no cure for curiosity.
quote_author: Ellen Parr
published: true
meta:
  _syntaxhighlighter_encoded: '1'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  aktt_notify_twitter: 'yes'
  aktt_tweeted: '1'
---

{% highlight html+php %}
&lt;?php

require_once('../../../wp-config.php');
require_once('../../../wp-includes/functions.php');

// CSRF attack protection. Check the Referal field to be the same
// domain of the script

$k_id = strip_tags($wpdb-&gt;escape($_GET['id']));
$k_action = strip_tags($wpdb-&gt;escape($_GET['action']));
$k_path = strip_tags($wpdb-&gt;escape($_GET['path']));
$k_imgIndex = strip_tags($wpdb-&gt;escape($_GET['imgIndex']));

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
{% endhighlight %}
