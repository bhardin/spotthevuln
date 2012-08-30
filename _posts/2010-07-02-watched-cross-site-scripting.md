---
layout: post
title: Watched - Cross Site Scripting
tags:
- arbitrary values
- attacker
- Cross-Site Scripting (XSS)
- html markup
- PHP
- php documentation
- query string
- Solution
- string parameters
- Wordpress
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: e2c6be61db0fc3e9575af717733d1f18
  _sexybookmarks_shortUrl: http://bit.ly/buQCK6
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ qTranslate  plugin

__Fixed in Version:__  2.0.2

__Issue Type:__ XSS

Original Code: <a title="Watched" href="http://spotthevuln.com/2010/06/watched/" target="_blank">Found Here</a>
## Description
Whew!  This is a lot of code for a simple change!  This bug affected the qTranslate plugin for WordPress.  The bug used the $_SERVER['REQUEST_URI'] variable without realizing it could contain arbitrary values supplied by an attacker.  The $_SERVER['REQUEST_URI'] variable is used directly in an HREF in the HTML markup, resulting in a classic XSS vulnerability.

The <a title="PHP Doc" href="http://php.net/manual/en/reserved.variables.server.php" target="_blank">PHP documentation </a>states that ['REQUEST_URI'] represents:
<blockquote>The URI which was given in order to access this page; for instance, '/index.html'.</blockquote>
No escaping is done before returning the predefined value.  Also, ['REQUEST_URI'] actually returns the file requested as well as any query string parameters in the URI.  For example, a request for <a href="http://server/index.php?blah=foo">http://server/index.php?blah=foo</a> will return 'index.php?blah=foo'.  With this in mind, the attacker is free to set up arbitrary query string parameters which contain the XSS payload
<blockquote>http://server/qtranslate_widget.php?xss="&gt;&lt;script&gt;alert(document.domain)&lt;/script&gt;</blockquote>
<h2>Developers Solution</h2>
[cce lang="diff"]

&lt;?php
function qtrans_convertURL($url='', $lang='', $forceadmin = false) {
 global $q_config;
 
 // invalid language
 if($url=='') $url = clean_url($q_config['url_info']['url']);
 if($lang=='') $lang = $q_config['language'];
 if(defined('WP_ADMIN')&amp;&amp;!$forceadmin) return $url;
 if(!qtrans_isEnabled($lang)) return "";
 
 // &amp; workaround
 $url = str_replace('&amp;amp;','&amp;',$url);
 $url = str_replace('&amp;#038;','&amp;',$url);
 
 // check if it's an external link
 $urlinfo = qtrans_parseURL($url);
 $home = rtrim(get_option('home'),"/");
 if($urlinfo['host']!='') {
  // check for already existing pre-domain language information
  if($q_config['url_mode'] == QT_URL_DOMAIN &amp;&amp; preg_match("#^([a-z]{2}).#i",$urlinfo['host'],$match)) {
   if(qtrans_isEnabled($match[1])) {
    // found language information, remove it
    $url = preg_replace("/".$match[1]."\./i","",$url, 1);
    // reparse url
    $urlinfo = qtrans_parseURL($url);
   }
  }
  if(substr($url,0,strlen($home))!=$home) {
   return $url;
  }
  // strip home path
  $url = substr($url,strlen($home));
 } else {
  // relative url, strip home path
  $homeinfo = qtrans_parseURL($home);
  if($homeinfo['path']==substr($url,0,strlen($homeinfo['path']))) {
   $url = substr($url,strlen($homeinfo['path']));
  }
 }
 
 // check for query language information and remove if found
 if(preg_match("#(&amp;|\?)lang=([^&amp;\#]+)#i",$url,$match) &amp;&amp; qtrans_isEnabled($match[2])) {
  $url = preg_replace("#(&amp;|\?)lang=".$match[2]."&amp;?#i","$1",$url);
 }
 
 // remove any slashes out front
 $url = ltrim($url,"/");
 
 // remove any useless trailing characters
 $url = rtrim($url,"?&amp;");
 
 // reparse url without home path
 $urlinfo = qtrans_parseURL($url);
 
 // check if its a link to an ignored file type
 $ignore_file_types = preg_split('/\s*,\s*/',strtolower($q_config['ignore_file_types']));
 $pathinfo = pathinfo($urlinfo['path']);
 if(isset($pathinfo['extension']) &amp;&amp; in_array(strtolower($pathinfo['extension']), $ignore_file_types)) {
  return $home."/".$url;
 }
 
 switch($q_config['url_mode']) {
  case QT_URL_PATH: // pre url
   // might already have language information
   if(preg_match("#^([a-z]{2})/#i",$url,$match)) {
    if(qtrans_isEnabled ($match[1])) {
     // found language information, remove it
     $url = substr($url, 3);
    }
   }
   if($lang!=$q_config['default_language']) $url = $lang."/".$url;
   break;
  case QT_URL_DOMAIN: // pre domain
   if($lang!=$q_config['default_language']) $home = preg_replace("#//#","//".$lang.".",$home,1);
   break;
  default: // query
   if($lang!=$q_config['default_language']){
    if(strpos($url,'?')===false) {
     $url .= '?';
    } else {
     $url .= '&amp;';
    }
    $url .= "lang=".$lang;
   }
 }
 
 // see if cookies are activated
 if(!$q_config['cookie_enabled'] &amp;&amp; !$q_config['url_info']['internal_referer'] &amp;&amp; $urlinfo['path'] == '' &amp;&amp; $lang == $q_config['default_language'] &amp;&amp; $q_config['language'] != $q_config['default_language']) {
  // :( now we have to make unpretty URLs
  $url = preg_replace("#(&amp;|\?)lang=".$match[2]."&amp;?#i","$1",$url);
  if(strpos($url,'?')===false) {
   $url .= '?';
  } else {
   $url .= '&amp;';
  }
  $url .= "lang=".$lang;
 }
 
 // &amp;amp; workaround
 $complete = str_replace('&amp;','&amp;amp;',$home."/".$url);
 return $complete;
}

