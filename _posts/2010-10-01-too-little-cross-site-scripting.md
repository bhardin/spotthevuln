---
layout: post
title: Too Little - Cross Site Scripting
tags:
- All
- attacker
- bugs
- code samples
- Cross-Site Scripting (XSS)
- debugging
- DoJo
- Dojo
- Encode
- error message
- PHP
- production environment
- querystring
- Solution
- switch cases
- test
- testing and debugging
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/dwAy57
  _sexybookmarks_permaHash: c6f9ad97053c90cbac7c12130197e976
---
## Details
__Affected Software:__ Dojox

__Fixed in Version:__  1.4.1

__Issue Type:__ XSS

Original Code: <a title="Too Little" href="http://spotthevuln.com/2010/09/too-little/" target="_blank">Found    Here</a>
## Description
The code sample presented this week have a few XSS vulnerabilities.  The first XSS bug is caused when $_GET['messId'] doesn't match any of the switch cases.  The default behavior is to throw a friendly error message indicating the messId is unknown along with the value passed via the querystring.  Unfortunately, the error message can also contain attacker controlled HTML/SCRIPT resulting in XSS.  The two other XSS bugs fixed by the developers in this patch are classic XSS.  Both issues take user/attacker controlled variables and display the variables in markup without sanitization/encoding.

Although the XSS bugs are fairly straight forward, what I find interesting in this example is why this page is here in the first place.  The code don't seem to provide any useful functionality (other than to provide a place for attackers to abuse XSS).  If you were a security engineer assigned to audit this page, it might be a good idea to ask WHY this page exists in the first place.  It turns out that this page is indeed a test/debugging page that is included with dojox, offering no functionality intended for production users.  Think long and hard before exposing test and debug functionality in your production environment.  Check production systems for example/testing/debugging functionality and disable/remove this functionality if possible.  Code developed for testing and debugging rarely undergoes the scrutiny of production code and will like contain security issues.
<h2>Developers Solution</h2>
[cce lang="diff"]
&lt;?php
// this just bounces a message as a response, and optionally emulates network latency.

// default delay is 0 sec, to change:
// getResponse.php?delay=[Int milliseconds]

// to change the message returned
// getResponse.php?mess=whatever%20string%20you%20want.

// to select a predefined message
// getResponse.php?messId=0

error_reporting(E_ALL ^ E_NOTICE);

$delay = 1; // 1 micro second to avoid zero division in messId 2
if(isset($_GET['delay']) &amp;&amp; is_numeric($_GET['delay'])){
$delay = (intval($_GET['delay']) * 1000);
}

if(isset($_GET['messId']) &amp;&amp; is_numeric($_GET['messId'])){
switch($_GET['messId']){
case 0:
echo "&lt;h3&gt;WARNING This should NEVER be seen, delayed by 2 sec!&lt;/h3&gt;";
$delay = 2;
break;
case 1:
echo "&lt;div dojotype='dijit.TestWidget'&gt;Testing attr('href', ...)&lt;/div&gt;";
break;
case 2:
echo "&lt;div dojotype='dijit.TestWidget'&gt;Delayed attr('href', ...) test&lt;/div&gt;
&lt;div dojotype='dijit.TestWidget'&gt;Delayed by " . ($delay/1000000) . " sec.&lt;/div&gt;";
break;
case 3:
echo "IT WAS the best of times, it was the worst of times, it was the age of wisdom, it was the age of foolishness, it was the epoch of belief, it was the epoch of incredulity, it was the season of Light, it was the season of Darkness, it was the spring of hope, it was the winter of despair, we had everything before us, we had nothing before us, we were all going direct to Heaven, we were all going direct the other way -- in short, the period was so far like the present period, that some of its noisiest authorities insisted on its being received, for good or for evil, in the superlative degree of comparison only";
break;
case 4:
echo "There were a king with a large jaw and a queen with a plain face, on the throne of England; there were a king with a large jaw and a queen with a fair face, on the throne of France. In both countries it was clearer than crystal to the lords of the State preserves of loaves and fishes, that things in general were settled for ever.";
break;
case 5:
echo "It was the year of Our Lord one thousand seven hundred and seventy- five. Spiritual revelations were conceded to England at that favoured period, as at this. Mrs. Southcott had recently attained her five-and- twentieth blessed birthday, of whom a prophetic private in the Life Guards had heralded the sublime appearance by announcing that arrangements were made for the swallowing up of London and Westminster. Even the Cock-lane ghost had been laid only a round dozen of years, after rapping out its messages, as the spirits of this very year last past (supernaturally deficient in originality) rapped out theirs. Mere messages in the earthly order of events had lately come to the English Crown and People, from a congress of British subjects in America:";
break;
default:
-echo "unknown messId:{$_GET['messId']}";
+echo "unknown messId:". htmlentities($_GET['messId']);
}
}

if(isset($_GET['bounceGetStr']) &amp;&amp;  $_GET['bounceGetStr']){
-echo "&lt;div id='bouncedGetStr'&gt;{$_SERVER["QUERY_STRING"]}&lt;/div&gt;";
+echo "&lt;div id='bouncedGetStr'&gt;".htmlentities($_SERVER["QUERY_STRING"])."&lt;/div&gt;";
}

if(isset($_GET['message']) &amp;&amp; $_GET['message']){
-echo $_GET['message'];
+echo htmlentities($_GET['message']);
}

usleep($delay);

?&gt;

[/cce] 
