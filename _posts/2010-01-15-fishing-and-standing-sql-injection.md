---
layout: post
title: Fishing and Standing - SQL Injection
tags:
- Solution
- SQL Injection
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 50d8ee7a2fe89653d27556912240d3d9
  _sexybookmarks_shortUrl: http://bit.ly/6jeUXi
  _headspace_page_title: SQL injection Vulnerability Code Example
---
## Details
__Affected Software:__ Joomla

__Fixed in Version:__  1.x

__Issue Type:__ SQL Injection

Original Code: <a title="Fishing and Standing" href="http://spotthevuln.com/2010/01/fishing-and-standing/" target="_blank">Found Here</a>
## Description
Straight up SQL injection in Joomla. The $order_by and the $direction values are taken directly from POST parameters. They are then passed to a SQL statement as part of ORDER BY clause without sanitization. The attacker is free to use traditional SQL injection methods to pass malicious SQL statements to the backend database.

This bug was fixed by sanitizing the values passed to $order_by and $direction. I like the fact that the Joomla developers used a whitelist to restrict the values passed to $order_by and $direction. The whitelist is established in the following line of code:
<blockquote>if (!in_array($order_by, array('username', 'email', 'num_posts', 'num_posts', 'registered')) || !in_array($direction, array('ASC', 'DESC')))</blockquote>
The code above makes it so the $order_by variable can only contain the following values: 'username', 'email', 'num_posts', 'num_posts', 'registered'. The line also ensure the $direction variable only contains either 'ASC' or 'DESC'.

In addition to the whitelist, the Joomla developers also escaped the variables before including them in the SQL statement.
<blockquote>

'ORDER BY'  =&gt; $forum_db-&gt;escape($order_by).' '.$forum_db-&gt;escape($direction)</blockquote>
## Developer's Solution
```diff
$form['username'] = $_POST['username'];

+//Check up for order_by and direction values
+$order_by = isset($_POST['order_by']) ? forum_trim($_POST['order_by']) : null;
+$direction = isset($_POST['direction']) ? forum_trim($_POST['direction']) : null;
+if ($order_by == null || $direction == null)
+    message($lang_common['Bad request']);
+if (!in_array($order_by, array('username', 'email', 'num_posts', 'num_posts', 'registered')) || !in_array($direction, array('ASC', 'DESC')))
+    message($lang_common['Bad request']);

($hook = get_hook('aus_find_user_selected')) ? eval($hook) : null;

...

$registered_after = forum_trim($_POST['registered_after']);
$registered_before = forum_trim($_POST['registered_before']);
-$order_by = $_POST['order_by'];
-$direction = $_POST['direction'];
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
+       'ORDER BY'  =&gt; $forum_db-&gt;escape($order_by).' '.$forum_db-&gt;escape($direction)
-     'ORDER BY'  =&gt; $order_by.' '.$direction
);


```
