---
layout: post
title: Time's Fun - XSS
tags:
- Cross-Site Scripting (XSS)
- Joomla
- PHP
- Solution
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 83dafcca8ee3ace146cbf04b0a2f02b4
  _sexybookmarks_shortUrl: http://bit.ly/6DnsOm
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  _headspace_description: XSS in Joomla versions < 1.0.14
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Joomla

__Fixed in Version:__  1.0.14

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Time's Fun" href="http://spotthevuln.com/2009/12/times-fun/" target="_blank">Found Here</a>
## Description
<div id="_mcePaste">
<p id="_mcePaste">This is a very subtle bug in Joomla's XSS filtering. Joomla establishes a blacklist of HTML tags and attributes that are not allowed. Blacklists always seem to be the easiest way to go... but rarely do they standup in the long run. We'll touch on blacklists in a bit, but for now on to the bug!  In this case, the attacker can provide a HTML tag and a HTML tag attribute (the attribute is optional). Knowing the dangers of giving users the option to provide arbitrary HTML tags and attributes, the Joomla team devised a blacklist of "bad" HTML tags and attributes. If the HTML tags and attributes provided by the user happens to match one of the tags or attributes listed in the blacklist, then we enter a failure condition. Knowing some of the trickery used by  web hackers, the Joomla team first converts the user controlled attribute to lower before checking to see if the provided attribute is in the attribute blacklist.</p>

<blockquote>
<div id="_mcePaste">(in_array(<strong><span style="color: #ff0000;">strtolower</span></strong>($attrSubSet[0]), $this-&gt;attrBlacklist)</div></blockquote>
<p id="_mcePaste">As a "catch all" it seems that the Joomla team wanted to disallow any HTML attributes that began with the letters "on". This should prevent an attacker from injecting an attribute such as "onload=javascript:payload()" or "onblur=javascript:payload()". This was done by taking a substr() of the provided attribute value and checking to see if the substr() contained the "on" characters.</p>

<blockquote>
<div>(substr($attrSubSet[0], 0, 2) == 'on')</div></blockquote>
<p id="_mcePaste">Unfortunately for the Joomla team HTML (for the most part) supports case insensitivity for HTML attributes. So while the check would stop an attacker from injecting an onload=javascript:payload() XSS payload, it would allow OnLoAd=javascript:payload(). The Joomla developers fixed this issue by forcing the $attrSubSet[] to lower before doing the comparison.</p>

<blockquote>
<div>(substr(<span style="color: #ff0000;"><strong>strtolower</strong></span>($attrSubSet[0]), 0, 2) == 'on'))))</div></blockquote>
<p id="_mcePaste">Now, onto the blacklist... To put it simply, blacklists are tough. Even if you somehow manage to get the blacklist correct today, that doesn't mean someone will change the rules and render your blacklist ineffective tomorrow. Looking at the blacklist Joomla HTML tag blacklist in this particular code sample, it seems the blacklist is geared towards HTML4. HTML5 has added new HTML tags which don't appear to be covered by the blacklist in this example... I wonder if the Joomla devs have updated their blacklist. Even if the blacklist was updated, it will have to be updated again when new HTML tags are added for the next version of HTML... let's hope no one forgets :)</p>

</div>
## Developer's Solution
```diff

var $xssAuto; // default = 1
var $tagBlacklist = array ('applet', 'body', 'bgsound', 'base', 'basefont', 'embed', 'frame', 'frameset', 'head', 'html', 'id', 'iframe', 'ilayer', 'layer', 'link', 'meta', 'name', 'object', 'script', 'style', 'title', 'xml');
var $attrBlacklist = array ('action', 'background', 'codebase', 'dynsrc', 'lowsrc'); // also will strip ALL event handlers

...

/**
* Internal method to strip a tag of certain attributes
*
* @access    protected
* @param    array    $attrSet    Array of attribute pairs to filter
* @return    array    $newSet        Filtered array of attribute pairs
*/
function filterAttr($attrSet)
{
/*
* Initialize variables
*/
$newSet = array ();

/*
* Iterate through attribute pairs
*/
for ($i = 0; $i &lt; count($attrSet); $i ++)
{
/*
* Skip blank spaces
*/
if (!$attrSet[$i])
{
continue;
}

/*
* Split into name/value pairs
*/
$attrSubSet = explode('=', trim($attrSet[$i]), 2);
list ($attrSubSet[0]) = explode(' ', $attrSubSet[0]);

/*
* Remove all "non-regular" attribute names
* AND blacklisted attributes
*/
-if ((!eregi("^[a-z]*$", $attrSubSet[0])) || (($this-&gt;xssAuto) &amp;&amp; ((in_array(strtolower($attrSubSet[0]), $this-&gt;attrBlacklist)) || (substr($attrSubSet[0], 0, 2) == 'on'))))
+if ((!eregi("^[a-z]*$", $attrSubSet[0])) || (($this-&gt;xssAuto) &amp;&amp; ((in_array(strtolower($attrSubSet[0]), $this-&gt;attrBlacklist)) || (substr(strtolower($attrSubSet[0]), 0, 2) == 'on'))))
{
continue;
}

/*
* XSS attribute value filtering
*/
if ($attrSubSet[1])
{
// strips unicode, hex, etc
$attrSubSet[1] = str_replace('&amp;#', '', $attrSubSet[1]);
// strip normal newline within attr value
$attrSubSet[1] = preg_replace('/\s+/', '', $attrSubSet[1]);
// strip double quotes
$attrSubSet[1] = str_replace('"', '', $attrSubSet[1]);
// [requested feature] convert single quotes from either side to doubles (Single quotes shouldn't be used to pad attr value)
if ((substr($attrSubSet[1], 0, 1) == "'") &amp;&amp; (substr($attrSubSet[1], (strlen($attrSubSet[1]) - 1), 1) == "'"))
{
$attrSubSet[1] = substr($attrSubSet[1], 1, (strlen($attrSubSet[1]) - 2));
}
// strip slashes
$attrSubSet[1] = stripslashes($attrSubSet[1]);
}

/*
* Autostrip script tags
*/
if (InputFilter :: badAttributeValue($attrSubSet))
{
continue;
}

/*
* Is our attribute in the user input array?
*/
$attrFound = in_array(strtolower($attrSubSet[0]), $this-&gt;attrArray);

/*
* If the tag is allowed lets keep it
*/
if ((!$attrFound &amp;&amp; $this-&gt;attrMethod) || ($attrFound &amp;&amp; !$this-&gt;attrMethod))
{
/*
* Does the attribute have a value?
*/
if ($attrSubSet[1])
{
$newSet[] = $attrSubSet[0].'="'.$attrSubSet[1].'"';
}
elseif ($attrSubSet[1] == "0")
{
/*
* Special Case
* Is the value 0?
*/
$newSet[] = $attrSubSet[0].'="0"';
} else
{
$newSet[] = $attrSubSet[0].'="'.$attrSubSet[0].'"';
}
}
}
return $newSet;
}

```
