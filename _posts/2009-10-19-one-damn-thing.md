---
layout: post
tags:
- Code Snippet
- PHP
---

# One Damn Thing

> It's not true that life is one damn thing after another; it is one damn thing over and over.
> - Edna St. Vincent Millay (1892 - 1950)

## Vulnerable Code

```php
<?

if ('' != $qv['tb']) $this->is_trackback = true;
if ('' != $qv['paged']) $this->is_paged = true;
if ('' != $qv['comments_popup']) $this->is_comments_popup = true;

// if we're previewing inside the write screen

if ('' != $qv['preview']) $this->is_preview = true;
if (strpos($_SERVER['PHP_SELF'], 'wp-admin/') !== false)
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
