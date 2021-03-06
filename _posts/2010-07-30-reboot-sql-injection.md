---
layout: post
title: Reboot - SQL Injection
tags:
- attacker
- bugs
- dynamic SQL
- numeric characters
- PHP
- plugins
- querystring
- sanitization
- Solution
- SQL Injection
- sql string
- SQLi
- Wordpress
- wordpress plugin
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 87e96d15d286736e2c2efe3bc5c27d1a
  _sexybookmarks_shortUrl: http://bit.ly/cN0sAu
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _wp_old_slug: ''
---
## Details
__Affected Software:__ The Hacker's Diet (Wordpress Plugin)

__Fixed in Version:__  0.9.7b

__Issue Type:__ SQL Injection

Original Code: <a title="reboot" href="http://spotthevuln.com/2010/07/reboot/" target="_blank">Found    Here</a>
## Description
This week's vulnerability was a SQL injection vulnerability affecting the Hacker's Diet Wordpress plugin. In the vulnerable version, the plugin assigns several variables using values obtained directly from the querystring. The variable assignments are shown below:
<blockquote>$weeks = $_GET["weeks"];
$start_date = $_GET["start_date"];
$end_date = $_GET["end_date"];
$goal = $_GET["goal"];
$user_id = $_GET["user"];
$maint_mode = $_GET["maint_mode"];</blockquote>
No sanitization or validation is done before assigning the values. Once the assignments are made, the attacker controlled values are then passed to a dynamic SQL string here resulting in SQL Injection:
<blockquote>$query = "select date, weight, trend from ".$table_prefix."hackdiet_weightlog where wp_id = $user_id and date &gt; \"".date("Y-m-d", strtotime("$weeks weeks ago"))."\" order by date asc";

$query = "select date, weight, trend from ".$table_prefix."hackdiet_weightlog where wp_id = $user_id and date &gt;= \"$start_date\" and date &lt;= \"$end_date\" order by date asc";</blockquote>
The plugin authors patched this vulnerability by validating that the $_GET["user"] and $_GET["weeks"] parameters contains only numeric characters. An interesting exercise would be to trace through the code and find where the following variables are being used:
<blockquote>
$start_date = $_GET["start_date"];
$end_date = $_GET["end_date"];
$goal = $_GET["goal"];
$maint_mode = $_GET["maint_mode"];</blockquote>
## Developer's Solution
```diff
&lt;?php
include (dirname(__FILE__)."/jpgraph/jpgraph.php");
include (dirname(__FILE__)."/jpgraph/jpgraph_line.php");
include (dirname(__FILE__)."/jpgraph/jpgraph_scatter.php");

// get our db settings without loading all of wordpress every save
$html = implode('', file("../../../wp-config.php"));
$html = str_replace ("require_once", "// ", $html);
$html = str_replace ("&lt;?php", "", $html);
$html = str_replace ("?&gt;", "", $html);
eval($html);

mysql_connect(DB_HOST, DB_USER, DB_PASSWORD);
mysql_select_db(DB_NAME);

+if (!is_numeric($_GET["user"]) || !is_numeric($_GET["weeks"])) {
+   exit;
+}

$weeks = $_GET["weeks"];
$start_date = $_GET["start_date"];
$end_date = $_GET["end_date"];
$goal = $_GET["goal"];
$user_id = $_GET["user"];
$maint_mode = $_GET["maint_mode"];

if ($weeks) {
-       $query = "select date, weight, trend from ".$table_prefix."hackdiet_weightlog where wp_id = $user_id and date &gt; \"".date("Y-m-d", strtotime("$weeks weeks ago"))."\" order by date asc";
+       $query = "select date, weight, trend from ".$table_prefix."hackdiet_weightlog where wp_id = \"" . $user_id . "\" and date &gt; \"".date("Y-m-d", strtotime("$weeks weeks ago"))."\" order by date asc";
} else if ($start_date and $end_date) {
-       $query = "select date, weight, trend from ".$table_prefix."hackdiet_weightlog where wp_id = $user_id and date &gt;= \"$start_date\" and date &lt;= \"$end_date\" order by date asc";
+       $query = "select date, weight, trend from ".$table_prefix."hackdiet_weightlog where wp_id = \"" . $user_id . "\" and date &gt;= \"$start_date\" and date &lt;= \"$end_date\" order by date asc";
}

result = mysql_query($query);
if (mysql_num_rows($result)) {
if (mysql_num_rows($result) == 1) {
// only one day, gotta finagle the display

$row = mysql_fetch_assoc($result);

// fake day before
$weight_data[] = 0;
if ($goal &gt; 0) {
$goal_data[] = $goal;
}
$x_data[] = date("n/j", strtotime("yesterday", strtotime($row["date"])));

// data
$weight_data[] = $row["weight"];
if ($goal &gt; 0) {
$goal_data[] = $goal;
}
$x_data[] = date("n/j", strtotime($row["date"]));

// fake day after
$weight_data[] = 0;
if ($goal &gt; 0) {
$goal_data[] = $goal;
}
$x_data[] = date("n/j", strtotime("tomorrow", strtotime($row["date"])));
} else {
$num_rows = mysql_num_rows($result);
if ($num_rows &lt;= 7 * 2) { // 0-2 weeks
$ticks = "daily";
} else if ($num_rows &lt;= 31 * 4) { // 2 weeks - 4 months
$ticks = "weekly";
} else { // 4 months +
$ticks = "monthly";
}

$count = 1;
while ($row = mysql_fetch_assoc($result)) {
$weight_data[] = $row["weight"];
$trend_data[] = $row["trend"];
if ($goal &gt; 0) {
$goal_data[] = $goal;
}
switch ($ticks) {
case "weekly":
if ($count == 1) {
$x_data[] = date("n/j", strtotime($row["date"]));
} else {
$x_data[] = "";
if ($count == 7) {
$count = 0;
}
}
break;
case "monthly":
if (date("j", strtotime($row["date"])) == "1") {
$x_data[] = date("n/j", strtotime($row["date"]));
} else {
$x_data[] = "";
}
break;
case "daily":
default:
$x_data[] = date("n/j", strtotime($row["date"]));
break;
}

$count++;
}
}
```
