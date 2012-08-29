---
layout: post
title: One Damn Thing - Privilege Escalation
tags:
- Privilege Escalation
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #solution #wordpress'
  _edit_last: '1'
  _headspace_page_title: Privilege Escalation PHP Code Example
  _headspace_description: Privilege Escalation Vulnerable Code Example
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 970f6efa27d6aefddc266fa4f59484f3
  _sexybookmarks_shortUrl: http://bit.ly/qDnOz
  _oembed_316cbb281543d7eaa0869f20b40fb4de: ! '{{unknown}}'
  _oembed_0892fd3773100b8a1ab842f0ca5020b2: ! '{{unknown}}'
  _oembed_bfa1dd76ea0130a620fca0dc7714ba89: ! '{{unknown}}'
---
## Details
__Affected Software:__ <a title="Wordpress bugs" href="http://spotthevuln.com/category/software/wordpress/" target="_blank">Wordpress</a>

__Fixed in Version:__  2.3.2

<strong>__Issue Type:__</strong> Privilege Escalation

<strong>Original Code: </strong><a title="Vulnerable Code" href="http://spotthevuln.com/2009/10/vulnerable-code-one-damn-thing/" target="_blank">Found Here</a>
## Description
The vulnerable code here deals with the use of the $_SERVER['PHP_SELF'] global variable.  PHP_SELF has a few quirks that are not well understood by many developers and can easily introduce security issues in web applications.  If we take a look at the PHP manual page for $_SERVER['PHP_SELF'], we see the following:
<blockquote>The filename of the currently executing script, relative to the document root. For instance, <var>$_SERVER['PHP_SELF']</var> in a script at the address <var>http://example.com/test.php/foo.bar</var> would be <var>/test.php/foo.bar</var>.</blockquote>
Examining the example provided in the PHP manual page, we see that an attacker is free to specify an arbitrary value (foo.bar) after a valid php file (test.php).  This arbitrary value will be included in the $_SERVER['PHP_SELF'] global variable.

Examining the vulnerable WordPress source code, we see that the WordPress developers used the PHP strpos() function to check to whether the $_SERVER['PHP_SELF'] global variable contained the string "wp-admin/".  If the strpos() function found the "wp-admin/" string within the $_SERVER['PHP_SELF'] variable, it would return TRUE which resulted in the setting of the "is_admin" value to true.  This ultimately granted the user administrative rights to certain portions of the web application.  The attacker could easily craft a request for a valid php file, while injecting the "wp-admin/" string into the $_SERVER['PHP_SELF'].  Such a request would look something like this:

http://wordpressblog.com/index.php/wp-admin/

The request above will request index.php from the server while setting the $_SERVER['PHP_SELF'] variable to /index.php/wp-admin/.  This tricks the vulnerable WordPress code into assuming the user is an administrator and grants the user administrative access to certain pages on the blog.

The WordPress developers addressed this vulnerability by removing the faulty checks and adding a function which was designed to determine whether the user has administrative privilege.
<h2>Developers Solution</h2>
[cce lang="diff"]

if ( '' != $qv['tb'] )
$this-&gt;is_trackback = true;

if ( '' != $qv['paged'] )
$this-&gt;is_paged = true;

if ( '' != $qv['comments_popup'] )
$this-&gt;is_comments_popup = true;

// if we're previewing inside the write screen
if ('' != $qv['preview'])
$this-&gt;is_preview = true;

- if ( strpos($_SERVER['PHP_SELF'], 'wp-admin/') !== false )
+ if ( is_admin() )
$this-&gt;is_admin = true;

if ( false !== strpos($qv['feed'], 'comments-') ) {
$qv['feed'] = str_replace('comments-', '', $qv['feed']);
$qv['withcomments'] = 1;
}

$this-&gt;is_singular = $this-&gt;is_single || $this-&gt;is_page || $this-&gt;is_attachment;

if ( $this-&gt;is_feed &amp;&amp; ( !empty($qv['withcomments']) || ( empty($qv['withoutcomments']) &amp;&amp; $this-&gt;is_singular ) ) )
$this-&gt;is_comment_feed = true;

if ( !( $this-&gt;is_singular || $this-&gt;is_archive || $this-&gt;is_search || $this-&gt;is_feed || $this-&gt;is_trackback || $this-&gt;is_404 || $this-&gt;is_admin || $this-&gt;is_comments_popup ) )
$this-&gt;is_home = true;

// Correct is_* for page_on_front and page_for_posts
if ( $this-&gt;is_home &amp;&amp; ( empty($this-&gt;query) || $qv['preview'] == 'true' ) &amp;&amp; 'page' == get_option('show_on_front') &amp;&amp; get_option('page_on_front') ) {
$this-&gt;is_page = true;
$this-&gt;is_home = false;
$qv['page_id'] = get_option('page_on_front');
}
[/cce] 
