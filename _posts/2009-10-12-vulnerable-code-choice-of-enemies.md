---
layout: post
title: Choice of Enemies
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Vulnerable Source Code
  aktt_tweeted: '1'
  _headspace_description: Can you identify the security vulnerability in this code
    snippet?
  _sexybookmarks_shortUrl: http://bit.ly/cPOUFY
  _sexybookmarks_permaHash: 63f093582dfe0d4d4eda9d3f88044ed1
---
<blockquote>I choose my friends for their good looks, my acquaintances for their good characters, and my enemies for their good intellects. A man cannot be too careful in the choice of his enemies.
- Oscar Wilde (1854 - 1900), The Picture of Dorian Gray, 1891</blockquote>

## Vulnerable Code

[ccnLe_php]


/**
* Removes characters from the username
*
* If $strict is true, only alphanumeric characters (as well as _, space, ., -, @) are returned.
*
* @since 2.0.0
*
* @param string $username The username to be sanitized.
* @param bool $strict If set limits $username to specific characters. Default false.
* @return string The sanitized username, after passing through filters.
*/
function sanitize_user( $username, $strict = false ) {
   $raw_username = $username;
   $username = strip_tags($username);
   // Kill octets
   $username = preg_replace('|%([a-fA-F0-9][a-fA-F0-9])|', '', $username);
   $username = preg_replace('/&amp;.+?;/', '', $username); // Kill entities

   // If strict, reduce to ASCII for max portability.
   if ( $strict )
   $username = preg_replace('|[^a-z0-9 _.\-@]|i', '', $username);

   return apply_filters('sanitize_user', $username, $raw_username, $strict);
}
[/ccnLe_php]
