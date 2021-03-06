---
layout: post
title: Drop Top - Cross-Site Scripting
tags:
- All Vulnerabilities
- attacker
- Cross-Site Scripting (XSS)
- html markup
- input value
- Java
- jive software
- JSP
- lt
- null return
- Openfire
- request password
- request url
- request username
- return url
- setup mode
- Solution
- static string
- string password
- string url
- string username
- stringutils
- url parameter
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: Drop Top - Cross-Site Scripting
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Drop Top - Cross-Site Scripting
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: 84b0ecf8806ba60ea50d56fa2863167a
  _sexybookmarks_shortUrl: http://bit.ly/cumr7r
---
## Details
__Affected Software:__ Openfire by Ignite Realtime

__Fixed in Version:__  3.6.1

__Issue Type:__ XSS

Original Code: <a title="Drop Top" href="http://spotthevuln.com/2010/07/drop-top/" target="_blank">Found   Here</a>
## Description
This is a straightforward XSS bug that affecting the Admin Console of OpenFire by Ignite Realtime/Jive software. The code fix is simple, encode a tainted URL variable before using it in markup. The URL variable is assigned an attacker controlled value here:
<blockquote>String url = ParamUtils.getParameter(request, "url");</blockquote>
And is later used in the HTML markup here:
<blockquote>&lt;input value="&lt;%= url %&gt;"&gt;</blockquote>
The one line fix is to encode the URL parameter, which was done here:
<blockquote>url = org.jivesoftware.util.StringUtils.escapeHTMLTags(url);</blockquote>
Looking through the code, we see that OpenFire had previously fixed an XSS vulnerability just a few lines above in the USERNAME variable. There is even comment indicating so!  It surprising that the Ignite Realtime/Jive developers missed this one as it is literally two lines below the previous fix.
<blockquote>String username = ParamUtils.getParameter(request, "username");

if (username != null) {

username = JID.escapeNode(username);

}

<span style="color: #ff0000;">// Escape HTML tags in username to prevent cross-site scripting attacks. This</span>

<span style="color: #ff0000;">// is necessary because we display the username in the page below.</span>

username = org.jivesoftware.util.StringUtils.escapeHTMLTags(username);

String password = ParamUtils.getParameter(request, "password");

