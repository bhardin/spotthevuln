---
layout: post
title: Flag - Cross-Site Scripting
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '3'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Drupal Core

__Fixed in Version:__  6.1

__Issue Type:__ Cross-Site Scripting

Original Code: <a href="http://spotthevuln.com/2011/03/flag/">Found Here</a>
## Details
This week's Cross-Site Scripting vulnerability is somewhat unusual in that it exists in javascript, rather than server side. The checkPlain function is used to output encode data fetched via Ajax/XHR (for instance, dynamically loading a new article). It seems to do the job, however the String.replace function in javascript only replaces the first instance by default; any additional instances of the character will remain intact.

The developer's fix is to set the global flag on the regex, so that all instances are replaced. When auditing code like this it would be wise carefully look upstream and check for other uses of the same data, where a different end use wasn't encoded. When designing a system, issues like this indicate the importance of a carefully planned and consistent input-escaping/output-encoding approach, so that missed occurrences are more apparent. In this case, the JS function here is used only by Drupal modules and plugins loading data via Ajax, and a parallel change was made in the Drupal PHP source to handle normal usage. That change called the standard PHP function check_plain to output encode the data on the back end.
## Developer's Solution
[sourcecode language="diff"]
/**
 * Encode special characters in a plain-text string for display as HTML.
 */
Drupal.checkPlain = function(str) {
  str = String(str);
  var replace = { '&amp;': '&amp;amp;', '&quot;': '&amp;quot;', '&lt;': '&amp;lt;', '&gt;': '&amp;gt;' };
  for (var character in replace) {
-    str = str.replace(character, replace[character]);
+    var regex = new RegExp(character, 'g');
+    str = str.replace(regex, replace[character]);
  }
  return str;
};
[/sourcecode]
