---
layout: post
title: Two Paradoxes - SQL Injection
tags:
- Solution
- SQL Injection
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '2'
  _headspace_page_title: SQL injection Vulnerability Code Example
  _headspace_description: SQL Injection Vulnerability Code Example
  _sexybookmarks_shortUrl: http://bit.ly/dzpTDn
  _sexybookmarks_permaHash: 2296388353bbdecc78348f481092c940
---
## Details
<strong>__Affected Software:__ WordPress</strong>

<strong>__Fixed in Version:__  2.2.1</strong>

<strong>__Issue Type:__ SQL Injection</strong>

<strong>Original Code: </strong><a href="http://spotthevuln.com/2009/11/vulnerable-code-two-paradoxes/">Found Here</a>
## Description
This one was a little tricky. This was a SQL injection vulnerability affecting WordPress Versions 2.2.0 and below. The SQL injection could be called through the XMLRPC.php file. This SQL Injection was a little tricky as it did not require single quotes or double quotes. In this case, the $args variable is attacker controlled. The WordPress developers understood the dangers of passing un-sanitized, attacker controlled variables to dynamic SQL queries and passed $args through the PHP escape() function before allowing $args to be used as part of a SQL query. This is usually sufficient in most scenarios and is the convention used throughout WordPress (escape(), then pass to dynamically built SQL statements). Unfortunately, the WordPress developers failed to realize $args[4] ($max_results) was being passed to the end of the SQL statement as part of a LIMIT clause. The LIMIT clause did not contain any single quotes or double quotes to break out of. The attacker could construct a legal SQL Injection query without the use of single quotes or double quotes (or any other special characters), allowing for SQL Injection against the WordPress installation. Although the $args variable was escaped, it didn't provide sufficient protection in this instance.

The WordPress developers addressed this vulnerability by casting $args[4] ($max_results) into an int, so although the attacker has control over this value, it cannot contain anything other than in integer value.
## Developer's Solution
```diff

/**
* WordPress XML-RPC API
* wp_suggestCategories
*/
function wp_suggestCategories($args) {
global $wpdb;

$this-&gt;escape($args);

$blog_id                                = (int) $args[0];
$username                               = $args[1];
$password                               = $args[2];
$category                               = $args[3];
-               $max_results                        = $args[4];
+               $max_results                     = (int) $args[4];

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
```
