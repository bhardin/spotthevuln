---
layout: post
title: Ki's From Overseas
tags:
- Code Snippet
- Java
---

# Keys from Overseas

> I got ki's, comin from overseas...

>- Sykes

```java
<% if( usertext == null ) usertext = ""; %>
<%
String action = "comment".equals(request.getParameter("action")) ?
context.getURL(WikiContext.COMMENT,context.getName()) :
context.getURL(WikiContext.EDIT,context.getName());
%>
<form accept-charset="<wiki:ContentEncoding/>" method="post"
action="<%=action%>"
name="editForm" enctype="application/x-www-form-urlencoded">
<p>
<%-- Edit.jsp &amp; Comment.jsp rely on these being found. So be careful, if you make changes. --%>
<input name="author" type="hidden" value="<%=session.getAttribute("author")%>" />
<input name="link" type="hidden" value="<%=session.getAttribute("link")%>" />
<input name="remember" type="hidden" value="<%=session.getAttribute("remember")%>" />

<input name="page" type="hidden" value="<wiki:Variable var="pagename"/>" />
<input name="action" type="hidden" value="save" />
<input name="edittime" type="hidden" value="<%=pageContext.getAttribute("lastchange",
PageContext.REQUEST_SCOPE )%>" />
<input name="addr" type="hidden" value="<%=request.getRemoteAddr()%>" />
```
