---
layout: post
title: Learning - Insecure Logging (Defense in Depth)
tags:
- All Vulnerabilities
- Apache
- bugs
- Defense In Depth
- hash
- hashing function
- information disclosure
- password protection
- PHP
- plugins
- protection mechanisms
- security flaws
- security software
- sensitive data
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '2'
  bfa_ata_body_title: Learning - Insecure Logging (Defense in Depth)
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Learning - Insecure Logging (Defense in Depth)
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  enclosure: ! 'http://spotthevuln.com/wordpress/wp-content/uploads/2010/05/mundanedetail.wav

    247042

    audio/x-wav

'
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 9bd69d5a66d280d37eb04982f10ca82b
  _sexybookmarks_shortUrl: http://bit.ly/cd3rN4
---
## Details
__Affected Software:__ AskApache Password Protect

__Fixed in Version:__  4.3.2

__Issue Type:__ Insecure Logging (Defense in Depth)

Original Code: <a title="Learning" href="http://spotthevuln.com/2010/05/learning/" target="_blank">Found Here</a>
## Description
This week's bug was discovered in the AskApache Password Protect plugin for WordPress. Once again, we are examining "security software" that is designed to provide various security protection mechanisms for a deployed WordPress blog. The description for the AskApache security plug-in is as follows:
<blockquote><em>Advanced Security: Password Protection, Anti-Spam, Anti-Exploits, more to come</em></blockquote>
A very noble effort indeed :)

This vulnerability was in the aa_pp_hashit() function. The aa_pp_hashit() function takes three arguments: $format, $user, and $pass. The aa_pp_hashit() function then attempts to create a hash containing the creds. Whenever I see functions utilizing crypto, I'm always reminded of this <a title="Mundane Detail" href="http://spotthevuln.com/wordpress/wp-content/uploads/2010/05/mundanedetail.wav" target="_blank">scene in Office Space</a> . In this particular patch, vulnerability was in this line:
<blockquote>aa_pp_mess('Created '.$format.' Hash for '.$user.' <span style="color: #ff0000;">with Password '.$pass</span>);</blockquote>
The aa_pp_mess() function actually logged the clear text username and password before putting it through a hashing function. There is rarely a need to log a clear text password... in fact, I'm going to go out on a limb here and say there is NEVER a good time when you should log a clear text password. Even password hashes or other weird representations of passwords shouldn't be logged. Logging sensitive data is always tricky. If you're logging sensitive data please consider the permissions required to access that sensitive data, ensure the file is properly ACL'd and conduct regular audits of log file access. Most importantly, ask yourself:  Why do I need to log this data?

