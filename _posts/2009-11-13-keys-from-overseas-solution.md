---
layout: post
tags:
- Cross-Site Scripting (XSS)
- Java
- JSPWiki
- Solution
---

# Solution: Key's from overseas

## Details

* __Affected Software:__ JSPWiki
* __Fixed in Version:__  2.60
* __Issue Type:__ Cross-Site Scripting

## Description
The vulnerable code from JSPWiki contained persistent Cross-Site Scripting vulnerabilities. Although we cannot see the call to session.setAttribute(), we can see that the author, link, and remember values were not sanitized before being used in FORM input fields. The exposure was fixed by using "c:out". c:out is part of the JSP Standard Tag Library (JSTL) and is used to output to the current JspWriter. If the "escapeXML" attribute is set to "true" (which is the default value), c:out will escape some HTML characters, helping to reduce the possibility of XSS. By default, c:out encodes the following characters: <, >, &, ", and '.

This week's code sample brings up the old application security argument as to whether it's better to sanitize input as it arrives or to escape data at the point of consumption (or as near to the point of consumption as possible). There are pros and cons for both methods. In this case, the JSPWiki developers chose to encode at the point of consumption. If the author, link, and remember values were instead sanitized when being assigned to the session store, the developers would have also prevented the XSS exposure... so which is better?

From a security code review perspective, encoding data at the point of entry requires additional flow analysis, as the code reviewer has to follow the data from the point of consumption back to assignment in order to ensure it hasn't been tainted along the way before being consumed. Sanitizing input as it arrives can be challenging as it may be difficult to determine how best to sanitize incoming data if that data is to be used in diverse ways throughout the application (inserted into a database, output to HTML, used in LDAP queries, used as part of a mail API...etc.). This typically results in more generic forms of sanitization routines which may not stand up to targeted or sophisticated attacks. Sanitizing data as it arrives does offer the advantage of being able to sanitize a particular piece of data once, as opposed to tracking down each instance of consumption and sanitizing at the points of consumption.

Sanitizing at the point of consumption offers advantages as the developer has a clear understanding as to how the data will be used (inserted into a database, output to HTML, used in LDAP queries, used as part of a mail API...etc.) and can implement the best sanitization routine for the situation. From a security code review perspective, code that sanitizes at the point of consumption requires less flow analysis as each consumption point can be evaluated independently of other consumption points. This independence comes at a cost as each point of consumption must now be evaluated in order to ensure the proper sanitization is being put in place. A single missed consumption point will likely result in the introduction of a vulnerability in the application.

It's also interesting seeing the vulnerable code sample has comments related to Edit.jsp and Comment.jsp. It seems the conditionals for these two JSP files may not be very robust and could be an interesting avenue for attack. I'm also curious as to whether this particular FORM has a robust CSRF token...

## Developer's Solution

```diff
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
<%-- Edit.jsp & Comment.jsp rely on these being found. So be careful, if you make changes. --%>
-<input name="author" type="hidden" value="<%=session.getAttribute("author")%>" />
-<input name="link" type="hidden" value="<%=session.getAttribute("link")%>" />
-<input name="remember" type="hidden" value="<%=session.getAttribute("remember")%>" />
+<input type="hidden" name="author" value="<c:out value='${author}' />" />
+<input type="hidden" name="link" value="<c:out value='${link}' />" />
+    <input type="hidden" name="remember" value="<c:out value='${remember}' />" />

<input name="page" type="hidden" value="<wiki:Variable var="pagename"/>" />
<input name="action" type="hidden" value="save" />
-        <input name="edittime" type="hidden" value="<%=pageContext.getAttribute("lastchange",
PageContext.REQUEST_SCOPE )%>" />
-        <input name="addr" type="hidden" value="<%=request.getRemoteAddr()%>" />
+    <input name="<%=SpamFilter.getHashFieldName(request)%>" type="hidden" value="<c:out value='${lastchange}' />" />

```
