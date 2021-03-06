---
layout: post
title: Renting - Cross-Site Scripting
tags:
- Cross-Site Scripting (XSS)
- Openfire
- PHP
- Solution
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 7050ff1e0b730f98bcb57166f8acaf28
  _sexybookmarks_shortUrl: http://bit.ly/aRjI0b
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: Renting - Cross-Site Scripting
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Renting - Cross-Site Scripting
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _wp_old_slug: renting-cross-site-scripting-2
---
## Details
__Affected Software:__ FastPath Plugin for OpenFire

__Fixed in Version:__  3.5.3

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Renting" href="http://spotthevuln.com/2010/06/renting/" target="_blank">Found Here</a>
## Description
This weeks' bug consisted of vulnerabilities in the FastPath plugin for OpenFire. The FastPath plugin adds support for queued chat requests, much like those chat queues found on support websites across the internet. Several code changes were made with this change list. The first set of changes simply cleans up the imports used by the page and has no significant security impact.

The second set of changes we see URL encode user/attacker controlled items placed in a HTTP Redirect. Although the security fix was labeled as "XSS fixes", this particular change was likely to prevent CRLF injection into the location header for the JSP redirect (which could ultimately lead to XSS... ).

The remainder of the code fixes simply encodes HTML output before redisplaying it back to the user, defeating the classic XSS attack.
## Developer's Solution
```diff

-&lt;%@ page import="org.jivesoftware.smack.util.StringUtils"%&gt;
-&lt;%@ page import="org.jivesoftware.webchat.util.ParamUtils, java.util.*"%&gt;
+&lt;%@ page import="java.util.*"%&gt;
&lt;%@ page import="org.jivesoftware.webchat.actions.WorkgroupStatus" %&gt;
+&lt;%@ page import="org.jivesoftware.webchat.util.*" %&gt;
&lt;!-- Get and Set Workgroup --&gt;
&lt;jsp:useBean /&gt;
&lt;jsp:setProperty property="*" /&gt;
&lt;%
boolean authFailed = ParamUtils.getParameter(request, "authFailed") != null;

String location = (String)session.getAttribute("pageLocation");
if (chatUser.hasSession()) {
chatUser.removeSession();
}

String workgroup = chatUser.getWorkgroup();
String chatID = chatUser.getChatID();
if (chatID == null) {
chatID = StringUtils.randomString(10);
}

Workgroup chatWorkgroup = WorkgroupStatus.getWorkgroup(workgroup);
if (!chatWorkgroup.isAvailable()) {
-       response.sendRedirect("email/leave-a-message.jsp?workgroup=" + workgroup);
+ response.sendRedirect("email/leave-a-message.jsp?workgroup=" + StringUtils.URLEncode(workgroup, "utf-8"));
return;
}

...&lt;SNIP&gt;...

&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Information &lt;/title&gt;

&lt;link rel="stylesheet"

href="style.jsp?workgroup=&lt;%= workgroup %&gt;"/&gt;&lt;script src="common.js"&gt;//Ignore&lt;/script&gt;
&lt;/head&gt;
&lt;body style="margin-top:0px; margin-bottom:20px; margin-right:20px;margin-left:20px"&gt;
&lt;table width="100%" cellpadding="3" cellspacing="2"&gt;
&lt;tr&gt;&lt;td colspan="2" height="1%"&gt;
&lt;img src="getimage?image=logo&amp;workgroup=&lt;%= workgroup %&gt;"/&gt;
&lt;/td&gt;
&lt;/tr&gt;
&lt;form action="queue.jsp" method="post"&gt;
&lt;!-- Identify all hidden variables. All variables will be passed to the metadata router.
You can do any name-value pairing you like. Such as product=Jive Live Assistant. Such
data can be used to effectivly route to a particular queue within a workgroup.
--&gt;
-       &lt;input value="&lt;%= workgroup %&gt;"/&gt;
-       &lt;input value="&lt;%= chatID %&gt;" /&gt;
+       &lt;input type="hidden" name="workgroup" value="&lt;%= StringUtils.escapeHTMLTags(workgroup) %&gt;"/&gt;
+       &lt;input type="hidden" name="chatID" value="&lt;%= StringUtils.escapeHTMLTags(chatID) %&gt;" /&gt;
&lt;!-- End of Hidden Variable identifiers --&gt;
&lt;tr&gt;
&lt;td colspan="2" height="1%"&gt;
&lt;br/&gt;&lt;%=  welcomeText %&gt;
&lt;/td&gt;
&lt;/tr&gt;
&lt;tr&gt;
&lt;td height="1%"&gt;
&lt;br/&gt;
&lt;/td&gt;
&lt;/tr&gt;

&lt;% if (authFailed) { %&gt;
&lt;tr valign="top"&gt;
&lt;td colspan="2" height="1%" nowrap&gt;&lt;span&gt;Authentication Failed&lt;/span&gt;&lt;/td&gt;
&lt;/tr&gt;
&lt;% } %&gt;

&lt;%
Iterator fields = workgroupForm.getFields();
while(fields.hasNext()){
hasElements = true;
FormField field = (FormField)fields.next();
String label = field.getLabel();
boolean required = field.isRequired();
String requiredStr = required ? "&amp;nbsp;&lt;span class=\"error\"&gt;*&lt;/span&gt;" : "";
if(!field.getType().equals(FormField.TYPE_HIDDEN)){
%&gt;
&lt;tr valign="top"&gt;
&lt;td height="1%" width="1%" nowrap&gt;&lt;%= label%&gt;&lt;%= requiredStr%&gt;&lt;/td&gt;&lt;td&gt;&lt;%= FormUtils.createAnswers

(field, request)%&gt;&lt;/td&gt;
&lt;/tr&gt;
&lt;% } } %&gt;

&lt;tr valign="top"&gt;
&lt;td height="1%"&gt;
&lt;!-- All workgroup defined variables --&gt;
&lt;%
fields = workgroupForm.getFields();
while(fields.hasNext()){
FormField field = (FormField)fields.next();
if(field.getType().equals(FormField.TYPE_HIDDEN)){
%&gt;
&lt;%= FormUtils.createDynamicField(field, request)%&gt;
&lt;% }} %&gt;
&lt;!-- End of Defined Variables --&gt;

&lt;% // Handle page location
if(location != null){ %&gt;
-                    &lt;input value="&lt;%= location%&gt;" /&gt;
+       &lt;input type="hidden" name="location" value="&lt;%= StringUtils.escapeHTMLTags(location)%&gt;" /&gt;
&lt;% } %&gt;
&lt;/td&gt;
&lt;td&gt; &lt;input value="&lt;%= startButton%&gt;"/&gt;&lt;/td&gt;
&lt;/tr&gt;
&lt;tr&gt;

&lt;/tr&gt;
&lt;/form&gt;
&lt;/table&gt;
-&lt;div style="position:absolute;bottom:0px;right:5px"&gt;&lt;img src="getimage?image=poweredby&amp;workgroup=&lt;%= workgroup %&gt;"/&gt;&lt;/div&gt;
+&lt;div style="position:absolute;bottom:0px;right:5px"&gt;&lt;img src="getimage?image=poweredby&amp;workgroup=&lt;%=

StringUtils.URLEncode(workgroup, "utf-8") %&gt;"/&gt;&lt;/div&gt;
&lt;/body&gt;
&lt;%if(!hasElements){ %&gt;
&lt;script&gt;
document.f.submit();
&lt;/script&gt;
&lt;%}%&gt;
&lt;/html&gt;

```
