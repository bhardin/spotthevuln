---
layout: post
title: More Than One Night - Defense in Depth
tags:
- addslashes
- Defense In Depth
- information disclosure
- PHP
- Solution
- SQL Injection
- SQLi
- Wordpress
status: publish
type: post
published: true
meta:
  _sexybookmarks_shortUrl: http://bit.ly/aw0TgH
  _sexybookmarks_permaHash: 771052054598c247d72650ac5c5b23ad
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: More Than One Night - Defense in Depth
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: More Than One Night - Defense in Depth
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ WordPress

__Fixed in Version:__  2.1

__Issue Type:__ Defense in Depth

Original Code: <a title="More than one night" href="http://spotthevuln.com/2010/03/more-than-one-night/" target="_blank">Found Here</a>
## Description
I found this issue interesting for a couple reasons. Upon first glance, the patch appears to be a defense against SQL Injection and in essence, it is. It seems that the $q['cat'] value is controlled by the user and is eventually used to help build a SQL statement. Before the $q['cat'] value is used in a SQL statement, it is actually sanitized by the following lines:
<blockquote>$q['cat'] = ''.urldecode($q['cat']).'';

$q['cat'] = addslashes_gpc($q['cat']);</blockquote>
Once the value is sanitized, it is used to build various SQL statements. Now this particular patch was developed by the WordPress team because they discovered that when a user/attacker passes a "." (period character) to $q['cat'], it would cause a SQL error which would be displayed to the user. While a single period character doesn't give the attacker the ability to execute arbitrary SQL, it does give the attacker an information disclosure bug. In an academic sense however, the attacker has convinced the database that their provided value should be interpreted as code as opposed to data (ala SQL Injection). The reason the period character slips through is because it is not defined as a special character in the addslashes() php function… this could be useful in other situations.

The WordPress prevented the information leak by checking to see if $q['cat'] is an integer value. The patch here is a single line fix.
<h2>Developers Solution</h2>
[cce lang="diff"]

// Category stuff

if ((empty($q['cat'])) || ($q['cat'] == '0') ||

// Bypass cat checks if fetching specific posts

( $this-&gt;is_single || $this-&gt;is_page )) {

$whichcat='';

} else {

$q['cat'] = ''.urldecode($q['cat']).'';

$q['cat'] = addslashes_gpc($q['cat']);

$join = " LEFT JOIN $wpdb-&gt;post2cat ON ($wpdb-&gt;posts.ID = $wpdb-&gt;post2cat.post_id) ";

$cat_array = preg_split('/[,\s]+/', $q['cat']);

$in_cats = $out_cats = '';

foreach ( $cat_array as $cat ) {

+ $cat = intval($cat);

$in = strstr($cat, '-') ? false : true;

$cat = trim($cat, '-');

if ( $in )

$in_cats .= "$cat, " . get_category_children($cat, '', ', ');

else

$out_cats .= "$cat, " . get_category_children($cat, '', ', ');

}

$in_cats = substr($in_cats, 0, -2);

$out_cats = substr($out_cats, 0, -2);

if ( strlen($in_cats) &gt; 0 )

$in_cats = " AND category_id IN ($in_cats)";

if ( strlen($out_cats) &gt; 0 )

$out_cats = " AND category_id NOT IN ($out_cats)";

$whichcat = $in_cats . $out_cats;

$distinct = 'DISTINCT';

}

// Category stuff for nice URIs

global $cache_categories;

if ('' != $q['category_name']) {

[/cce] 
