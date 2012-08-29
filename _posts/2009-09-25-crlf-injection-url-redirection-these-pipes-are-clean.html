---
layout: post
title: These Pipes are Clean - CRLF
tags:
- Carriage Return/Line Feed (CRLF) Injection
- Cross-Site Scripting (XSS)
- HTTP header injection
- Solution
- URL Redirection
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_tweeted: '1'
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Carriage Return Line-Feed (CRLF) Vulnerable Code Example
  _headspace_description: Carriage Return Line Feed Vulnerability Code Example
  _sexybookmarks_permaHash: 7369dd991e98a22b00a63eb0be973734
  _sexybookmarks_shortUrl: http://bit.ly/caO0rg
---
## Details
__Affected Software:__ <a href="http://spotthevuln.com/category/software/wordpress/">Wordpress</a>

__Fixed in Version:__  2.8.1

<strong>__Issue Type:__</strong> <a href="http://spotthevuln.com/category/vulnerability/URL-redirection/">URL Redirection</a> / <a href="http://spotthevuln.com/category/vulnerability/CRLF-Injection/">CRLF Injection</a> / <a href="http://spotthevuln.com/category/vulnerability/HTTP-Header-Injection/">HTTP Header Injection</a> / <a href="http://spotthevuln.com/category/vulnerability/xss/">XSS</a>

<strong>Original Code: </strong><a href="http://spotthevuln.com/2009/09/vulnerable-code-these-pipes-are-clean/">Found Here</a>
## Description
When appended together the %0d (Carriage Return) and %0a (Line Feed) characters represent a Carriage Return Line Feed (CRLF).

The vulnerable WordPress code snippet actually contained logic to detect carriage returns and line feed characters, attempting to strip CRLF from data being assigned to the $url variable.

The CRLF detection logic used by the vulnerable WordPress version was not very robust.  The CRLF detection logic simply checked for the presence of "%0d" and "%0a" in the $url variable and failed to consider UPPERCASE "%0D" or "%0A".  The $url variable is assigned the CRLF tainted string and eventually passed to a HTTP Location header, giving the attacker an opportunity for URL Redirection, CRLF injection, HTTP header injection, and even <a href="http://misc-security.com/2009/05/21/xss-cross-site-scripting/">Cross Site Scripting (XSS)</a>.

In addition to adding UPPERCASE variants of "%0D" and "%0A" to the detection logic, a function to recursively detect the presence of CRLF was also added.  Before this function was added, it was possible to defeat the detection logic by simply passing a string such as %0%0d%0ad%0%0d%0aa which would have "%0d%0a" character sequences stripped out, resulting in %0d%0a being passed to the $url variable.  The WordPress developers addressed this issue by adding a recursive verifier (_deep_request()), whose source is included in the Developers Solution.
<h2>Developers Solution</h2>
[cc lang="diff"]

function clean_url( $url, $protocols = null, $context = 'display' ) {
$original_url = $url;

if ('' == $url) return $url;
$url = preg_replace('|[^a-z0-9-~+_.?#=!&amp;;,/:%@$\|*\'()\\x80-\\xff]|i', '', $url);
-       $strip = array('%0d', '%0a');
-       $url = str_replace($strip, '', $url);
+       $strip = array('%0d', '%0a', '%0D', '%0A');
+       $url = _deep_replace($strip, $url);
$url = str_replace(';//', '://', $url);
/* If the URL doesn't appear to contain a scheme, we
* presume it needs http:// appended (unless a relative
* link starting with / or a php file).
*/
if ( strpos($url, ':') === false &amp;&amp;
substr( $url, 0, 1 ) != '/' &amp;&amp; substr( $url, 0, 1 ) != '#' &amp;&amp; !preg_match('/^[a-z0-9-]+?\.php/i', $url) )
$url = 'http://' . $url;

// Replace ampersands and single quotes only when displaying.
if ( 'display' == $context ) {
$url = preg_replace('/&amp;([^#])(?![a-z]{2,8};)/', '&amp;$1', $url);
$url = str_replace( "'", ''', $url );
}

if ( !is_array($protocols) )
$protocols = array('http', 'https', 'ftp', 'ftps', 'mailto', 'news', 'irc', 'gopher', 'nntp', 'feed', 'telnet');
if ( wp_kses_bad_protocol( $url, $protocols ) != $url )
return '';

return apply_filters('clean_url', $url, $original_url, $context);
}

With _DEEP_REPLACE for recursive checks!
+/**
+ * Perform a deep string replace operation to ensure the values in $search are no longer present
+ *
+ * Repeats the replacement operation until it no longer replaces anything so as to remove "nested" values
+ * e.g. $subject = '%0%0%0DDD', $search ='%0D', $result ='' rather than the '%0%0DD' that
+ * str_replace would return
+ *
+ * @since 2.8.1
+ * @access private
+ *
+ * @param string|array $search
+ * @param string $subject
+ * @return string The processed string
+ */
+function _deep_replace($search, $subject){
+        $found = true;
+        while($found) {
+                $found = false;
+                foreach( (array) $search as $val ) {
+                        while(strpos($subject, $val) !== false) {
+                                $found = true;
+                                $subject = str_replace($val, '', $subject);
+                        }
+                }
+        }
+
+        return $subject;
+}

[/cc] 
