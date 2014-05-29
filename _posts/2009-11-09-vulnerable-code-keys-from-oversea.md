---
layout: post
title: Ki's From Overseas
tags:
- Code Snippet
- Java
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#code #development'
  _edit_last: '2'
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 1ab35111f7a2fd5c0a2f703bcf833ea8
  _sexybookmarks_shortUrl: http://bit.ly/26615n
---
<blockquote>I got ki's, comin from overseas...
- Sykes</blockquote>
[ccnLe_java]

&lt;% if( usertext == null ) usertext = ""; %&gt;
&lt;%
String action = "comment".equals(request.getParameter("action")) ?
context.getURL(WikiContext.COMMENT,context.getName()) :
context.getURL(WikiContext.EDIT,context.getName());
%&gt;
&lt;form accept-charset="&lt;wiki:ContentEncoding/&gt;" method="post"
action="&lt;%=action%&gt;"
name="editForm" enctype="application/x-www-form-urlencoded"&gt;
&lt;p&gt;
&lt;%-- Edit.jsp &amp; Comment.jsp rely on these being found. So be careful, if you make changes. --%&gt;
&lt;input name="author" type="hidden" value="&lt;%=session.getAttribute("author")%&gt;" /&gt;
&lt;input name="link" type="hidden" value="&lt;%=session.getAttribute("link")%&gt;" /&gt;
&lt;input name="remember" type="hidden" value="&lt;%=session.getAttribute("remember")%&gt;" /&gt;

&lt;input name="page" type="hidden" value="&lt;wiki:Variable var="pagename"/&gt;" /&gt;
&lt;input name="action" type="hidden" value="save" /&gt;
&lt;input name="edittime" type="hidden" value="&lt;%=pageContext.getAttribute("lastchange",
PageContext.REQUEST_SCOPE )%&gt;" /&gt;
&lt;input name="addr" type="hidden" value="&lt;%=request.getRemoteAddr()%&gt;" /&gt;

[/ccnLe_java]
