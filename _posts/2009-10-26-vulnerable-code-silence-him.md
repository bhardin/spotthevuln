---
layout: post
title: Silence Him!
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#php #secure #coding'
  _edit_last: '1'
  _headspace_page_title: Vulnerable Source Code
  _headspace_description: Can you identify the security vulnerability in this code
    snippet?
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 43f32f4fb409a7f662d91747ac1cece3
  _sexybookmarks_shortUrl: http://bit.ly/cB98nt
---
<blockquote>You have not converted a man because you have silenced him.
- John Viscount Morley</blockquote>

## Vulnerable Code

[ccnLe_php]

function convert_all() {
global $wpdb;

$wpdb-&gt;query("UPDATE $wpdb-&gt;term_taxonomy SET taxonomy = 'post_tag', parent = 0 WHERE taxonomy = 'category'");
clean_category_cache($category-&gt;term_id);
}

function init() {
echo '&lt;!--'; print_r($_POST); print_r($_GET); echo '--&gt;';

if (isset($_POST['maybe_convert_all_cats'])) {
$step = 3;
} elseif (isset($_POST['yes_convert_all_cats'])) {
$step = 4;
} elseif (isset($_POST['no_dont_do_it'])) {
die('no_dont_do_it');
} else {
$step = (isset($_GET['step'])) ? (int) $_GET['step'] : 1;
}

$this-&gt;header();

if (!current_user_can('manage_categories')) {
print '&lt;div&gt;';
print '&lt;p&gt;' . __('Cheatin' uh?') . '&lt;/p&gt;';
print '&lt;/div&gt;';
} else {
switch ($step) {
case 1 :
$this-&gt;welcome();
break;

case 2 :
$this-&gt;convert_them();
break;

case 3 :
$this-&gt;convert_all_confirm();

[/ccnLe_php]
