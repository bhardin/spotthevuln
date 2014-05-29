---
layout: post
title: Left to Chance
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Vulnerable Source Code
  _headspace_description: Can you identify the security vulnerability in this code
    snippet?
  _sexybookmarks_shortUrl: http://bit.ly/956qjL
  _sexybookmarks_permaHash: 0336bead7057506e0412ca005b33ed33
---
<blockquote>The generation of random numbers is too important to be left to chance.
- Robert R. Coveyou, Oak Ridge National Laboratory</blockquote>

## Vulnerable Code

```html+php
<?
<h2><?php _e('Importing...') ?></h2>
<?php
$cat_id = $_POST['cat_id'];
if (($cat_id == '') || ($cat_id == 0)) {
$cat_id  = 1;
}

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