String url = ParamUtils.getParameter(request, "url");</blockquote>
The assignment of the PASSWORD variable is interesting :)
## Developer's Solution
```diff

&lt;%!

static String go(String url) {

if (url == null) {

return "index.jsp";

}

else {

return url;

}

}

%&gt;

&lt;%-- Check if in setup mode --%&gt;

&lt;%

if (admin.isSetupMode()) {

response.sendRedirect("setup/index.jsp");

return;

}

%&gt;

&lt;% // get parameters

String username = ParamUtils.getParameter(request, "username");

if (username != null) {

username = JID.escapeNode(username);

}

// Escape HTML tags in username to prevent cross-site scripting attacks. This

// is necessary because we display the username in the page below.

username = org.jivesoftware.util.StringUtils.escapeHTMLTags(username);

String password = ParamUtils.getParameter(request, "password");

String url = ParamUtils.getParameter(request, "url");

+   url = org.jivesoftware.util.StringUtils.escapeHTMLTags(url);

// SSO between cluster nodes

String secret = ParamUtils.getParameter(request, "secret");

String nodeID = ParamUtils.getParameter(request, "nodeID");

String nonce = ParamUtils.getParameter(request, "nonce");

// The user auth token:

AuthToken authToken;

... SNIP ...

&lt;html&gt;

&lt;head&gt;

&lt;title&gt;&lt;%= AdminConsole.getAppName() %&gt; &lt;fmt:message key="login.title" /&gt;&lt;/title&gt;

&lt;script language="JavaScript"&gt;

&lt;!--

// break out of frames

if (self.parent.frames.length != 0) {

self.parent.location=document.location;

}

function updateFields(el) {

if (el.checked) {

document.loginForm.username.disabled = true;

document.loginForm.password.disabled = true;

}

else {

document.loginForm.username.disabled = false;

document.loginForm.password.disabled = false;

document.loginForm.username.focus();

}

}

//--&gt;

&lt;/script&gt;

&lt;link rel="stylesheet" href="style/global.css" type="text/css"&gt;

&lt;link rel="stylesheet" href="style/login.css" type="text/css"&gt;

&lt;/head&gt;

&lt;body&gt;

&lt;form action="login.jsp" name="loginForm" method="post"&gt;

&lt;%  if (url != null) { try { %&gt;

&lt;input type="hidden" value="&lt;%= url %&gt;"&gt;

&lt;%  } catch (Exception e) { Log.error(e); } } %&gt;

&lt;input value="true"&gt;

&lt;div align="center"&gt;

&lt;!-- BEGIN login box --&gt;

&lt;div id="jive-loginBox"&gt;

&lt;div align="center"&gt;

&lt;span id="jive-login-header" style="background: transparent url(images/login_logo.gif) no-repeat left; padding: 29px 0 10px 205px;"&gt;

&lt;fmt:message key="admin.console" /&gt;

&lt;/span&gt;

&lt;div style="text-align: center; width: 380px;"&gt;

&lt;table cellpadding="0" cellspacing="0" border="0" align="center"&gt;

&lt;tr&gt;

&lt;td align="right"&gt;

&lt;table cellpadding="2" cellspacing="0" border="0"&gt;

&lt;noscript&gt;

&lt;tr&gt;

&lt;td colspan="3"&gt;

&lt;table cellpadding="0" cellspacing="0" border="0"&gt;

&lt;tr valign="top"&gt;

&lt;td&gt;&lt;img src="images/error-16x16.gif" width="16" height="16" border="0" alt="" vspace="2"&gt;&lt;/td&gt;

&lt;td&gt;&lt;div style="padding-left:5px; color:#cc0000;"&gt;&lt;fmt:message key="login.error" /&gt;&lt;/div&gt;&lt;/td&gt;

&lt;/tr&gt;

&lt;/table&gt;

&lt;/td&gt;

&lt;/tr&gt;

&lt;/noscript&gt;

&lt;%  if (errors.size() &gt; 0) { %&gt;

&lt;tr&gt;

&lt;td colspan="3"&gt;

&lt;table cellpadding="0" cellspacing="0" border="0"&gt;

&lt;% for (String error:errors.values()) { %&gt;

&lt;tr valign="top"&gt;

&lt;td&gt;&lt;img src="images/error-16x16.gif" width="16" height="16" border="0" alt="" vspace="2"&gt;&lt;/td&gt;

&lt;td&gt;&lt;div style="padding-left:5px; color:#cc0000;"&gt;&lt;%= error%&gt;&lt;/div&gt;&lt;/td&gt;

&lt;/tr&gt;

&lt;% } %&gt;

&lt;/table&gt;

&lt;/td&gt;

&lt;/tr&gt;

&lt;%  } %&gt;

&lt;tr&gt;

&lt;td&gt;&lt;input size="15" maxlength="50" value="&lt;%= (username != null ? username : "") %&gt;"&gt;&lt;/td&gt;

&lt;td&gt;&lt;input size="15" maxlength="50"&gt;&lt;/td&gt;

&lt;td align="center"&gt;&lt;input value="&amp;nbsp; &lt;fmt:message key="login.login" /&gt; &amp;nbsp;"&gt;&lt;/td&gt;

&lt;/tr&gt;

&lt;tr valign="top"&gt;

&lt;td&gt;&lt;label for="u01"&gt;&lt;fmt:message key="login.username" /&gt;&lt;/label&gt;&lt;/td&gt;

&lt;td&gt;&lt;label for="p01"&gt;&lt;fmt:message key="login.password" /&gt;&lt;/label&gt;&lt;/td&gt;

&lt;td&gt;&amp;nbsp;&lt;/td&gt;

&lt;/tr&gt;

&lt;/table&gt;

&lt;/td&gt;

&lt;/tr&gt;

&lt;tr&gt;

&lt;td align="right"&gt;

&lt;div align="right"&gt;

&lt;%= AdminConsole.getAppName() %&gt;, &lt;fmt:message key="login.version" /&gt;: &lt;%= AdminConsole.getVersionString() %&gt;

&lt;/div&gt;

&lt;/td&gt;

&lt;/tr&gt;

&lt;/table&gt;

&lt;/div&gt;

&lt;/div&gt;

&lt;/div&gt;

&lt;!-- END login box --&gt;

&lt;/div&gt;

&lt;/form&gt;

&lt;script type="text/javascript"&gt;

&lt;!--

if (document.loginForm.username.value == '')  {

document.loginForm.username.focus();

} else {

document.loginForm.password.focus();

}

//--&gt;

&lt;/script&gt;

&lt;/body&gt;

&lt;/html&gt;

```
