---
layout: vulnerability
tags: Code Snippet, PHP
---

# Attack the Hot Fudge Sundae

> Any reviewer who expresses rage and loathing for a novel is preposterous. He or she is like a person who has put on full armor and attacked a hot fudge sundae.

> - Kurt Vonnegut

## Vulnerable Code

```php
<?php
/**
 * Update a post with new post data.
 *
 * The date does not have to be set for drafts. You can set the date and it will
 * not be overridden.
 *
 * @since 1.0.0
 *
 * @param array|object $postarr Post data.
 * @return int 0 on failure, Post ID on success.
 */
function wp_update_post($postarr = array())
{
  if (is_object($postarr)) $postarr = get_object_vars($postarr);
  // First, get all of the original fields
  $post = wp_get_single_post($postarr['ID'], ARRAY_A);
  // Escape data pulled from DB.
  $post = add_magic_quotes($post);
  // Passed post category list overwrites existing category list if not empty.
  if (isset($postarr['post_category']) & amp; & amp;
  is_array($postarr['post_category']) & amp; & amp;
  0 != count($postarr['post_category'])) $post_cats = $postarr['post_category'];
  else $post_cats = $post['post_category'];
  // Drafts shouldn't be assigned a date unless explicitly done so by the user
  if (in_array($post['post_status'], array(
    'draft',
    'pending'
  )) & amp; & amp;
  empty($postarr['edit_date']) & amp; & amp;
  ('0000-00-00 00:00:00' == $post['post_date_gmt'])) $clear_date = true;
  else $clear_date = false;
  // Merge old and new fields with new fields overwriting old ones.
  $postarr = array_merge($post, $postarr);
  $postarr['post_category'] = $post_cats;
  if ($clear_date) {
    $postarr['post_date'] = current_time('mysql');
    $postarr['post_date_gmt'] = '';
  }
  if ($postarr['post_type'] == 'attachment') return wp_insert_attachment($postarr);
  return wp_insert_post($postarr);
}
```
