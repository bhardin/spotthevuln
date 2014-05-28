---
layout: post
tags: Code Snippet, PHP
---

# These Pipes are Clean

> I believe that professional wrestling is clean and everything else in the world is fixed.
>
> - Frank Deford

## Vulnerable Code Snippet

```php
<?php
function clean_url($url, $protocols = null, $context = 'display')
{
  $original_url = $url;
  if ('' == $url) return $url;
  $url = preg_replace('|[^a-z0-9-~+_.?#=!&amp;;,/:%@$\|*\'()\\x80-\\xff]|i', '', $url);
  $strip = array(
    '%0d',
    '%0a'
  );
  $url = str_replace($strip, '', $url);
  $url = str_replace(';//', '://', $url);

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
?>
```