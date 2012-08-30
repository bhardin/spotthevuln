---
layout: post
title: CaddyShack - Cross Site Scripting
tags:
- All Vulnerabilities
- attacker
- cross site scripting
- Cross-Site Scripting (XSS)
- Java
- jsp code
- Openfire
- security code
- Solution
- vulnerabilities
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _aktt_hash_meta: ''
  aktt_notify_twitter: 'yes'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/aPprIc
  _sexybookmarks_permaHash: 3b3c82ee37a34df70955fa848e34c1ec
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
---
## Details
__Affected Software:__ WebChat Module for Jive

__Fixed in Version:__  August of 2008

__Issue Type:__ Cross Site Scripting

Original Code: <a title="Caddyshack" href="http://spotthevuln.com/2010/08/caddyshack/" target="_blank">Found    Here</a>
## Description
This week's vulnerability affected a webchat module created by Jive Software.  The bug is straightforward,  the JSP code takes an attacker controlled value and uses it to build dynamic HTML.  Although the bug is straightforward, this week's example was a great/simple exercise in identifying a vulnerable pattern and tracing to find other vulnerable patterns in the code.  This week's sample has three separate vulnerabilities that were all addressed via single patch.  All these have similar symptoms/patterns (although the specifics are a bit different).  Identifying vulnerable patterns and searching for these patterns in other places in code is an essential skill for security code auditors.  Did you find all three bugs that were patched?
<h2>Developers Solution</h2>
[cce lang="diff"] public class FormUtils {

private FormUtils() {
}

public static String createAnswers(FormField formField, HttpServletRequest request) {
final StringBuffer builder = new StringBuffer();
if (formField.getType().equals(FormField.TYPE_TEXT_SINGLE)) {
String cookieValue = getCookieValueForField(formField.getVariable(), request);
String insertValue = "";
if(ModelUtil.hasLength(cookieValue)){
insertValue = "value=\""+cookieValue+"\"";
}
- builder.append("&lt;input type=\"text\" name=\"" + formField.getVariable() + "\" "+insertValue+" style=\"width:75%\"&gt;");
+builder.append("&lt;input type=\"text\" name=\"" + formField.getVariable() + "\" "+StringUtils.escapeHTMLTags(insertValue)+" style=\"width:75%\"&gt;");
}
else if (formField.getType().equals(FormField.TYPE_TEXT_MULTI)) {
builder.append("&lt;textarea name=\"" + formField.getVariable() + "\" cols=\"30\" rows=\"3\"&gt;");
builder.append("&lt;/textarea&gt;");
}
else if (formField.getType().equals(FormField.TYPE_LIST_SINGLE)) {
builder.append("&lt;select name=\"" + formField.getVariable() + "\" &gt;");
Iterator iter = formField.getOptions();
String cookieValue = ModelUtil.emptyStringIfNull(getCookieValueForField(formField.getVariable(), request));
while (iter.hasNext()) {
FormField.Option option = (FormField.Option)iter.next();
String selected = option.getValue().equals(cookieValue) ? "selected" : "";
- builder.append("&lt;option value=\"" + option.getValue() + "\" "+selected+"&gt;" + option.getLabel() + "&lt;/option&gt;");
+builder.append("&lt;option value=\"" + StringUtils.escapeHTMLTags(option.getValue()) + "\" "+selected+"&gt;" + option.getLabel() + "&lt;/option&gt;");
}
builder.append("&lt;/select&gt;");
}
else if (formField.getType().equals(FormField.TYPE_BOOLEAN)) {
Iterator iter = formField.getOptions();
int counter = 0;
while (iter.hasNext()) {
FormField.Option option = (FormField.Option)iter.next();
String value = option.getLabel();
builder.append("&lt;input type=\"checkbox\" value=\"" + value + "\" name=\"" + formField.getVariable() + counter + "\"&gt;");
builder.append("&amp;nbsp;");
-builder.append(value);
+builder.append(StringUtils.escapeHTMLTags(value));
builder.append("&lt;br/&gt;");
counter++;
}
}
[/cce] 
