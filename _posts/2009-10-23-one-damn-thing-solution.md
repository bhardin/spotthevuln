---
layout: post
title: One Damn Thing - Privilege Escalation
tags:
- Privilege Escalation
- Solution
- Wordpress
---

# Solution: One Damn Thing

## Details

* __Affected Software:__ Wordpress
* __Fixed in Version:__  2.3.2
* __Issue Type:__ Privilege Escalation

## Description

The vulnerable code here deals with the use of the `$_SERVER['PHP_SELF']` global variable. `PHP_SELF` has a few quirks that are not well understood by many developers and can easily introduce security issues in web applications. If we take a look at the PHP manual page for `$_SERVER['PHP_SELF']`, we see the following:

> The filename of the currently executing script, relative to the document root. For instance, `$_SERVER['PHP_SELF']` in a script at the address `http://example.com/test.php/foo.bar` would be `/test.php/foo.bar`.

Examining the example provided in the PHP manual page, we see that an attacker is free to specify an arbitrary value (foo.bar) after a valid php file (test.php). This arbitrary value will be included in the `$_SERVER['PHP_SELF']` global variable.

Examining the vulnerable WordPress source code, we see that the WordPress developers used the PHP `strpos()` function to check to whether the `$_SERVER['PHP_SELF']` global variable contained the string `"wp-admin/"`. If the `strpos()` function found the `"wp-admin/"` string within the `$_SERVER['PHP_SELF']` variable, it would return `TRUE` which resulted in the setting of the `"is_admin"` value to true. This ultimately granted the user administrative rights to certain portions of the web application. The attacker could easily craft a request for a valid php file, while injecting the `"wp-admin/"` string into the `$_SERVER['PHP_SELF']`. Such a request would look something like this:

http://wordpressblog.com/index.php/wp-admin/

The request above will request index.php from the server while setting the `$_SERVER['PHP_SELF']` variable to `/index.php/wp-admin/`. This tricks the vulnerable WordPress code into assuming the user is an administrator and grants the user administrative access to certain pages on the blog.

The WordPress developers addressed this vulnerability by removing the faulty checks and adding a function which was designed to determine whether the user has administrative privilege.

## Developer's Solution

```diff
<?

if ('' != $qv['tb']) $this->is_trackback = true;
if ('' != $qv['paged']) $this->is_paged = true;
if ('' != $qv['comments_popup']) $this->is_comments_popup = true;

// if we're previewing inside the write screen

if ('' != $qv['preview']) $this->is_preview = true;

- if ( strpos($_SERVER['PHP_SELF'], 'wp-admin/') !== false )
+ if ( is_admin() )
  $this->is_admin = true;

if (false !== strpos($qv['feed'], 'comments-')) {
  $qv['feed'] = str_replace('comments-', '', $qv['feed']);
  $qv['withcomments'] = 1;
}

$this->is_singular = $this->is_single || $this->is_page || $this->is_attachment;

if ($this->is_feed && (!empty($qv['withcomments']) || (empty($qv['withoutcomments']) && $this->is_singular))) $this->is_comment_feed = true;

if (!($this->is_singular || $this->is_archive || $this->is_search || $this->is_feed || $this->is_trackback || $this->is_404 || $this->is_admin || $this->is_comments_popup)) $this->is_home = true;

// Correct is_* for page_on_front and page_for_posts

if ($this->is_home && (empty($this->query) || $qv['preview'] == 'true') && 'page' == get_option('show_on_front') && get_option('page_on_front')) {
  $this->is_page = true;
  $this->is_home = false;
  $qv['page_id'] = get_option('page_on_front');
}
```
