---
layout: post
title: One Damn Thing
tags:
- Code Snippet
- PHP
---
<blockquote>It's not true that life is one damn thing after another; it is one damn thing over and over.
- Edna St. Vincent Millay (1892 - 1950)</blockquote>

## Vulnerable Code

```php
<?
if ( '' != $qv['tb'] )
$this-&gt;is_trackback = true;

if ( '' != $qv['paged'] )
$this-&gt;is_paged = true;

if ( '' != $qv['comments_popup'] )
$this-&gt;is_comments_popup = true;

// if we're previewing inside the write screen
if ('' != $qv['preview'])
$this-&gt;is_preview = true;

if ( strpos($_SERVER['PHP_SELF'], 'wp-admin/') !== false )
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
```
