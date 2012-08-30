---
layout: post
title: Attack the Hot Fudge Sundae - XSS
tags:
- Cross-Site Scripting (XSS)
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_tweeted: '1'
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  _headspace_description: ! 'Background__Affected Software:__ WordpressFixed in: 2.'
  _sexybookmarks_permaHash: b33df3a090a5c54b41cc3633dea550c8
  _sexybookmarks_shortUrl: http://bit.ly/12smd1
---

## Background
__Affected Software:__ <a href="http://spotthevuln.com/category/software/wordpress/">Wordpress</a>

__Fixed in:__ 2.8

__Issue Type:__ <a href="http://spotthevuln.com/category/vulnerability/xss/">Cross Site Scripting</a></div>
<strong>Original Code: </strong><a href="http://spotthevuln.com/2009/10/vulnerable-source-code-hot-fudge-sundae/">Found Here</a>
## Description
<div>

The vulnerable code takes explicit actions to escape Arrays before echoing the contents of the Array back to the user.

Objects however, are NOT explicitly escaped in the vulnerable code. If an attacker could pass their payload as a PHP object, it would be echoed back to the user without encoding or sanitation.

The WordPress developers simple placed conditional logic to determine whether the data being passed was from an array or an object and escaped the data as needed, protecting the user from XSS.</div>
## Developers Solution
[cc lang="diff"]

/**
* Update a post with new post data.
*
* The date does not have to be set for drafts. You can set the date and it will
* not be overridden.
*
* @since 1.0.0
*
- * @param array|object $postarr Post data.
+ * @param array|object $postarr Post data. Arrays are expected to be escaped, objects are not.
* @return int 0 on failure, Post ID on success.
*/
function wp_update_post($postarr = array()) {
-       if ( is_object($postarr) )
+       if ( is_object($postarr) ) {
+               // non-escaped post was passed
$postarr = get_object_vars($postarr);
+               $postarr = add_magic_quotes($postarr);
+       }
// First, get all of the original fields
$post = wp_get_single_post($postarr['ID'], ARRAY_A);

// Escape data pulled from DB.
$post = add_magic_quotes($post);

// Passed post category list overwrites existing category list if not empty.
if ( isset($postarr['post_category']) &amp;&amp; is_array($postarr['post_category'])
&amp;&amp; 0 != count($postarr['post_category']) )
$post_cats = $postarr['post_category'];
else
$post_cats = $post['post_category'];

// Drafts shouldn't be assigned a date unless explicitly done so by the user
if ( in_array($post['post_status'], array('draft', 'pending')) &amp;&amp; empty($postarr['edit_date']) &amp;&amp;
('0000-00-00 00:00:00' == $post['post_date_gmt']) )
$clear_date = true;
else
$clear_date = false;

// Merge old and new fields with new fields overwriting old ones.
$postarr = array_merge($post, $postarr);
$postarr['post_category'] = $post_cats;
if ( $clear_date ) {
$postarr['post_date'] = current_time('mysql');
$postarr['post_date_gmt'] = '';
}

if ($postarr['post_type'] == 'attachment')
return wp_insert_attachment($postarr);

return wp_insert_post($postarr);
}

[/cc] 