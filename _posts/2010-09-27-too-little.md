---
layout: post
title: Too Little
tags:
- All
- Code Snippet
status: publish
type: post
published: true
meta:
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_notify_twitter: 'yes'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/9Hjwpg
  _sexybookmarks_permaHash: fda39ca78db4fbc1721cc9ce12fe7cd8
---
<blockquote><strong>Any supervisor worth his salt would rather deal with people who attempt too much than with those who try too little</strong><strong>.
</strong> <strong> -Lee Iacocca</strong></blockquote>

```php
<?php
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
echo "unknown messId:{$_GET['messId']}";
}
}

if(isset($_GET['bounceGetStr']) &amp;&amp;  $_GET['bounceGetStr']){
echo "&lt;div id='bouncedGetStr'&gt;{$_SERVER["QUERY_STRING"]}&lt;/div&gt;";
}

if(isset($_GET['message']) &amp;&amp; $_GET['message']){
echo $_GET['message'];
}

usleep($delay);

?&gt;
```
