---
layout: post
title: Money - SQL Injection
tags:
- All
- dynamic query
- dynamic SQL
- PHP
- Solution
- SQL Injection
- sql queries
- sql statements
- SQLi
- SurfIDS
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  aktt_notify_twitter: 'yes'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/fM3DqD
  _sexybookmarks_permaHash: 26399af135b575e6d7498f0479ea5ab5
---
<h3>Details</h3>
__Affected Software:__ Surfnet IDS

__Fixed in Version:__  1.03.07

__Issue Type:__ SQL Injection

Original Code: <a title="Money" href="http://spotthevuln.com/2011/01/money/" target="_blank">Found    Here</a>
<h3>Description</h3>
There were a couple of SQL injection bugs here. Beginning at line 35, we see that the Surfnet IDS developers have accepted three POST parameters and have assigned tainted values to three different variables: $keyname, $vlanid, $action. $keyname is eventually passed to three different dynamic SQL queries, all of which result in SQL injection. Those queries can be seen on lines 52, 59, and 69. $vlanid is passed to a dynamic SQL query, resulting in SQL injection. This dynamic query can be seen on line 59. Finally, $action is passed to a dynamic SQL statement, resulting in yet another SQL injection bug. This dynamic query can be found on line 69. All of the bugs were straightforward SQL injection bugs and should have been caught early in the dev cycle.

The developers addressed the issue by and/or validating all of the POST parameters before using those values in SQL statements.

<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="35-48,52,59,69,"]
&lt;?php
...snip...

include '../include/config.inc.php';
include '../include/connect.inc.php';
include '../include/functions.inc.php';

session_start();
header(&quot;Cache-control: private&quot;);

if (!isset($_SESSION['s_admin'])) {
  pg_close($pgconn);
  $address = getaddress($web_port);
  header(&quot;location: ${address}login.php&quot;);
  exit;
}

$s_org = intval($_SESSION['s_org']);
$s_admin = intval($_SESSION['s_admin']);
$s_access = $_SESSION['s_access'];
$s_access_sensor = intval($s_access{0});

if ($s_access_sensor == 0) {
  $m = 90;
  pg_close($pgconn);
  header(&quot;location: sensorstatus.php?selview=$selview&amp;m=$m&quot;);
  exit;
}

if (isset($_GET['selview'])) {
  $selview = intval($_GET['selview']);
}

$error = 0;
-$keyname = $_POST['keyname'];
-$vlanid = $_POST['vlanid'];
-$action = $_POST['action'];
-if (isset($_POST[tapip])) {
+$keyname = pg_escape_string($_POST['keyname']);
+	$vlanid = intval($_POST['vlanid']);
+	$action = pg_escape_string($_POST['action']);
+	$action_pattern = '/^(NONE|REBOOT|SSHOFF|SSHON|CLIENT|RESTART|BLOCK)$/';
+	if (preg_match($action_pattern, $action) != 1) {
+	  $m = 44;
+	  $error = 1;
+	}
+
+if (isset($_POST[tapip]) &amp;&amp; $error != 1) {

  $tapip = pg_escape_string(stripinput($_POST[tapip]));
  if (preg_match($ipregexp, $tapip)) {
    $sql_checkip = &quot;SELECT tapip FROM sensors WHERE tapip = '$tapip' AND NOT keyname = '$keyname'&quot;;
    $result_checkip = pg_query($pgconn, $sql_checkip);
    $checkip = pg_num_rows($result_checkip);
    if ($checkip &gt; 0) {
      $m = 101;
      $error = 1;
    } else {
      $sql_updatestatus = &quot;UPDATE sensors SET tapip = '$tapip' WHERE keyname = '$keyname' AND vlanid ='$vlanid'&quot;;
      $result_updatestatus = pg_query($pgconn, $sql_updatestatus);
      $m = 7;
    }
  } else {
    $m = 102;
    $error = 1;
  }
}
if ($error == 0) {
  $sql_updatestatus = &quot;UPDATE sensors SET action = '&quot; .$action. &quot;' WHERE keyname = '$keyname'&quot;;
  $result_updatestatus = pg_query($pgconn, $sql_updatestatus);
  $m = 7;
}

pg_close($pgconn);
if ($m != 1) {
  header(&quot;location: sensorstatus.php?selview=$selview&amp;m=$m&amp;key=$keyname&quot;);
} else {
  header(&quot;location: sensorstatus.php?selview=$selview&amp;m=$m&quot;);
}
?&gt;
[/sourcecode]
