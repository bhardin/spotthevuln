---
layout: post
title: Temptation - Cross-Site Scripting
tags:
- Cross-Site Scripting (XSS)
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  _sexybookmarks_shortUrl: http://bit.ly/7UplCm
  _sexybookmarks_permaHash: 59ee31f7da5ab5a7063f8d4e3d7864bd
---
## Details
__Affected Software:__ Joomla

__Fixed in Version:__  1.x

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Temptation" href="http://spotthevuln.com/2010/01/temptation/" target="_blank">Found Here</a>
## Description
Classic XSS on Joomla. This particular XSS isn't anything really special, but all the references to SQL and databases surrounding the vulnerable code could have thrown some people off during code review. In this code $this_page is tainted with attacker controlled data in the two lines below:
<blockquote>$this_page = (!empty($_SERVER['PHP_SELF'])) ? $_SERVER['PHP_SELF'] : $_ENV['PHP_SELF'];

$this_page .= '&amp;' . ((!empty($_SERVER['QUERY_STRING'])) ? $_SERVER['QUERY_STRING'] : $_ENV['QUERY_STRING']);</blockquote>
$this_page is then used to build an HTML string. The Joomla developers fixed this vulnerability by simply calling htmlspecialchars() before echoing $this_page to the user.
## Developer's Solution
```diff

function sql_freeresult($query_id = false)
{
if (!$query_id)
{
$query_id = $this-&gt;query_result;
}

return ($query_id) ? @mysql_free_result($query_id) : false;

}


function sql_escape($msg)
{

return mysql_escape_string(stripslashes($msg));

}



function sql_error($sql = '')
{

$result = array(

'message' =&gt; @mysql_error(),
'code' =&gt; @mysql_errno()

);


if (!$this-&gt;return_on_error)
{

if ($this-&gt;transaction)
{

$this-&gt;sql_transaction('rollback');

}

$this_page = (!empty($_SERVER['PHP_SELF'])) ? $_SERVER['PHP_SELF'] : $_ENV['PHP_SELF'];
$this_page .= '&amp;' . ((!empty($_SERVER['QUERY_STRING'])) ? $_SERVER['QUERY_STRING'] : $_ENV['QUERY_STRING']);

-$message = '&lt;u&gt;SQL ERROR&lt;/u&gt; [ ' . SQL_LAYER . ' ]&lt;br /&gt;&lt;br /&gt;' . @mysql_error() . '&lt;br /&gt;&lt;br /&gt;&lt;u&gt;CALLING PAGE&lt;/u&gt;&lt;br /&gt;&lt;br /&gt;'  . $this_page . (($sql != '') ? '&lt;br /&gt;&lt;br /&gt;&lt;u&gt;SQL&lt;/u&gt;&lt;br /&gt;&lt;br /&gt;' . $sql : '') . '&lt;br /&gt;';
+$message = '&lt;u&gt;SQL ERROR&lt;/u&gt; [ ' . SQL_LAYER . ' ]&lt;br /&gt;&lt;br /&gt;' . @mysql_error() . '&lt;br /&gt;&lt;br /&gt;&lt;u&gt;CALLING PAGE&lt;/u&gt;&lt;br /&gt;&lt;br /&gt;'  . htmlspecialchars($this_page) . (($sql != '') ? '&lt;br /&gt;&lt;br /&gt;&lt;u&gt;SQL&lt;/u&gt;&lt;br /&gt;&lt;br /&gt;' . $sql : '') . '&lt;br /&gt;';
trigger_error($message, E_USER_ERROR);

}


return $result;

}


} // class sql_db


} // if ... define


?&gt;

```
