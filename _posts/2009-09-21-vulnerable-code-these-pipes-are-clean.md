---
layout: post
title: These Pipes are Clean
tags:
- Code Snippet
- PHP
status: publish
quote: I believe that professional wrestling is clean and everything else in the world is fixed
quote_author: Frank Deford
type: post
published: true
meta:
  _aioseop_description: What's wrong with this code snippet?
  _aioseop_title: Vulnerable PHP Code
  aktt_tweeted: '1'
  _edit_last: '1'
  _aktt_hash_meta: ''
  aktt_notify_twitter: 'yes'
  _headspace_page_title: Vulnerable Source Code
  _headspace_description: Can you identify the security vulnerability in this code
    snippet?
  _sexybookmarks_permaHash: 0896e6c87af7910c6785096d54d7a8ea
  _sexybookmarks_shortUrl: http://bit.ly/b5vUAz
---
The code snippet shown below has a security vulnerability.

__Can you spot the vulnerability in this piece of code?__ If so, feel free to leave a comment. None of the comments will be shown until Friday, to prevent spoilers.

If you are a development manager or an instructor you can integrate these security source code challenges into your development program or your curriculum.

## Vulnerable Code Snippet

{% highlight html+php linenos %}

function clean_url( $url, $protocols = null, $context = 'display' ) {
$original_url = $url;

if ('' == $url) return $url;
$url = preg_replace('|[^a-z0-9-~+_.?#=!&amp;;,/:%@$\|*\'()\\x80-\\xff]|i', '', $url);
$strip = array('%0d', '%0a');
$url = str_replace($strip, '', $url);
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
$url = str_replace( "'", '', $url );
}

if ( !is_array($protocols) )
$protocols = array('http', 'https', 'ftp', 'ftps', 'mailto', 'news', 'irc', 'gopher', 'nntp', 'feed', 'telnet');
if ( wp_kses_bad_protocol( $url, $protocols ) != $url )
return '';

return apply_filters('clean_url', $url, $original_url, $context);
}

{% endhighlight %}