---
layout: solution
tags:
- Carriage Return/Line Feed (CRLF) Injection
- Cross-Site Scripting (XSS)
- HTTP header injection
- URL Redirection
---

# Solution: These Pipes are Clean

## Details

* __Vulnerability Type__: URL Redirection, CRLF Injection, HTTP Header Injection, Cross-Site Scripting (XSS)
* __Affected Software:__ Wordpress
* __Fixed in Version:__  2.8.1

## Description
When appended together the `%0d` (Carriage Return) and `%0a` (Line Feed) characters represent a Carriage Return Line Feed (CRLF).

The vulnerable WordPress code snippet actually contained logic to detect carriage return and line feed characters. The logic was attempting to strip CRLFs from data being assigned to the `$url` variable.

However, The CRLF detection logic wasn't robust enough. The CRLF detection logic simply checked for the presence of `%0d` and `%0a` in `$url` and failed to consider uppercase versions of these control characters: `%0D` or `%0A`.  The `$url` variable is assigned the CRLF tainted string and eventually passed to a HTTP `Location` header, giving the attacker an opportunity for URL Redirection, CRLF injection, HTTP header injection, and even Cross-Site Scripting.

In addition to adding uppercase variants of `%0D` and `%0A` to the detection logic, a function to recursively detect the presence of CRLF was also added.

Before this function was added, it was possible to defeat the detection logic by simply passing a string such as `%0%0d%0ad%0%0d%0aa` which would have the `%0d%0a` character sequences stripped out, resulting in `%0d%0a` being passed to the `$url` variable. The WordPress developers addressed this issue by adding a recursive verifier function `_deep_replace()`, whose source is included in the Developers Solution.

## Developers Solution
```diff

<?php
function clean_url($url, $protocols = null, $context = 'display')
{
  $original_url = $url;
  if ('' == $url) return $url;
  $url = preg_replace('|[^a-z0-9-~+_.?#=!&amp;;,/:%@$\|*\'()\\x80-\\xff]|i', '', $url);

- $strip = array('%0d', '%0a');
- $url = str_replace($strip, '', $url);
- $url = str_replace(';//', '://', $url);
+ $strip = array('%0d', '%0a', '%0D', '%0A');
+ $url = _deep_replace($strip, $url);

  /* If the URL doesn't appear to contain a scheme, we
  * presume it needs http:// appended (unless a relative
  * link starting with / or a php file). */
  if (strpos($url, ':') === false & amp; & amp;
  substr($url, 0, 1) != '/' & amp; & amp;
  substr($url, 0, 1) != '#' & amp; & amp;
  !preg_match('/^[a-z0-9-]+?\.php/i', $url)) $url = 'http://' . $url;
  // Replace ampersands and single quotes only when displaying.
  if ('display' == $context) {
    $url = preg_replace('/&amp;([^#])(?![a-z]{2,8};)/', '&amp;$1', $url);
    $url = str_replace("'", '', $url);
  }
  if (!is_array($protocols)) $protocols = array(
    'http',
    'https',
    'ftp',
    'ftps',
    'mailto',
    'news',
    'irc',
    'gopher',
    'nntp',
    'feed',
    'telnet'
  );
  if (wp_kses_bad_protocol($url, $protocols) != $url) return '';
  return apply_filters('clean_url', $url, $original_url, $context);
}

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
+function _deep_replace($search, $subject) {
+  $found = true;
+  while ($found) {
+    $found = false;
+    foreach((array)$search as $val) {
+      while (strpos($subject, $val) !== false) {
+        $found = true;
+        $subject = str_replace($val, '', $subject);
+      }
+    }
+  }
+
+  return $subject;
+}
```
