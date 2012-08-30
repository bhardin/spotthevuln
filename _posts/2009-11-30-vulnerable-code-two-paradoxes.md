---
layout: post
title: Two Paradoxes
tags:
- All
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Vulnerable Source Code
  _headspace_description: Can you identify the security vulnerability in this code
    snippet?
  _sexybookmarks_shortUrl: http://bit.ly/dm2Apg
  _sexybookmarks_permaHash: 58a82cc7ffd8e45c4f44f80bd77dfd98
---
<blockquote>Two paradoxes are better than one; they may even suggest a solution.
- Edward Teller (1908 - 2003)</blockquote>
[ccnLe_php]

function wp_suggestCategories($args) {
global $wpdb;

$this-&gt;escape($args);

$blog_id = (int) $args[0];
$username = $args[1];
$password = $args[2];
$category  = $args[3];
$max_results  = $args[4];

if(!$this-&gt;login_pass_ok($username, $password)) {
return($this-&gt;error);
}

// Only set a limit if one was provided.
$limit = "";
if(!empty($max_results)) {
$limit = "LIMIT {$max_results}";
}

$category_suggestions = $wpdb-&gt;get_results("
SELECT cat_ID category_id,
cat_name category_name
FROM {$wpdb-&gt;categories}
WHERE cat_name LIKE '{$category}%'
{$limit}
");

return($category_suggestions);
}
[/ccnLe_php] 
