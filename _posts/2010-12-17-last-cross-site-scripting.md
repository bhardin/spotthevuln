---
layout: post
title: Last - Cross-Site Scripting
tags:
- All
- apache status
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- echoes
- Encode
- PHP
- rigorous security
- Solution
- web application
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/eBRsih
  _sexybookmarks_permaHash: 9b9c6b0100c1e134a761d785d80b6617
---
<h3>Details</h3>
__Affected Software:__ AskApache Password Protector

__Fixed in Version:__  4.0.1

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Last" href="http://spotthevuln.com/2010/12/last/" target="_blank">Found    Here</a>
<h3>Description</h3>
Upon first glance, we see that the vulnerable code sample comes from an error page of some sort. Error pages are often overlooked when it comes to security (or even general QA). Make sure you put your error pages through the same rigorous security process as you would any other page. The Same Origin Policy won't distinguish between a forgotten error page and the highly trafficked portal page of your web application. A vulnerability on an error page can have the same devastating effect as a vulnerability on the main portal page. Looking at this bug, we see that the error page is a bit too helpful and echoes back all the information contained in the $_SERVER superglobal. Unfortunately, this superglobal contains all sorts of user/attacker controlled information, resulting in XSS. In this fix, the developers wisely removed the vulnerable line entirely.
<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="44"]
&lt;?php
ob_start();
//http://www.askapache.com/htaccess/apache-status-code-headers-errordocument.html
/*
array( floor($code / 100)
 1=&gt;'INFO', 2=&gt;'SUCCESS', 3=&gt;'REDIRECT', 4|5=&gt;'ERROR', 4=&gt;'CLIENT_ERROR', 5=&gt;'SERVER_ERROR', 'VALID_RESPONSE');
*/

... &lt;SNIP&gt; ...

if (isset($_SERVER['REDIRECT_STATUS'])) $err_code = $_SERVER['REDIRECT_STATUS'];

$err_req_meth = $_SERVER['REQUEST_METHOD'];
$err_req = htmlentities(strip_tags($_SERVER['REQUEST_URI']));
$err_phrase = $err_status_codes[$err_code][0];
$err_body = str_replace(
 array('INTERROR', 'THEREQUESTURI', 'THEREQMETH'),
 array('The server encountered an internal error or misconfiguration and was unable to complete your request.',$err_req, $err_req_meth),$err_status_codes[$err_code][1]);

@header(&quot;HTTP/1.1 $err_code $err_phrase&quot;, 1);
@header(&quot;Status: $err_code $err_phrase&quot;, 1);

//400 || 408 || 413 || 414 || 500 || 503 || 501
//@header(&quot;Connection: close&quot;, 1);

if ( $err_code=='400'||$err_code=='403'||$err_code=='405'||$err_code[0]=='5'){
 @header(&quot;Connection: close&quot;, 1);
 if ($err_code == '405') @header('Allow: GET,HEAD,POST,OPTIONS,TRACE');
 echo &quot;&lt;!DOCTYPE HTML PUBLIC \&quot;-//IETF//DTD HTML 2.0//EN\&quot;&gt;\n&lt;html&gt;\n&lt;head&gt;\n&lt;title&gt;{$err_code} {$err_phrase}&lt;/title&gt;\n&lt;h1&gt;{$err_phrase}&lt;/h1&gt;\n&lt;p&gt;{$err_body}&lt;br&gt;\n&lt;/p&gt;\n&lt;/body&gt;&lt;/html&gt;&quot;;
} else echo '&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Strict//EN&quot;
       &quot;http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd&quot;&gt;
&lt;html xml:lang=&quot;en&quot; lang=&quot;en&quot;&gt;
&lt;head&gt;
  &lt;title&gt;'.$err_code.' '.$err_phrase.'&lt;/title&gt;
  &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;'.$err_code.' '.$err_phrase.'&lt;/h1&gt;
&lt;hr /&gt;
&lt;p&gt;
'.$err_body.'&lt;br /&gt;
&lt;/p&gt;
&lt;pre&gt;
-'.print_r($_SERVER,1).'
&lt;/pre&gt;
  &lt;/body&gt;
&lt;/html&gt;';
?&gt;
[/sourcecode]
