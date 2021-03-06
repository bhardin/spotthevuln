---
layout: post
tags:
- Solution
- SQL Injection
- Wordpress
---

# Solution: Choice of Enemies

## Details

* __Affected Software:__ Wordpress
* __Fixed in Version:__ 2.6.2
* __Issue Type:__ SQL Injection through MySQL Column Truncation

## Description

This was a tricky SQL Injection through truncation issue which affected WordPress installations using MySQL back ends - the majority of WordPress installations.

When a user is registered inside WordPress, the `username` value undergoes some sanitizing/validating before being inserted into the WordPress database. One of the routines that validates usernames is `sanitize_user()`. Inside of the `sanitize_user()` function, WordPress attempts to sanitize the username by stripping certain characters (entities and octets).

The sanitization routine failed to properly validate the length of the username and whether whitespace characters were included. Before the application allows the newly created username to be inserted into the database, it first checked to see if that username already existed in the WordPress database. The default comparison operation done by MySQL ignored whitespace so when WordPress issues a query for a username of `"admin   "` (admin followed by four spaces), it would match a query for `"admin"` (admin followed by no spaces). The default data type for the WordPress username column also limited the column length to 16 characters.

Knowing this, an attacker could create a new user account named `"admin           x"` (admin followed by eleven spaces and an arbitrary character, resulting in 17 characters total). When the application would query the WordPress database to check to see if this particular username had been taken, it would indicate that this particular username was not in use. However, when the application attempted to insert the newly created username into the WordPress database, the last character in the username would be truncated by the MySQL database because the username column had a maximum length of 16 and the username being inserted had a length of 17. In a default installation of MySQL, _this would NOT raise an error_.

The truncated admin username allows for the creation of two accounts named "admin" in the WordPress user database. One admin account is attacker controlled and the other belongs to the actual admin of the WordPress installation. An exploitable condition arises when security checks are done against one admin account (the attacker account, in which the attacker knows all the account details), but later serve data/content from the other admin account (data from the actual WordPress administrator), giving the attacker the ability to hijack the WordPress administrator account. For more information on SQL truncation read [Stefan Esser's MySQL truncation issues](http://www.suspekt.org/2008/08/18/mysql-and-sql-column-truncation-vulnerabilities/).

Interestingly, WordPress had a conditional which would reduce the username to only ASCII characters in place before this issue was discovered and exploited in the wild. This conditional would have likely eliminated this vulnerability; however this conditional is disabled by default. Also, the code fix implemented by the WordPress team did not validate length (only stripped whitespace), keeping the possibility of truncation issues open (ie. truncation issues against users with usernames of 16 characters).

## Developer's Solution

```diff
<?php

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

+  // Consolidate contiguous whitespace
+  $username = preg_replace('|\s+|', ' ', $username);

   return apply_filters('sanitize_user', $username, $raw_username, $strict);
}
```

