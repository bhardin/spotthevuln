---
layout: post
title: Everything
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: ccb753a36c97b8a039149c7e995dd662
  _sexybookmarks_shortUrl: http://bit.ly/aD3PiL
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
---
<blockquote><strong>Anybody who knows everything should be told a thing or two.
- Franklin P. Jones</strong></blockquote>
```php
<?php
                &lt;script type="text/javascript"&gt;
                        // workaround for bug in Safari 3. See #7189
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
                                        var tp = qparts[x].split("=");
                                        if(tp[0] == "dojoUrl"){
                                                window.dojoUrl = tp[1];
                                        }
                                        if(tp[0] == "testUrl"){
                                                window.testUrl = tp[1];
                                        }
                                        if(tp[0] == "testModule"){
                                                window.testModule = tp[1];
                                        }
                                        if(tp[0] == "registerModulePath"){
                                                var modules = tp[1].split(";");
                                                window.registerModulePath=[];
                                                for (var i=0; i&lt;modules.length;i++){
                                                         window.registerModulePath.push(modules[i].split(","));
                                                }
                                        }
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

```