The vulnerability was fixed by removing references to user password (and even references to the user that called the function). Now I just have to figure out why the AskApache devs are passing a default value for $pass :)
## Developer's Solution
```diff

// aa_pp_hashit
//-------------------------------------------------------------------------------------------
function aa_pp_hashit($format,$user='',$pass=''){
        global $aa_PP;
-   aa_pp_mess('Created '.$format.' Hash for '.$user.' with Password '.$pass);
+   aa_pp_mess('Created '.$format.' Hash');
    $hash='';
    switch ($format){
        case 'TEST':
                $hash=array();
                foreach($aa_PP['algorithms'] as $key=&gt;$value)$hash[]=aa_pp_hashit($key,"test{$key}","test{$key}");
        return $hash;
        break;
        case 'PLAIN':
        $hash=$user.':'.$pass;
        break;
        case 'CRYPT':
        $seed = NULL;
        for ($i = 0; $i &lt; 8; $i++) {$seed .= substr('0123456789abcdef', rand(0,15), 1);}
        $hash=$user.':'.crypt($pass, "$1$".$seed);
        break;
        case 'SHA1':
        $hash=$user.':{SHA}'.base64_encode(pack("H*", sha1($pass)));
        break;
        case 'MD5': // php.net/crypt.php#73619
        $saltt = substr(str_shuffle("abcdefghijklmnopqrstuvwxyz0123456789"), 0, 8);
        $len = strlen($pass);$text = $pass.'$apr1$'.$saltt;$bin = pack("H32", md5($pass.$saltt.$pass));
        for($i = $len; $i &gt; 0; $i -= 16) { $text .= substr($bin, 0, min(16, $i)); }
        for($i = $len; $i &gt; 0; $i &gt;&gt;= 1) { $text .= ($i &amp; 1) ? chr(0) : $pass{0}; }
        $bin = pack("H32", md5($text));
        for($i=0; $i&lt;1000; $i++) { $new = ($i &amp; 1) ? $pass : $bin; if ($i % 3) $new .= $saltt; if ($i % 7) $new .= $pass; $new .= ($i &amp; 1) ? $bin : $pass; $bin = pack("H32", md5($new)); }
        for($i=0; $i&lt;5; $i++) { $k = $i + 6; $j=$i + 12; if($j==16){ $j = 5; } $TRp = $bin[$i].$bin[$k].$bin[$j].$TRp; }
        $TRp = chr(0).chr(0).$bin[11].$TRp;
        $TRp = strtr(strrev(substr(base64_encode($TRp), 2)),"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
        "./0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz");
        $hash="$user:$"."apr1"."$".$saltt."$".$TRp;
        break;
    }

    return $hash;
}//=========================================================================================================================
// aa_pp_show_encryptions
//-------------------------------------------------------------------------------------------
function aa_pp_show_encryptions($label,$type=0){
    global $aa_PP;

    if($type==0)
        {
        ?&gt;
        &lt;p&gt;&lt;label&gt;&lt;?php _e($label); ?&gt;&lt;br /&gt;
        &lt;select name="aapassformat" id="aapassformat"&gt;
        &lt;?php foreach($aa_PP['algorithms'] as $key=&gt;$value){?&gt;
                &lt;option value="&lt;?php echo $key;?&gt;"&lt;?php if($aa_PP['format']==$key)echo ' selected="selected"';elseif($aa_PP['algorithms'][$key]['enabled']!='1')echo ' disabled="disabled"';?&gt;&gt;&lt;?php echo $key;?&gt;   &lt;/option&gt;
        &lt;?php }?&gt;
        &lt;/select&gt;
        &lt;/label&gt;&lt;/p&gt;
     &lt;?php
     }
         elseif($type==3)
         {
     ?&gt;
        &lt;p&gt;&lt;label&gt;&lt;?php _e($label); ?&gt;&lt;br /&gt;
        &lt;input id="aapassformat" name="aapassformat" type="hidden" value="&lt;?php echo $aa_PP['format']; ?&gt;" /&gt;&lt;/label&gt;&lt;/p&gt;
        &lt;ul&gt;
        &lt;?php foreach($aa_PP['algorithms'] as $key=&gt;$value){?&gt;
                &lt;li&gt;&lt;label&gt;&lt;input name="aapassformat" id="aapassformat&lt;?php echo strtolower($key);?&gt;" type="radio" value="&lt;?php echo $key;?&gt;" &lt;?php if($aa_PP['format']==$key)echo 'checked="checked"';
                elseif($aa_PP['algorithms'][$key]['enabled']!='1')echo 'disabled="disabled"'; ?&gt; /&gt; &lt;strong&gt;&lt;?php echo $key;?&gt;&lt;/strong&gt; -
            &lt;?php echo $aa_PP['algorithms'][$key]['desc'];?&gt;&lt;/label&gt;&lt;/li&gt;
        &lt;?php }?&gt;
        &lt;/ul&gt;
    &lt;?php
    }
    else if($type==4)
        {
     ?&gt;
        &lt;h4&gt;&lt;?php _e($label); ?&gt;&lt;/h4&gt;
        &lt;?php foreach($aa_PP['algorithms'] as $key=&gt;$value){?&gt;
                &lt;p&gt;&lt;strong&gt;&lt;?php echo $key;?&gt;&lt;/strong&gt; - &lt;?php echo $aa_PP['algorithms'][$key]['desc'];?&gt;&lt;/p&gt;
        &lt;?php }?&gt;
        &lt;hr style="visibility:hidden;padding-top:.25em;clear:both;" /&gt;
    &lt;?php
    }
}//=========================================================================================================================

// aa_pp_mess
//-------------------------------------------------------------------------------------------
function aa_pp_mess($message=''){
        if(@defined('AA_PP_DEBUG_LOGFILE'))error_log($message, 3, AA_PP_DEBUG_LOGFILE);
-        else error_log($message);
+  else if(AA_PP_DEBUG)error_log($message)
    if(AA_PP_DEBUG){ ?&gt; &lt;div id="message" style="margin:1em auto;"&gt;&lt;p&gt;&lt;?php echo $message;?&gt;&lt;/p&gt;&lt;/div&gt; &lt;?php }
}//=========================================================================================================================


```