...&lt;SNIP...

function qtrans_use($lang, $text, $show_available=false) {
        global $q_config;
        // return full string if language is not enabled
        if(!qtrans_isEnabled($lang)) return $text;
        if(is_array($text)) {
                // handle arrays recursively
                foreach($text as $key =&gt; $t) {
                        $text[$key] = qtrans_use($lang,$text[$key],$show_available);
                }
                return $text;
        }
        
        if(is_object($text)) {
                foreach(get_object_vars($text) as $key =&gt; $t) {
                        $text-&gt;$key = qtrans_use($lang,$text-&gt;$key,$show_available);
                }
                return $text;
        }
        
        // get content
        $content = qtrans_split($text);
        // find available languages
        $available_languages = array();
        foreach($content as $language =&gt; $lang_text) {
                $lang_text = trim($lang_text);
                if(!empty($lang_text)) $available_languages[] = $language;
        }
        
        // if no languages available show full text
        if(sizeof($available_languages)==0) return $text;
        // if content is available show the content in the requested language
        $content[$lang] = trim($content[$lang]);
        if(!empty($content[$lang])) {
                return $content[$lang];
        }
        // content not available in requested language (bad!!) what now?
        if(!$show_available){
                // check if content is available in default language, if not return first language found. (prevent empty result)
                if($lang!=$q_config['default_language'])
                        return "(".$q_config['language_name'][$q_config['default_language']].") ".qtrans_use($q_config['default_language'], $text, $show_available);
                foreach($content as $language =&gt; $lang_text) {
                        $lang_text = trim($lang_text);
                        if(!empty($lang_text)) {
                                return $lang_text;
                        }
                }
        }
        // display selection for available languages
        $available_languages = array_unique($available_languages);
        $language_list = "";
        if(preg_match('/%LANG:([^:]*):([^%]*)%/',$q_config['not_available'][$lang],$match)) {
                $normal_seperator = $match[1];
                $end_seperator = $match[2];
                // build available languages string backward
                $i = 0;
                foreach($available_languages as $language) {
                        if($i==1) $language_list  = $end_seperator.$language_list;
                        if($i&gt;1) $language_list  = $normal_seperator.$language_list;
-                        $language_list = "&lt;a href=\"".qtrans_convertURL($_SERVER['REQUEST_URI'], $language)."\"&gt;".$q_config['language_name'][$language]."&lt;/a&gt;".$language_list;
+                       $language_list = "&lt;a href=\"".qtrans_convertURL('', $language)."\"&gt;".$q_config['language_name'][$language]."&lt;/a&gt;".$language_list;
                        $i++;
                }
        }
        return "&lt;p&gt;".preg_replace('/%LANG:([^:]*):([^%]*)%/', $language_list, $q_config['not_available'][$lang])."&lt;/p&gt;";
}
 
?&gt;

[/cce] 
