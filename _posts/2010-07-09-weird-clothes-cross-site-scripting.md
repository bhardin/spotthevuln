---
layout: post
title: Weird Clothes - Cross Site Scripting
tags:
- attacker
- bugs
- collaboration server
- Cross-Site Scripting (XSS)
- Java
- JSP
- Openfire
- search parameter
- Solution
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 3bf2a61107a52bfdd0fcd726eb8805ab
  _sexybookmarks_shortUrl: http://bit.ly/a0foeK
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
__Affected Software:__ Openfire by Ignite Realtime

__Fixed in Version:__  3.6.3

__Issue Type:__ XSS

Original Code: <a title="Weird Clothes" href="http://spotthevuln.com/2010/07/weird-clothes/" target="_blank">Found  Here</a>
## Description
XSS bug in Openfire by Ignite Realtime. Openfire is an Open Source, real time collaboration server. The bug is very straightforward and a simple string like the one presented below takes advantage of the vulnerability.
<blockquote>http://www.example.com/group-summary.jsp?search=%22%3E%3C[xss]</blockquote>
This bug was actually part of a number of security bugs reported by Core Security Technologies. You can read their advisory <a title="Core" href="http://www.coresecurity.com/content/openfire-multiple-vulnerabilities" target="_blank">here</a>.

The patch simply HTML encodes the tainted search parameter...
## Developers Solution
[cce lang="diff"]

&lt;%  // Get parameters
int start = ParamUtils.getIntParameter(request,"start",0);
int range = ParamUtils.getIntParameter(request,"range",webManager.getRowsPerPage("group-summary", 15));

if (request.getParameter("range") != null) {
webManager.setRowsPerPage("group-summary", range);
}

int groupCount = webManager.getGroupManager().getGroupCount();
Collection&lt;Group&gt; groups = webManager.getGroupManager().getGroups(start, range);

String search = null;
if (webManager.getGroupManager().isSearchSupported() &amp;&amp; request.getParameter("search") != null
&amp;&amp; !request.getParameter("search").trim().equals(""))
{
search = request.getParameter("search");
+    // Santize variables to prevent vulnerabilities
+    search = StringUtils.escapeHTMLTags(search);

// Use the search terms to get the list of groups and group count.
groups = webManager.getGroupManager().search(search, start, range);
// Get the count as a search for *all* groups. That will let us do pagination even
// though it's a bummer to execute the search twice.
groupCount = webManager.getGroupManager().search(search).size();
}

// paginator vars
int numPages = (int)Math.ceil((double)groupCount/(double)range);
int curPage = (start/range) + 1;
%&gt;

&lt;%  if (request.getParameter("deletesuccess") != null) { %&gt;

&lt;div class="jive-success"&gt;
&lt;table cellpadding="0" cellspacing="0" border="0"&gt;
&lt;tbody&gt;
&lt;tr&gt;&lt;td class="jive-icon"&gt;&lt;img src="images/success-16x16.gif" width="16" height="16" border="0" alt=""&gt;&lt;/td&gt;
&lt;td class="jive-icon-label"&gt;
&lt;fmt:message key="group.summary.delete_group" /&gt;
&lt;/td&gt;&lt;/tr&gt;
&lt;/tbody&gt;
&lt;/table&gt;
&lt;/div&gt;&lt;br&gt;

&lt;%  } %&gt;

&lt;% if (webManager.getGroupManager().isSearchSupported()) { %&gt;

&lt;form action="group-summary.jsp" method="get" name="searchForm"&gt;
&lt;table border="0" width="100%" cellpadding="0" cellspacing="0"&gt;
&lt;tr&gt;
&lt;td valign="bottom"&gt;
&lt;fmt:message key="group.summary.total_group" /&gt; &lt;b&gt;&lt;%= groupCount %&gt;&lt;/b&gt;
&lt;%  if (numPages &gt; 1) { %&gt;

, &lt;fmt:message key="global.showing" /&gt; &lt;%= LocaleUtils.getLocalizedNumber(start+1) %&gt;-&lt;%= LocaleUtils.getLocalizedNumberstartrange &gt; groupCount ? groupCount:start+range) %&gt;

&lt;%  } %&gt;
&lt;/td&gt;
&lt;td align="right" valign="bottom"&gt;
&lt;fmt:message key="group.summary.search" /&gt;: &lt;input type="text" size="30" maxlength="150" name="search" value="&lt;%= ((search!=null) ? search : "") %&gt;"&gt;
&lt;/td&gt;
&lt;/tr&gt;
&lt;/table&gt;
&lt;/form&gt;

&lt;script language="JavaScript" type="text/javascript"&gt;
document.searchForm.search.focus();
&lt;/script&gt;

&lt;% }
// Otherwise, searching is not supported.
else {
%&gt;
&lt;p&gt;
&lt;fmt:message key="group.summary.total_group" /&gt; &lt;b&gt;&lt;%= groupCount %&gt;&lt;/b&gt;
&lt;%  if (numPages &gt; 1) { %&gt;

, &lt;fmt:message key="global.showing" /&gt; &lt;%= (start+1) %&gt;-&lt;%= (start+range) %&gt;

&lt;%  } %&gt;
&lt;/p&gt;
&lt;% } %&gt;

&lt;%  if (numPages &gt; 1) { %&gt;

&lt;p&gt;
&lt;fmt:message key="global.pages" /&gt;
[
&lt;%  for (int i=0; i&lt;numPages; i++) {
String sep = ((i+1)&lt;numPages) ? " " : "";
boolean isCurrent = (i+1) == curPage;
%&gt;
&lt;a href="group-summary.jsp?start=&lt;%= (i*range) %&gt;&lt;%= search!=null? "&amp;search=" + URLEncoder.encode(search, "UTF-8") : ""%&gt;"
class="&lt;%= ((isCurrent) ? "jive-current" : "") %&gt;"
&gt;&lt;%= (i+1) %&gt;&lt;/a&gt;&lt;%= sep %&gt;

&lt;%  } %&gt;
]
&lt;/p&gt;

[/cce]
