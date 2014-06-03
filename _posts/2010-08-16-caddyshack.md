---
layout: post
title: CaddyShack
tags:
- Code Snippet
- Java
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/dxnXrN
  _sexybookmarks_permaHash: c6a72486c62c972e86fad87d088a7bc4
---
<blockquote><strong>I asked Dalai Lama the most important question that I think you could ask - if he had ever seen Caddyshack.</strong>
<strong> -Jesse Ventura
</strong></blockquote>
```java
public class FormUtils {

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
builder.append("<input name="\&quot;&quot;" type="\&quot;text\&quot;" />");
}
else if (formField.getType().equals(FormField.TYPE_TEXT_MULTI)) {
builder.append("<textarea cols="\&quot;30\&quot;" rows="\&quot;3\&quot;" name="\&quot;&quot;">");&lt;br /&gt; builder.append("</textarea>");
}
else if (formField.getType().equals(FormField.TYPE_LIST_SINGLE)) {
builder.append("<select name="\&quot;&quot;"> <option value="\&quot;&quot;">" + option.getLabel() + "</option</select>");
}
else if (formField.getType().equals(FormField.TYPE_BOOLEAN)) {
Iterator iter = formField.getOptions();
int counter = 0;
while (iter.hasNext()) {
FormField.Option option = (FormField.Option)iter.next();
String value = option.getLabel();
builder.append("<input name="\&quot;&quot;" type="\&quot;checkbox\&quot;" value="\&quot;&quot;" />");
builder.append(" ");
builder.append(value);
builder.append(" ");

counter++;
}
}
```
