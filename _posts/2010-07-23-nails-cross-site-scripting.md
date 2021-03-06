---
layout: post
title: Nails - Cross-Site Scripting
tags:
- All Vulnerabilities
- attacker
- bugs
- Cross-Site Scripting (XSS)
- document write
- DoJo
- html markup
- JavaScript
- security flaws
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  bfa_ata_display_body_title: ''
  bfa_ata_body_title: ''
  _edit_last: '2'
  bfa_ata_meta_title: ''
  bfa_ata_body_title_multi: ''
  _aktt_hash_meta: ''
  aktt_notify_twitter: 'yes'
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 7539691333be600038fa2ff6a4991658
  _sexybookmarks_shortUrl: http://bit.ly/cEDGlu
---
## Details
__Affected Software:__ DojoToolkit

__Fixed in Version:__  1.4.2

__Issue Type:__ XSS

Original Code: <a title="Nails" href="http://spotthevuln.com/2010/07/nails/" target="_blank">Found    Here</a>
## Description
This week's vulnerability is a DOM based XSS that could be found in a JavaScript file provided by the DojoToolkit. This JavaScript file was included (via script src) in many pages throughout the DojoToolkit, making those pages vulnerable to XSS. Unlike traditional XSS bugs, server side processing is not required for certain types of DOM based XSS. This is an important concept to understand as some code auditors will skip static pages assuming the attacker will not have the ability to control any values used by the page.

The bug starts here:
<blockquote>if(window.location.href.indexOf("?") &gt; -1){</blockquote>
The JavaScript pulls the address of the loaded page and checks to see if the address contains the "?" character. If the "?" character is found, the JavaScript begins parsing and splitting the URI into various arrays. This parsing and splitting is done in the lines provided below:
<blockquote>var str = window.location.href.substr(window.location.href.indexOf("?")+1).split(/#/);
var ary  = str[0].split(/&amp;/);
for(var i=0; i&lt;ary.length; i++){
var split = ary[i].split(/=/),</blockquote>
The vulnerable assignment occurs here:
<blockquote><span style="color: #ff0000;">value </span>= split[1];</blockquote>
The JavaScript above essentially grabs a querystring value (attacker supplied) and assigns it to the "value" variable. Later, the "value" variable is used in several places, for example:
<blockquote>dojo.config.locale = locale = value;

document.getElementsByTagName("html")[0].dir = value;

<span style="color: #ff0000;">theme </span>= <span style="color: #ff0000;">value</span>;</blockquote>
Considering the assignments listed above, we have a couple different variables that are tainted. I've highlighted the tainted variables in red. Tracing the "theme" assignment shown above, we see the tainted value being passed to a document.write statement, resulting in XSS.
<blockquote>var <span style="color: #ff0000;">themeCss </span>= d.moduleUrl("dijit.themes",theme+"/"+<span style="color: #ff0000;">theme</span>+".css");

var <span style="color: #ff0000;">themeCssRtl </span>= d.moduleUrl("dijit.themes",theme+"/"+<span style="color: #ff0000;">theme</span>+"_rtl.css");

document.write('&lt;link rel="stylesheet" type="text/css" href="'+<span style="color: #ff0000;">themeCss</span>+'"&gt;');

document.write('&lt;link rel="stylesheet" type="text/css" href="'+<span style="color: #ff0000;">themeCssRtl</span>+'"&gt;');</blockquote>
The patch checked in by the DojoToolkit team sanitizes the "value" JavaScript variable by allowing only word characters (^\w).
## Developer's Solution
```diff

if(window.location.href.indexOf("?") &gt; -1){
var str =  window.location.href.substr(window.location.href.indexOf("?")+1).split(/#/);
var ary  = str[0].split(/&amp;/);
for(var i=0; i&lt;ary.length; i++){
var split = ary[i].split(/=/),
key = split[0],
-value = split[1];
+value = split[1].replace(/[^\w]/g, ""); // replace() to prevent XSS attack
switch(key){
case "locale":
// locale string | null
dojo.config.locale = locale = value;
break;
case "dir":
// rtl | null
document.getElementsByTagName("html")[0].dir = value;
break;
case "theme":
// tundra | soria | noir | squid | nihilo | null
theme = value;
break;
case "a11y":
if(value){ testMode = "dijit_a11y"; }
}
}
}

// always include the default theme files:
if(theme || testMode){

if(theme){
var themeCss = d.moduleUrl("dijit.themes",theme+"/"+theme+".css");
var themeCssRtl =  d.moduleUrl("dijit.themes",theme+"/"+theme+"_rtl.css");
document.write('&lt;link rel="stylesheet" type="text/css"  href="'+themeCss+'"&gt;');
document.write('&lt;link rel="stylesheet" type="text/css"  href="'+themeCssRtl+'"&gt;');
}

if(dojo.config.parseOnLoad){
dojo.config.parseOnLoad = false;
dojo.config._deferParsing = true;
}

d.addOnLoad(function(){

// set the classes
var b = dojo.body();
if(theme){
dojo.removeClass(b, defTheme);
if(!d.hasClass(b, theme)){ d.addClass(b, theme); }
var n = d.byId("themeStyles");
if(n){ d.destroy(n); }
}
if(testMode){ d.addClass(b, testMode); }
if(dojo.config._deferParsing){
// attempt to elimiate race condition introduced by this
// test helper file. 120ms to allow CSS to finish/process?
setTimeout(dojo.hitch(d.parser, "parse", b), 120);
}

});
}

})();

```
