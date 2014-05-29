---
layout: post
title: Expect - Cross Site Scripting
tags:
- All
- bugs
- Cross-Site Scripting (XSS)
- default case
- DoJo
- Encode
- html encoding
- PHP
- querystring
- Solution
- switch statement
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/dyZoQ3
  _sexybookmarks_permaHash: d0921adfede00884bb3a3144dd5aa7ba
---
## Details
__Affected Software:__ DojoX

__Fixed in Version:__  1.4.1

__Issue Type:__ Cross Site Scripting (XSS)

Original Code: <a title="Expect" href="http://spotthevuln.com/2010/10/expect/" target="_blank">Found    Here</a>
## Description
This week's bug was an easy one with a straightforward fix. The vulnerable code sample is basically a single switch statement. Most of the cases don't do anything interesting (from a security standpoint) however, the default case at the end of the switch statement leads to a vulnerable condition. If all of the case statements fail, the application throws a useful error message echoing back a value directly from the query string. The DojoX developers addressed the vulnerability by html encoding the value before echoing it back to the user. Simple bug, simple fix. I hope all the readers have a good weekend and a Happy Halloween!
## Developers Solution
[sourcecode language="diff"]
&lt;?php
	// this file is just a bouncer for ContentPane.html test
	error_reporting(E_ALL ^ E_NOTICE);

	if(isset($_GET['mode'])){
		switch($_GET['mode']){
			case 'htmlPaths':
				echo &quot;&lt;img src='../images/testImage.gif' id='imgTest'/&gt;
					&lt;div id='inlineStyleTest' style='width:188px;height:125px;background-image:url(../images/testImage.gif)'&gt;&lt;/div&gt;
					&lt;style&gt;@import 'getResponse.php?mode=importCss';&lt;/style&gt;
					&lt;link type='text/css' rel='stylesheet' href='getResponse.php?mode=linkCss'&gt;
					&lt;div id='importCssTest'&gt;&lt;/div&gt;
					&lt;div id='linkCssTest'&gt;&lt;/div&gt;
					&lt;div id='importMediaTest'&gt;&lt;/div&gt;
					&lt;div id='linkMediaTest'&gt;&lt;/div&gt;
					&lt;!-- these may download but not render --&gt;
					&lt;style media='print'&gt;@import 'getResponse.php?mode=importMediaPrint';&lt;/style&gt;
					&lt;link media='print' type='text/css' rel='stylesheet' href='getResponse.php?mode=linkMediaPrint'&gt;
					&quot;;
				break;

			case 'importCss':
				header('Content-type: text/css; charset=utf-8');
				echo &quot;#importMediaTest {
					margin: 4px;
					border: 1px dashed red;
					width: 200px;
					height: 200px;
				}
				#importCssTest {
						margin: 4px;
						border: 1px solid blue;
						width: 100px;
						height: 100px;
					}&quot;;
				break;

			case 'linkCss':
				header('Content-type: text/css; charset=utf-8');
				echo &quot;#linkMediaTest {
					margin: 4px;
					border: 2px dashed red;
					width: 200px;
					height: 200px;
				}
				#linkCssTest {
					margin: 4px;
					border: 2px dashed red;
					width: 100px;
					height: 100px;
				}&quot;;
				break;

			case 'importMediaPrint': // may download but not render
				header('Content-type: text/css; charset=utf-8');
				echo &quot;#importMediaTest {
					margin: 10px;
					border: 5px dashed gray;
					width: 100px;
					height: 100px;
				}&quot;;
				break;

			case 'linkMediaPrint': // may download but not render
				header('Content-type: text/css; charset=utf-8');
				echo &quot;#linkMediaTest {
					margin: 10px;
					border: 5px dashed gray;
					width: 100px;
					height: 100px;
				}&quot;;
				break;

			case 'remoteJsTrue':
				header('Content-type: text/javascript; charset=utf-8');
				echo &quot;unTypedVarInDocScope = true;&quot;;
				break;

			case 'remoteJsFalse':
				header('Content-type: text/javascript; charset=utf-8');
				echo &quot;unTypedVarInDocScope = false;&quot;;
				break;
			case 'entityChars':
				header('Content-type: text/css; charset=utf-8');
				if($_GET['entityEscaped'] == null){
					print(&quot;var div = document.createElement(\&quot;div\&quot;); document.body.appendChild(div); div.innerHTML = \&quot;&lt;div id=\\\&quot;should_not_be_here2\\\&quot;&gt;&lt;/div&gt;\&quot;; window.__remotePaneLoaded2 = true;&quot; );
				}else{
					print(&quot;window.__remotePaneLoaded2 = true;&quot;);
				}
				break;
			default:
-				echo &quot;unkown mode {$_GET['mode']}&quot;;
+				echo &quot;unkown mode {htmlentities($_GET['mode'])}&quot;;
		}
	}
?&gt;

[/sourcecode]
