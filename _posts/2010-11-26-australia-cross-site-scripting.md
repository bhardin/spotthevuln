---
layout: post
title: Australia - Cross Site Scripting
tags:
- All
- code execution
- cross site scripting
- Cross-Site Scripting (XSS)
- echo system
- FreePBX
- freepbx
- html markup
- pbx system
- PHP
- php code
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/fTsAPp
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: aa0af7653b792551579a4dff518a0591
---
## Details
__Affected Software:__ FreePBX

__Fixed in Version:__  2.5

__Issue Type:__ Cross Site Scripting (XSS)

Original Code: <a title="Australia" href="http://spotthevuln.com/2010/11/australia/" target="_blank">Found    Here</a>
## Description
One of the more weird XSS vulnerabilities I've seen :)
Here we see FreePBX using a potion of a log file in their HTML markup. Specifically, the PHP code uses the system() API to execute a command on the PBX system. The results of the system() command are printed to the HTML markup. In this case, FreePBX runs a tail on a log file displaying some of the entries contained within that log file. Looking at the vulnerable code sample, it is impossible to understand exactly what is contained in these log files, however it appears that HTML can exist in the log entries due to the sed command being run on the log file output:

<blockquote>| sed -e "s/$/&lt;br&gt;/"</blockquote>

The developers realized that the log files could contain other dangerous HTML elements and modified their sed command to try and filter those elements out. Maybe a better approach would be to use a proper encoding API?  Luckily, it doesn't seem like the attacker can control anything passed to system(), otherwise this would have been a code execution bug as opposed to just an XSS!

## Developers Solution
[sourcecode language="diff"]
&lt;?php

$display = $_REQUEST['display'];
$type = isset($_REQUEST['type']) ? $_REQUEST['type'] : 'tool';
$action = isset($_REQUEST['action']) ? $_REQUEST['action'] : '';

?&gt;
&lt;/div&gt;
&lt;div class=&quot;content&quot;&gt;
&lt;?php

switch($action) {
	case 'showlog':
?&gt;
		&lt;h2&gt;
			&lt;?php echo sprintf(_('%s - last 2000 lines'),$amp_conf['ASTLOGDIR'].&quot;/full&quot;) ?&gt;
		&lt;/h2&gt;
		&lt;a href=&quot;config.php?&lt;?php echo &quot;display=$display&amp;type=$type&amp;action=showlog&quot;?&gt;&quot;&gt;&lt;?php echo _(&quot;Redisplay Asterisk Full debug log (last 2000 lines)&quot;) ?&gt;&lt;/a&gt;&lt;br&gt;
		&lt;hr&gt;&lt;br&gt;
		&lt;?php
-		echo system ('tail --line=2000 '.$amp_conf['ASTLOGDIR'].'/full | sed -e &quot;s/$/&lt;br&gt;/&quot;');
+		system ('tail --line=2000 '.$amp_conf['ASTLOGDIR'].'/full | sed -e &quot;s,&lt;,\&amp;lt;,g;s,&gt;,\&amp;gt;,g;s/$/&lt;br&gt;/&quot;');
		break;

	default:
		echo &quot;&lt;h2&gt;&quot;._(&quot;Asterisk Log Files&quot;).&quot;&lt;/h2&gt;&quot;;
?&gt;
				&lt;a href=&quot;config.php?&lt;?php echo &quot;display=$display&amp;type=$type&amp;action=showlog&quot;?&gt;&quot;&gt;&lt;?php echo _(&quot;Display Asterisk Full debug log (last 2000 lines)&quot;) ?&gt;&lt;/a&gt;&lt;br&gt;
				&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;&lt;br&gt;
&lt;?php
    break;
}
?&gt;
&lt;/div&gt;
[/sourcecode]
