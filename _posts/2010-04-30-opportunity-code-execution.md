---
layout: post
title: Opportunity - Code Execution
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '2'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/cn8cWW
  _sexybookmarks_permaHash: 627b72201a3c7fd7b3ae8ac5c02c1acc
---
## Details
__Affected Software:__ WordPress Plugin iBegin Share

__Fixed in Version:__  Rev 91762

__Issue Type:__ Code Execution

Original Code: <a title="Opportunity" href="http://spotthevuln.com/2010/04/opportunity/" target="_blank">Found Here</a>
## Description
<p class="MsoNormal" style="margin: 0in 0in 10pt;"><span style="font-family: Calibri; font-size: small;">In this specific example, we have a condition where the attacker has the ability to pass an arbitrary URL (with an arbitrary protocol handler) to the fopen() PHP API.<span style="mso-spacerun: yes;">  </span>By allowing the attacker to pass arbitrary URLs to the fopen() API, the developers have introduced an opportunity for several different types of exploits.<span style="mso-spacerun: yes;">  </span>Using this vulnerability, the attacker has the ability to fopen() any file on the local file system of the web server.<span style="mso-spacerun: yes;">  </span>Configuration and files holding sensitive information located on the web server would be a great target, especially if the contents of the files were eventually echoed back to the attacker.<span style="mso-spacerun: yes;">  </span>In addition to the information disclosure bug, some servers are configured to allow fopen() to actually execute PHP code as if it were an include.<span style="mso-spacerun: yes;">  </span>In addition to the information disclosure, this bug could also be used to pull of XSS and a variety of other exploits.</span></p>
<p class="MsoNormal" style="margin: 0in 0in 10pt;"><span style="font-family: Calibri; font-size: small;">The iBegin Share developers addressed the issue by forcing the URL provided to have the "HTTP/HTTPS" prefix...<span style="mso-spacerun: yes;">  </span>I wonder if this strategy is robust against the code execution attack</span>.</p>

## Developer's Solution
```diff
<div id="_mcePaste">

switch($db['type'])
{
   case 'mysql':
       $db_link = new MySQLdb($db['host'], $db['username'], $db['password'], false, $db['database']);
    break;
    case 'postgresql':
        $db_link = new PostgreSQL($db['host'], $db['username'], $db['password'], false, $db['database']);
    break;
}
unset($db);

// Set Header and cache expiration
$offset = 60 * 60 * 24 * 2; // 2 days to expiry date.
@ob_start("ob_gzhandler");
header("Expires: " . gmdate("D, d M Y H:i:s", time() + $offset) . " GMT");
header('Cache-Control: ');
header('Pragma: ');

foreach ($_GET as $key=&gt;$value)
{
    $_GET[$key] = urldecode($value);
}

$raw_content = '(No content available)';
if (!empty($_GET['content']))
{
-   $fp = @fopen(urldecode($_GET['content']),'r');
-   if (is_resource($fp))
+   $content = urldecode($_GET['content']);
+   if (preg_match('/^https?\:/', $content))

    {
        $fp = @fopen($content,'r');
        if (is_resource($fp))
        {
-           $raw_content = '';
-           while(!feof($fp)) $raw_content .= fread($fp,4096);
+           $fp = @fopen($content,'r');
+           if (is_resource($fp))
+           {
+             $raw_content = '';
+             while(!feof($fp)) $raw_content .= fread($fp,4096);
+           }

        }
    }
}

// just a namespace
class iBeginShare
{
    function isValidEmail($email)
    {
        $email = trim($email);
        return (bool)preg_match("/^([a-z0-9\+_\-]+)(\.[a-z0-9\+_\-]+)*@([a-z0-9\-]+\.)+[a-z]{2,6}$/ix", $email);
    }
    function quoteSmart($value)
    {
        if (is_array($value))
        {
            foreach ($value as $key =&gt; $value2):
                $value[$key] = htmlspecialchars((string) trim($value2), ENT_QUOTES, 'UTF-8');
            endforeach;
            return $value;
        }
        else
         {
            return htmlspecialchars((string) trim($value), ENT_QUOTES, 'UTF-8');
        }
    }

</div>
```
