---
layout: post
title: Fishing and Standing
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 65e2e80d3e26beaf77ed803318cec730
  _sexybookmarks_shortUrl: http://bit.ly/8ZyqJJ
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Fishing and Standing
  bfa_ata_body_title: Fishing and Standing
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
---
<blockquote>There is a fine line between fishing and just standing on the shore like an idiot.

- Steven Wright</blockquote>
[ccnLe_php]
$form['username'] = $_POST['username'];
($hook = get_hook('aus_find_user_selected')) ? eval($hook) : null;

...

$registered_after = forum_trim($_POST['registered_after']);
$registered_before = forum_trim($_POST['registered_before']);
$order_by = $_POST['order_by'];
$direction = $_POST['direction'];
$user_group = $_POST['user_group'];
...

// Load the misc.php language file
require FORUM_ROOT.'lang/'.$forum_user['language'].'/misc.php';

// Find any users matching the conditions
$query = array(
'SELECT'    =&gt; 'u.id, u.username, u.email, u.title, u.num_posts, u.admin_note, g.g_id, g.g_user_title',
'FROM'        =&gt; 'users AS u',
'JOINS'        =&gt; array(
array(
'LEFT JOIN'        =&gt; 'groups AS g',
'ON'            =&gt; 'g.g_id=u.group_id'
)
),
'WHERE'        =&gt; 'u.id&gt;1 AND '.implode(' AND ', $conditions),
'ORDER BY'  =&gt; $order_by.' '.$direction <ins></ins>
);

[/ccnLe_php] 
