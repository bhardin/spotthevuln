---
layout: post
title: Rabbit - Auth Bypass and SQLi
tags:
- PHP
- Solution
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
__Affected Software:__ BlackEnergy C&C

__Fixed in Version:__  Not Patched

__Issue Type:__ Authentication Bypass and SQL Injection

Original Code: <a href="http://spotthevuln.com/2011/06/rabbit/">Found Here</a>
## Details
A couple of interesting bugs here. As Abe astutely pointed out, pretty much all of the PHP at the end of the code sample is vulnerable to SQL injection. Veteran Spot the Vuln readers can easily spot the tainted $_POST and $_GET parameters being passed directly into dynamically built SQL statements. This obviously results in compromise of the backend database and the application. I've highlighted the SQL injection points in the code sample below. The injection points are fairly obvious.

Now, onto the more interesting bug. In order to reach the code paths that are vulnerable to SQL injection, we must first "login" to the application. The "login" routine is contained in lines 11-25.

Later in the code, $logined is checked before allowing the user to reach the vulnerable code paths. That code can be found on lines 79-96. So let's work backwards here. We see obvious SQL injection bugs in several places, but this code paths can only be reached if the $logined variable is true (line 95). The value for the $logined variable is controlled by the following else statement:
<code lang="PHP">
else {
    		$logined = @$_COOKIE['logined'];
    		if ($logined === $pass)
    		{
          		$logined = true;
    		}
	}
</code>
Looking at the code above, we see that the application is taking the value for a Cookie named "logined" and assigning that value to variable $logined. The application then checks to see if $logined is equal to the password for the registered user ($logined === $pass). If $logined === $pass, then the application sets the $logined value to true. In this example, the developer missed a critical case. The client (browser) is free to tamper any part of the HTTP request, including the COOKIE values sent to the application. All we need to do is issue a HTTP request with a cookie of: logined = true;  <- - any value for the logined cookie will work.

If we pass Cookie: logined = true; in our HTTP request, our tainted cookie value will be assigned to the $logined variable. You can see this in line 20 of the code sample. Although we will fail the $logined === $pass check, the application fails to clear the $logined variable value so our tainted value remains. Later in the code, $logined is checked... if it is true the application assumes we are logged in and gives us access to the vulnerable code paths. We can now exploit the SQL injection vulnerabilities and extract the real $pass value because in order for the comparison to be done, the value must have been stored in cleartext in the database.

There you go, authentication bypass + SQL injection for Blackenergy C&C :)



## Developers Solution
<code lang="PHP" highlight="109,120,132">
<?

	include "common.php";

	$luser = @$_POST['user'];
	$lpass = @$_POST['pass'];
	$login = @$_POST['login'];

	$logined = false;

	if ($login)
	{
    		Sleep(1);
    		if ($luser == $user && $lpass == $pass)
    		{
        		setcookie("logined", $pass);
        		header("location: index.php");
    		}
	} else {
    		$logined = @$_COOKIE['logined'];
    		if ($logined === $pass)
    		{
          		$logined = true;
    		}
	}

?>
<html>
<head>
<STYLE type=text/css>
BODY {
        BACKGROUND: #666666;
        FONT: 11px Verdana, Arial
}
P {
        FONT: 10pt Verdana, Arial;
        COLOR: #000000;
        TEXT-ALIGN: justify
}
TD {
        FONT: 8pt Verdana,Arial;
        COLOR: #000000
}
A {
        COLOR: #000000;
        TEXT-DECORATION: underline
}
A.nav {
        COLOR: #000000; TEXT-DECORATION: none
}
A:hover {
        BACKGROUND: silver; TEXT-DECORATION: underline overline
}
INPUT, SELECT, TEXTAREA {
        FONT-SIZE: 8pt; FONT-FAMILY: Verdana, Helvetica;
        border: 1px solid silver;
        color: #606060;
        background-color: #222222;
        margin-top: 0px;
        margin-bottom: 0px;
}
.HEAD TD {
       BACKGROUND: silver; TEXT-ALIGN: center; FONT-WEIGHT: bold
}
.SLIST TD {
        BACKGROUND: #888888
}
</STYLE>
</head>
<body>

<script>
function wnd( url )
{
        window.open( url, "", "statusbar=no,menubar=no,toolbar=no,scrollbars=yes,resizable=no,width=600,height=400");
}
</script>

<?

    	if (!$logined)
    	{

?>

<form action=index.php method=POST>
<table>
<tr><td>user:</td><td><input type=text name=user></td></tr>
<tr><td>pass:</td><td><input type=password name=pass></td></tr>
<tr><td></td><td><input type=submit name=login value=login></td></tr>
</table>
</form>

<?
        	exit;
	}

	switch (@$_GET['d'])
	{
	case "add":
        	if (empty($_POST['url']))
			break;

        	if (isset($_POST['country'])) $_POST['country'] = strtoupper($_POST['country']);

        	$sql = "INSERT INTO `files`
                	(`url`, `dnum`, `country`)
                	VALUES
                	('{$_POST['url']}', '".intval($_POST['dnum'])."', '{$_POST['country']}')
        	";

        	mysql_query($sql);
        	header ("location: index.php");
        	break;

	case "del":
        	if (!isset($_GET['id']))
            		break;

		$sql = "DELETE FROM `files` WHERE `id`='{$_GET['id']}'";
        	mysql_query($sql);
        	header ("location: index.php");
        	break;
	}

	if (isset($_POST['opt']))
	{
		if (!isset($_POST['opt']['spoof_ip']))
			$_POST['opt']['spoof_ip'] = 0;

		foreach (array_keys($_POST['opt']) as $k)
			mysql_query("REPLACE INTO `opt` (`name`, `value`) VALUES ('$k', '{$_POST['opt'][$k]}')");

		header("location: index.php");
	}

	$bopt = array();

	$r = mysql_query("SELECT * FROM `opt`");
	while ($f = mysql_fetch_array($r))
		$bopt[$f['name']] = $f['value'];

?>
</code>
