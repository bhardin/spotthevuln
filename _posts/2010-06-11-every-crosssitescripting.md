---
layout: post
title: Everything - Cross Site Scripting
tags:
- All Vulnerabilities
- arbitrary values
- attacker
- client side script
- cross site scripting
- Cross-Site Scripting (XSS)
- document write
- DoJo
- JavaScript
- javascript variables
- parameter values
- querystring
- regular expression
- sdk
- Solution
- text javascript
- validation code
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 48f627a4b8ec185db9ee7e9b068fb65a
  _sexybookmarks_shortUrl: http://bit.ly/bEY87W
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
__Affected Software:__ Dojo Toolkit SDK

__Fixed in Version:__  1.4.2

__Issue Type:__ Cross Site Scripting (XSS)

Original Code: <a title="Everything" href="http://spotthevuln.com/2010/06/everything/" target="_blank">Found Here</a>
## Description
This was a bug reported by the <a title="Gotham" href="http://www.gdssecurity.com/" target="_blank">Gotham Digital Science </a>against the Dojo toolkit SDK.  The Dojo toolkit is a popular toolkit used by numerous websites… so in essence this bug provided attackers an opportunity to XSS a large number of websites across the Internet.

The bug begins by the capturing of untrusted parameter values from the querystring.  This is done by the following JavaScript:
<blockquote><code>var qstr = window.location.search.substr(1);</code></blockquote>
qstr is then split based on the "&amp;" character, proving values for various JavaScript variables including DoJoURL and TestURL.   The attacker is free to provide arbitrary values for DoJoURL and TestURL by simply providing the proper querystring values.  For example:
<blockquote>runner.html?dojoUrl=attacker-controlled&amp;testUrl=attackercontrolled</blockquote>
the attacker supplied values are then used in a document.write() statement, giving the attacker the opprotuntiy to inject arbitrary client side script into any website that happens to include the Dojo library.  The vulnerable document.write() statements are provided below:
<blockquote>document.write("&lt;scr"+"ipt type='text/javascript' djConfig='isDebug: true' src='"+<span style="color: #ff0000;">dojoUrl</span>+"'&gt;&lt;/scr"+"ipt&gt;");

document.write("&lt;scr"+"ipt type='text/javascript' src='"+<span style="color: #ff0000;">testUrl</span>+".js'&gt;&lt;/scr"+"ipt&gt;");</blockquote>
The Dojo developers addressed this vulnerability by replacing characters from the attacker controlled input.  The specific regular expression used is provided below:
<blockquote>value=tp[1].replace(/[&lt;&gt;"']/g, "");</blockquote>
I see a major issue with this code fix… can you spot it as well?
<h2>Developers Solution</h2>
[cce lang="diff"]

                &lt;script type="text/javascript"&gt;
                        // workaround for bug in Safari 3.  See #7189
                        if (/3[\.0-9]+ Safari/.test(navigator.appVersion))
                        {
                                window.console = {
                                    origConsole: window.console,
                                    log: function(s){
                                                this.origConsole.log(s);
                                        },
                                        info: function(s){
                                                this.origConsole.info(s);
                                        },
                                        error: function(s){
                                                this.origConsole.error(s);
                                        },
                                        warn: function(s){
                                                this.origConsole.warn(s);
                                        }
                               };
                        }
                &lt;/script&gt;
 
                &lt;script type="text/javascript"&gt;
                        window.dojoUrl = "../../dojo/dojo.js";
                        window.testUrl = "";
                        window.testModule = "";
 
                        // parse out our test URL and our Dojo URL from the query string
                        var qstr = window.location.search.substr(1);
                        if(qstr.length){
                                var qparts = qstr.split("&amp;");
                                for(var x=0; x&lt;qparts.length; x++){
-                                       var tp = qparts[x].split("=");
-                                       if(tp[0] == "dojoUrl"){
-                                               window.dojoUrl = tp[1];
-                                       }
-                                       if(tp[0] == "testUrl"){
-                                               window.testUrl = tp[1];
-                                       }
-                                       if(tp[0] == "testModule"){
-                                               window.testModule = tp[1];
-                                       }
-                                       if(tp[0] == "registerModulePath"){
-                                               var modules = tp[1].split(";");
-                                               window.registerModulePath=[];
-                                               for (var i=0; i&lt;modules.length;i++){
-                                                        window.registerModulePath.push(modules[i].split(","));
-                                               }
-                                       }
+                                       var tp = qparts[x].split("="), name=tp[0], value=tp[1].replace(/[&lt;&gt;"']/g, "");  // replace() to avoid XSS attack 
+                                       switch(name){ 
+                                               case "dojoUrl": 
+                                               case "testUrl": 
+                                               case "testModule": 
+                                                       window[name] = value; 
+                                                       break; 
+                                               case "registerModulePath": 
+                                                       var modules = value.split(";"); 
+                                                       window.registerModulePath=[]; 
+                                                       for (var i=0; i&lt;modules.length;i++){ 
+                                                               window.registerModulePath.push(modules[i].split(",")); 
+                                                       } 
+                                               break; 
                                }
                        }
 
                        document.write("&lt;scr"+"ipt type='text/javascript' djConfig='isDebug: true' src='"+dojoUrl+"'&gt;&lt;/scr"+"ipt&gt;");
                &lt;/script&gt;
                &lt;script type="text/javascript"&gt;
                        try{
                                dojo.require("doh.runner");
                        }catch(e){
                                document.write("&lt;scr"+"ipt type='text/javascript' src='runner.js'&gt;&lt;/scr"+"ipt&gt;");
                        }
                        if(testUrl.length){
                                document.write("&lt;scr"+"ipt type='text/javascript' src='"+testUrl+".js'&gt;&lt;/scr"+"ipt&gt;");
                        }
                &lt;/script&gt;
                &lt;style type="text/css"&gt;
                        @import "../../dojo/resources/dojo.css";
                        /*
                        body {
                                margin: 0px;
                                padding: 0px;
                                font-size: 13px;
                                color: #292929;
                                font-family: Myriad, Lucida Grande, Bitstream Vera Sans, Arial, Helvetica, sans-serif;
                                *font-size: small;
                                *font: x-small;
                        }
 
                        th, td {
                                font-size: 13px;
                                color: #292929;
                                font-family: Myriad, Lucida Grande, Bitstream Vera Sans, Arial, Helvetica, sans-serif;
                                font-weight: normal;
                        }
 
                        * body {
                                line-height: 1.25em;
                        }

[/cce] 
