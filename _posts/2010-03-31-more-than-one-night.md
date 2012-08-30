---
layout: post
title: More Than One Night
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: b8218d2018ce195078dfb87ea8383fb4
  _sexybookmarks_shortUrl: http://bit.ly/bPX7Vb
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
<blockquote><strong>Sometimes I lie awake at night, and I ask, "Where have I gone wrong?"   </strong><strong>Then a voice says to me, "This is going to take more than one night.".</strong><strong>
- Charlie Brown
</strong></blockquote>
[ccnLe_php]

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

[/ccnLe_php] 
