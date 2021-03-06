---
layout: post
title: Action - Defense in Depth
tags:
- Defense In Depth
- PHP
- PixelPost
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ PixelPost

__Fixed in Version:__  ?

__Issue Type:__ Insecure password reset functionality

Original Code: <a href="http://spotthevuln.com/2011/05/action/">Found Here</a>
## Details
This week's bug is more of a design issue as opposed to an implementation issue. I actually first heard about this code from SkullSecurity's excellent articles on <a href="http://www.skullsecurity.org/blog/2011/hacking-crappy-password-resets-part-2">"Hacking Crappy Password Resets"</a> articles published in late March. SkullSecurity does an excellent job of explaining that line 31(the line that does the actual password generation) is full of bad security design. First, the password reset code is using the MD5() function in PHP. MD5() takes a string and returns a MD5 hash of that string. In this password reset code, we see that we are hashing the value of: 'time' . rand(1, 16000). SkullSecurity astutely points out that the password reset code is using the string 'time', NOT the function time() which would return the number of seconds since epoch. The string 'time' has a random number between 1 and 16000 appended to it and is passed to MD5(). Assuming rand() generates a truly random number between 1 and 16000, we only have 16000 possible combinations for the newly reset password. Unfortunately, in many circumstances, rand() doesn't generate a truly random number. Many times, rand() will use system time as a seed value and produce a number that is predictable. In many cases, it may be possible to predict or leak system time for the machine generating the random password. Stefan Esser wrote an <a href="http://www.suspekt.org/2008/08/17/mt_srand-and-not-so-random-numbers/">excellent article</a> on the challenges of generating random numbers in PHP which covers many of these scenarios.

So what should the author do?  Change 'time' to time()?  Even if the author changed the string 'time' to the function time(), time() in PHP only returns seconds since epoch time. If an attacker can leak system time, they'll still be able to predict the password reset value...


## Developer's Solution
[sourcecode language="php" highlight="31"]
&lt;?php

...snip...
// forgot password?
if(isset($_GET['x']) &amp;&amp; $_GET['x']=='passreminder')
{
echo '&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Transitional//EN&quot;
  &quot;http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&quot;&gt;
	&lt;html&gt;
	&lt;head&gt;&lt;title&gt;$admin_lang_pw_title&lt;/title&gt;
	&lt;meta http-equiv=&quot;Content-Type&quot; content=&quot;text/html;charset=utf-8&quot; /&gt;&lt;/head&gt;
	&lt;body&gt;
	&lt;p style=&quot;border:solid 2px;padding:5px;color:red;font-weight:bold;font-size:11px;margin-left:auto;margin-right:auto;margin-top:10%;font-family:verdana,arial,sans-serif;text-align:center;&quot;&gt;';

	if ($cfgrow['admin']!= $_POST['user'])
	{
		echo &quot;&lt;span class=\&quot;confirm\&quot;&gt;$admin_lang_pw_wronguser&lt;/span&gt;&lt;br /&gt;&quot;;
		echo &quot;&lt;br /&gt;&lt;a href='index.php'&gt;$admin_lang_pw_back&lt;/a&gt;&lt;/body&gt;&lt;/html&gt;&quot;;
		die();
	}
	if ($cfgrow['email']== &quot;&quot;)
	{
		echo &quot;&lt;span class=\&quot;confirm\&quot;&gt;$admin_lang_pw_noemail&lt;/span&gt;&lt;br /&gt;&quot;;
		echo &quot;&lt;br /&gt;&lt;a href='index.php' &gt; $admin_lang_pw_back &lt;/a&gt;&lt;/body&gt;&lt;/html&gt;&quot;;
		die();
	}

	if (strtolower($cfgrow['email'])==strtolower($_POST['reminderemail']))
	{
		// generate a random new pass
		$user_pass = substr( MD5('time' . rand(1, 16000)), 0, 6);
		$query = &quot;update &quot;.$pixelpost_db_prefix.&quot;config set password=MD5('$user_pass') where admin='&quot;.$cfgrow['admin'].&quot;'&quot;;
		if(mysql_query($query))
		{
			$subject = &quot;$admin_lang_pw_subject&quot;;
			$body  = &quot;$admin_lang_pw_text_1 \n\n&quot;;
			$body .= &quot;$admin_lang_pw_usertext &quot;.$cfgrow['admin'].&quot; \n&quot;;
			$body .= &quot;$admin_lang_pw_mailtext &quot;.$cfgrow['email'].&quot; \n\n&quot;;
			$body .= &quot;$admin_lang_pw_newpw $user_pass&quot;;
			$body .= &quot;\n\n$admin_lang_pw_text_7&quot;.$cfgrow['siteurl'].&quot;admin $admin_lang_pw_text_8&quot;;

			$headers = &quot;Content-type: text/plain; charset=UTF-8\n&quot;;
			$headers .= &quot;$admin_lang_pw_text_2 &lt;&quot;.$cfgrow['email'].&quot;&gt;\n&quot;;

			$recipient_email = $cfgrow['email'];
			if (mail($recipient_email,$subject,$body,$headers))
				{echo &quot;$admin_lang_pw_text_3&quot; .$cfgrow['email'];}
			else { echo &quot;$admin_lang_pw_text_3&quot;;}
			echo &quot;&lt;br /&gt;&lt;a href='index.php' &gt; $admin_lang_pw_back &lt;/a&gt;&lt;/body&gt;&lt;/html&gt;&quot;;
			die();
		}
		else
		{
			$dberror = mysql_error();
			echo &quot;$admin_lang_pw_text_5 &quot; .$dberror .&quot;$admin_lang_pw_text_5 &quot; ;
				echo &quot;&lt;br /&gt;&lt;a href='index.php' &gt; $admin_lang_pw_back &lt;/a&gt;&lt;/body&gt;&lt;/html&gt;&quot;;
			die();
		}
	}
	else
	{
		echo &quot;&lt;span class=\&quot;confirm\&quot;&gt;$admin_lang_pw_notsent&lt;/span&gt;&lt;br /&gt;&quot;;
		echo &quot;&lt;br /&gt;&lt;a href='index.php'&gt; $admin_lang_pw_back &lt;/a&gt;&lt;/body&gt;&lt;/html&gt;&quot;;
		die();
	}// end else (strtolower($cfgrow['email'])==strtolower($_POST['reminderemail']) &amp; $cfgrow['email']!= &quot;&quot;)

} // end if($_GET['x']=='passreminder')
?&gt;
[/sourcecode]
