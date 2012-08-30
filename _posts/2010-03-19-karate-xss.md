---
layout: post
title: Karate - XSS
tags:
- AppFuse
- Cross-Site Scripting (XSS)
- Solution
- template
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: Karate - XSS
  bfa_ata_display_body_title: 'on'
  bfa_ata_body_title_multi: Karate - XSS
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 98889b87f94e89f9dbc0b32137fea21b
  _sexybookmarks_shortUrl: http://bit.ly/d4sBIP
---
## Details
__Affected Software:__ Appfuse

__Fixed in Version:__  1.8.1

__Issue Type:__ Cross Site Scripting

Original Code: <a title="Karate" href="http://spotthevuln.com/2010/03/karate/" target="_blank">Found  Here</a>
## Description
Auditing template languages can be tricky.  Many times the tools and automation fall apart when dealing with template languages.  Also, many template languages separate markup and logic, forcing the code auditor to trace every alteration of markup.

In this specific example, we see that the developer has removed two lines that echoed the user's firstname and lastname to the HTML markup.  The removed lines were replaced with what appears to be encoded firstname and lastname values.  What's interesting is the birthdate and userid appear to be displayed without any encoding.  It would be interesting to see if the user/attacker could control these in a manner that could result in XSS.
## Developers Solution
[cce lang="diff"]

&lt;title&gt;${rc.getMessage("userList.title")}&lt;/title&gt;

&lt;button onclick="location.href='userform.html'"style="float: right; margin-top: -30px; width: 100px"&gt;Add User&lt;/button&gt;

&lt;table class="table" id="userList"&gt;

&lt;thead&gt;

&lt;tr&gt;

&lt;th&gt;${rc.getMessage("user.id")}&lt;/th&gt;

&lt;th&gt;${rc.getMessage("user.firstName")}&lt;/th&gt;

&lt;th&gt;${rc.getMessage("user.lastName")}&lt;/th&gt;

&lt;th&gt;${rc.getMessage("user.birthday")}&lt;/th&gt;

&lt;/tr&gt;

&lt;/thead&gt;

&lt;tbody&gt;

&lt;#list userList as user&gt;

&lt;#if user_index % 2 == 0&gt; &lt;tr class="odd"&gt;

&lt;#else&gt; &lt;tr class="even"&gt;

&lt;/#if&gt;

&lt;td&gt;&lt;a href="userform.html?id=${user.id}"&gt;${user.id}&lt;/a&gt;&lt;/td&gt;

-    &lt;td&gt;${user.firstName}&lt;/td&gt;

-    &lt;td&gt;${user.lastName}&lt;/td&gt;

+    &lt;td&gt;${user.firstName?html}&lt;/td&gt;

+    &lt;td&gt;${user.lastName?html}&lt;/td&gt;

&lt;td&gt;&lt;#if user.birthday??&gt;${user.birthday?date}&lt;/#if&gt;&lt;/td&gt;

&lt;/tr&gt;

&lt;/#list&gt;

&lt;/tbody&gt;

&lt;/table&gt;

&lt;script type="text/javascript"&gt;highlightTableRows("userList");&lt;/script

[/cce] 
