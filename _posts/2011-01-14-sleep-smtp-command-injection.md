---
layout: post
title: Sleep - SMTP Command Injection
tags:
- All
- carriage return
- Carriage Return/Line Feed (CRLF) Injection
- Encode
- PHP
- PunBB
- rfc 821
- smtp commands
- smtp message
- socket connection
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/hZlXUz
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: 818fc40e8ee6c7c139977e6261353369
---
<h3>Details</h3>
__Affected Software:__ PunBB

__Fixed in Version:__  1.3.2

__Issue Type:__ SMTP Command Injection

Original Code: <a title="Sleep" href="https://spotthevuln.com/2011/01/sleep/" target="_blank">Found    Here</a>
<h3>Description</h3>
Interesting bug here. In 2008, Stefan Esser reported a bug to the PunBB team which described a SMTP command injection vulnerability. If we look at the code below, we see that PunBB opens a socket connection to a SMTP host and passes various user/attacker controlled values to the SMTP server. Because of this setup, it is possible to craft a SMTP message that tricks the SMTP server into thinking the data provided for the message is completed, and executes any data that follows as SMTP commands. The attacker accomplished this by injecting Carriage Return and Line Feed characters following by a period character on a line by itself (as defined in RFC 821 - SMTP). The PunBB developers addressed this vulnerability by sanitizing CRLFs and period characters.

The Web Application Hackers Handbook (by Dafydd Stuttard) describes various forms of SMTP injection in a pretty comprehensive manner. If the PunBB developers used the test cases described by Dafydd in his book would have likely identified this vulnerability before shipping. Here's a <a href="http://my.safaribooksonline.com/book/networking/security/9780470170779/a-web-application-hacker-s-methodology/stuttard0779c20-sec1-0010">sample</a> from the Web Application Hackers Handbook that talks about SMTP injection (see section 8.2)


<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="9,10,11"]
&lt;?php
...snip...
function smtp_mail($to, $subject, $message, $headers = '')
{
	global $pun_config;

	$recipients = explode(',', $to);

+	// Sanitize the message
+	$message = str_replace(&quot;\r\n.&quot;, &quot;\r\n..&quot;, $message);
+	$message = (substr($message, 0, 1) == '.' ? '.'.$message : $message);

	// Are we using port 25 or a custom port?
	if (strpos($pun_config['o_smtp_host'], ':') !== false)
		list($smtp_host, $smtp_port) = explode(':', $pun_config['o_smtp_host']);
	else
	{
		$smtp_host = $pun_config['o_smtp_host'];
		$smtp_port = 25;
	}

	if (!($socket = fsockopen($smtp_host, $smtp_port, $errno, $errstr, 15)))
		error('Could not connect to smtp host &quot;'.$pun_config['o_smtp_host'].'&quot; ('.$errno.') ('.$errstr.')', __FILE__, __LINE__);

	server_parse($socket, '220');

	if ($pun_config['o_smtp_user'] != '' &amp;&amp; $pun_config['o_smtp_pass'] != '')
	{
		fwrite($socket, 'EHLO '.$smtp_host.&quot;\r\n&quot;);
		server_parse($socket, '250');

		fwrite($socket, 'AUTH LOGIN'.&quot;\r\n&quot;);
		server_parse($socket, '334');

		fwrite($socket, base64_encode($pun_config['o_smtp_user']).&quot;\r\n&quot;);
		server_parse($socket, '334');

		fwrite($socket, base64_encode($pun_config['o_smtp_pass']).&quot;\r\n&quot;);
		server_parse($socket, '235');
	}
	else
	{
		fwrite($socket, 'HELO '.$smtp_host.&quot;\r\n&quot;);
		server_parse($socket, '250');
	}

	fwrite($socket, 'MAIL FROM: &lt;'.$pun_config['o_webmaster_email'].'&gt;'.&quot;\r\n&quot;);
	server_parse($socket, '250');

	$to_header = 'To: ';

	@reset($recipients);
	while (list(, $email) = @each($recipients))
	{
		fwrite($socket, 'RCPT TO: &lt;'.$email.'&gt;'.&quot;\r\n&quot;);
		server_parse($socket, '250');

		$to_header .= '&lt;'.$email.'&gt;, ';
	}

	fwrite($socket, 'DATA'.&quot;\r\n&quot;);
	server_parse($socket, '354');

	fwrite($socket, 'Subject: '.$subject.&quot;\r\n&quot;.$to_header.&quot;\r\n&quot;.$headers.&quot;\r\n\r\n&quot;.$message.&quot;\r\n&quot;);

	fwrite($socket, '.'.&quot;\r\n&quot;);
	server_parse($socket, '250');

	fwrite($socket, 'QUIT'.&quot;\r\n&quot;);
	fclose($socket);

	return true;
}
[/sourcecode]
