---
layout: post
title: Shape - SQL Injection
tags:
- PHP
- Solution
- SQL Injection
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Zunkerbot C&C

__Fixed in Version:__  Not Patched

__Issue Type:__ SQL Injection

Original Code: <a href="http://spotthevuln.com/2011/06/shape/">Found Here</a>

## Description
This week's bug affects the task.php for the Zunkerbot C&C. Looking at line 3, we see that magic quotes is set:  set_magic_quotes_runtime(1);
<br/>
Obviously, this was done by the malware author to prevent SQL injection attacks. Assuming everything is working correctly, a rival malware author should be able to inject any quotes to break out of existing SQL statements. Unfortunately for the Zunkerbot author, magicqoutes doesn't cover all cases. Take for example lines 59 and 70. Here we see $s_id and $s_ip are enclosed in quotes. These values should be protected against SQL injection. $s_id and $s_ip aren't the only variables being used in this SQL statement however. At the end of the two SQL statements is the following LIMIT clause: limit '.$_POST['S_RESULTS'];
<br/>
$_POST['S_RESULTS'] is NOT encapsulated within quotes, therefore the attacker is free to add their own SQL without having to use any quotes. Magic quotes does not escape semicolon characters (;), so the attacker is free to stack SQL queries and run arbitrary SQL on the Zunkerbot C&C. With this SQL injection vulnerability in hand, a rival botmaster could take over vulnerable Zunkerbot botnets with a single GET request.

## Vulnerable Code
<code lang="PHP" highlight="3,59,70">
<?php

include_once('auth.php');

set_magic_quotes_runtime(1);

if(is_readable('html.php')) include_once('html.php');
 else die('Could not find HTML library.');

if(is_readable('mycommon.php')) require('mycommon.php');
else die('Could not open configuration file.');

 if(is_readable('lang.php')) include_once('lang.php');
 else die('Could not find language library.');

$CTRL=1;
if(!isset($_GET['wohead']))
 include_once('head.php');

$msg = '';
$srch = '';

...<snip>...

 if(isset($_POST['S_COMPID'])){
 $srch = search_bot();
 };


 $param =array(
 "SRCH"=>$srch,
 "LAND"=>get_land($mres),
 "TASKS"=>get_task($mres),
 "MSG"=>$msg
 );




 echo HTML_TASK_ADD($param);

//include_once('bottom.php');
//Functions++++++++++++++++++++++++++



function search_bot(){
global $mres,$_POST;

if($_POST['S_COMPID'] == '')
if($_POST['S_IP'] == '')
return '';




if($_POST['S_COMPID'] > ''){
	$s_id = str_replace('*',"%",$_POST['S_COMPID']);
$q = 'SELECT * FROM `bots` WHERE `FCompID` like ("'.$s_id.'") limit '.$_POST['S_RESULTS'];
 $result = mysql_query($q,$mres);

 return  HTML_serch_res_tbl($result);
};


if($_POST['S_IP'] > ''){

	$s_ip = str_replace('*',"%",$_POST['S_IP']);

 $q = 'SELECT * FROM `bots` WHERE `ip_addr` like ("'.$s_ip.'") limit '.$_POST['S_RESULTS'];
 $result = mysql_query($q,$mres);

  return  HTML_serch_res_tbl($result);
};



};



function HTML_serch_res_tbl($result){
global $LNG;

	 $nr = @mysql_num_rows($result);
if(!$nr)
  return "<font color=#990000>Message</font>:<em> No Entries found.</em>";

$ret = " <br><table width=\"543\" border=\"0\" cellpadding=\"1\" cellspacing=\"1\"> "
." <tr class=\"file2\"> "
." <td colspan=\"8\" class=\"bhead\"><div align=\"center\">Select Results</div></td> "
." </tr> "
." <tr class=\"file2\"> "
." <td width=\"21\" bgcolor=#FCFCFC>Add</td> "
." <td width=\"26\" nowrap=\"nowrap\" bgcolor=#FCFCFC>Land</td> "
." <td width=\"82\" bgcolor=#FCFCFC nowrap=\"nowrap\">IP</td> "
." <td width=\"93\" bgcolor=#FCFCFC nowrap=\"nowrap\">Rep. Count total </td> "
." <td width=\"56\" bgcolor=#FCFCFC nowrap=\"nowrap\">Last Report</td> "
." <td width=\"100\" bgcolor=#FCFCFC nowrap=\"nowrap\">First Report</td> "
." <td width=\"40\" bgcolor=#FCFCFC nowrap=\"nowrap\">Bot Ver.</td> "
." <td width=\"44\" bgcolor=#FCFCFC nowrap=\"nowrap\">CompID</td> "
." </tr> ";
?>
</code>
