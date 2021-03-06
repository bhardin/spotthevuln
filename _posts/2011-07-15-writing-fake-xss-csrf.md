---
layout: post
title: Writing - Fake XSS + CSRF
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
__Affected Software:__ EOF-0x01 Command and Control

__Fixed in Version:__  ?

__Issue Type:__ XSS and XSRF

Original Code: <a href="http://spotthevuln.com/2011/07/writing/">Found Here</a>
## Details
This week, we had a couple of bugs here affecting EOF-0x01 Command and Control. A red herring is the use of echo($_POST['pw']); to build HTML markup. Upon first glance, this seems like a straight forward XSS bug. This issue is mitigated by the fact that $_POST['pw'] is only displayed if it is equal to $botpw (whose default value happens to be 'bla') . So unless the botmaster has an XSS payload for their password, this one is going to be really difficult to exploit. The other interesting part is the if statements that look at $_POST['action']. If the user has provided the correct $_POST['pw'] and also provides a $_POST['action'] of 2 or 3, DeleteCommandsFromQueue() and EditCommandForBot() will be executed respectively. Developers (even malware developers) should be wary of allowing Create, Update, or Delete operations without defending against cross site request forgery. These functions are not protected.

## Vulnerable Code
<code lang="PHP" highlight="63,67,71,83,88">
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<title> </title>
<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1">

<?php
include("./config.php");
include("./functions.php");

$query = $_SERVER['QUERY_STRING'];
parse_str($query);

ConnectToDB($server, $user, $pw, $dbname);
?>

<style type="text/css">
<!--
@import url("./style.css");
-->
</style>

<script>
<!--
function setfocus()
{
	document.form1.cmd.focus();
	document.form1.logfield.scrollTop = '9999';
}
-->
</script>

</head>

<body onload="setfocus()">
<?php
if($_POST['pw']!=$botpw)
{
?>
<table width="242" border="0" cellpadding="0" cellspacing="0" bgcolor="#D0EAD2" class="tableborder">
  <!--DWLayoutTable-->
  <tr>
    <td width="239" height="44" valign="top"><form action="./control.php" method="post" name="login" id="login">
        Password:<br>
        <input name="pw" type="password" id="pw">
        <input name="login" type="submit" id="login" value="Login">
    </form></td>
  </tr>
</table>
<?php
}
else
{
?>
<table width="516" border="0" cellpadding="0" cellspacing="0" bgcolor="#D5E1F0" class="tableborder">
        <!--DWLayoutTable-->
        <tr>
          <td width="78" height="43" valign="middle"><form action="./control.php" method="post" name="logout" id="logout">
              <input name="logout" type="submit" id="logout" value="Logout">
          </form></td>
          <td width="143" valign="middle"><form action="./control.php" method="post" name="command" id="command">
              <input name="command" type="submit" id="command" value="Command center">
              <input name="pw" type="hidden" id="pw" value="<?php echo($_POST['pw']); ?>">
          </form></td>
		  <td width="193" valign="middle"><form action="./control.php" method="post" name="queue" id="queue">
              <input name="queue" type="submit" id="queue" value="Manage commandqueue">
              <input name="pw" type="hidden" id="pw" value="<?php echo($_POST['pw']); ?>">
          </form></td>
          <td width="101" valign="middle"><form action="./control.php" method="post" name="logdel" id="logdel">
              <input name="logdel" type="submit" id="logdel" value="Delete log">
              <input name="pw" type="hidden" id="pw" value="<?php echo($_POST['pw']); ?>">
          </form></td>
        </tr>
</table>
<?php
if(isset($_POST['queue']))
{

if(isset($_POST['action']))
{
	if($_POST['action']==2)
	{
		DeleteCommandsFromQueue();
	}

	if($_POST['action']==4)
	{
		EditCommandForBot();
	}
}

if($_POST['action']!=3)
{
?>
<br>
<form action="./control.php" method="post" name="form1" id="form1">
<table width="648" border="0" cellpadding="0" cellspacing="0" bgcolor="#F2ECD7" class="tableborder">
  <!--DWLayoutTable-->
  <tr>
    <td height="486" colspan="2" valign="top">Bot:<br>
          <select name="botselect" id="botselect">
		  <?php
		  ShowAllBotsCmdList();
		  ?>
        </select>
</code>
