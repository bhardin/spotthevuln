---
layout: post
tags:
- Cross-Site Scripting (XSS)
- Wordpress
---

# Solution: Left to Chance

## Details

* __Affected Software:__ WordPress
* __Fixed in Version:__ 2.0.11
* __Issue Type:__ Cross-Site Scripting

## Description

Yet another Cross-Site Scripting vulnerability affecting WordPress. In this case, the `$cat_id` variable is assigned a user controlled value and is used without sanitization. It's a little difficult to determine using the code sample below, but the `$cat_id` is displayed in WordPress markup in multiple locations without being sanitized. More importantly, examining the WordPress source we see that almost all of the variables are not thoroughly sanitized before being assigned to a variable or being used by the application. The WordPress team should have a solid understanding of the data definitions for each of the variables it uses and should make an effort to ensure user controlled data matches that data definition. In the case of `$cat_id`, the WordPress developers understood that `$cat_id` should only have a numerical value and made use of the `abs()` function to help validate `$_POST['cat_id']` before assigning the user controlled value to `$cat_id`.

## Developer's Solution

```diff
<h2><?php _e('Importing...') ?></h2>
<?php

- $cat_id = $_POST['cat_id'];
- if (($cat_id == '') || ($cat_id == 0)) {
+ $cat_id = abs( (int) $_POST['cat_id'] );
+ if ( $cat_id < 1 )
    $cat_id  = 1;
- }
$opml_url = $_POST['opml_url'];
if (isset($opml_url) &amp;&amp; $opml_url != '' &amp;&amp; $opml_url != 'http://') {
$blogrolling = true;
}
else // try to get the upload file.
{
$overrides = array('test_form' => false, 'test_type' => false);
$file = wp_handle_upload($_FILES['userfile'], $overrides);

if ( isset($file['error']) )
die($file['error']);

$url = $file['url'];
$opml_url = $file['file'];
$blogrolling = false;
}

if (isset($opml_url) &amp;&amp; $opml_url != '') {
$opml = wp_remote_fopen($opml_url);
include_once('link-parse-opml.php');

$link_count = count($names);
for ($i = 0; $i < $link_count; $i++) {
if ('Last' == substr($titles[$i], 0, 4))
$titles[$i] = '';
if ('http' == substr($titles[$i], 0, 4))
$titles[$i] = '';
```
