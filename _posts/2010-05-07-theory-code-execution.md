---
layout: post
title: Theory - Code Execution
tags:
- backup
- File Inclusion
- globals
- PHP
- plugin
- register_globals
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  _sexybookmarks_shortUrl: http://bit.ly/aw0WFE
  _sexybookmarks_permaHash: 2a89eb8f1990ef85157c572634a3db47
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ BackupWordPress

__Fixed in Version:__  0.4.3

__Issue Type:__ Code Execution

Original Code: <a title="Theory" href="http://spotthevuln.com/2010/05/theory/" target="_blank">Found Here</a>
## Description
This particular bug was a remote file inclusion vulnerability in a WordPress plugin known as BackupWordPress. This particular vulnerability was actually publically disclosed on Milworm by the "Xmors Underground Team" (http://www.milw0rm.com/exploits/4593). The vulnerability, combined with the register_globals behavior in older versions of PHP allowed attackers to simply provide the "$GLOBALS['bkpwp_plugin_path']" via the URL in a GET request, supplying an attacker controlled location for the include.

The developers fixed this particular vulnerability by removing the $GLOBALS from the source.
## Developer's Solution
```diff

&lt;?php

-require_once $GLOBALS['bkpwp_plugin_path']."Archive/Predicate.php";
+require_once BKPWP_PLUGIN_PATH."Archive/Predicate.php";
require_once "MIME/Type.php";

/**
 * Keep only the files that have a specific MIME type
 *
 * @see        File_Archive_Predicate, File_Archive_Reader_Filter
 */
class File_Archive_Predicate_MIME extends File_Archive_Predicate
{
    var $mimes;

    /**
     * @param $extensions array or comma separated string of allowed extensions
     */
    function File_Archive_Predicate_MIME($mimes)
    {
        if (is_string($mimes)) {
            $this-&gt;mimes = explode(",",$mimes);
        } else {
            $this-&gt;mimes = $mimes;
        }
    }
    /**
     * @see File_Archive_Predicate::isTrue()
     */
    function isTrue(&amp;$source)
    {
        $sourceMIME = $source-&gt;getMIME();
        foreach ($this-&gt;mimes as $mime) {
            if (MIME_Type::isWildcard($mime)) {
                $result = MIME_Type::wildcardMatch($mime, $sourceMIME);
            } else {
                $result = ($mime == $sourceMIME);
            }
            if ($result !== false) {
                return $result;
            }
        }
        return false;
    }
}

?&gt;

```
