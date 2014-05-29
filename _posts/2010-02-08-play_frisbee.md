---
layout: post
title: Play Frisbee
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  _sexybookmarks_shortUrl: http://bit.ly/c6LC2j
  _sexybookmarks_permaHash: 96602779f850d30b7a82ed18e368d2fa
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  bfa_ata_body_title_multi: Play Frisbee
  bfa_ata_display_body_title: ''
  bfa_ata_body_title: Play Frisbee
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
---
<blockquote><strong>Hippies, hippies... they want to save the world but all they do is smoke pot and play frisbee!</strong><strong>.
- Eric Cartman
</strong></blockquote>
```php
<?
function theme_admin_block($variables) {
$block = $variables['block'];

// Don't display the block if it has no content to display.
if (empty($block['show'])) {
return '';
}

if (empty($block['content'])) {
$output = &lt;&lt;&lt; EOT
&lt;div&gt;
&lt;h3&gt;
$block[title]
&lt;/h3&gt;
&lt;div&gt;
$block[description]
&lt;/div&gt;
&lt;/div&gt;
EOT;
}
else {
$output = &lt;&lt;&lt; EOT
&lt;div&gt;
&lt;h3&gt;
$block[title]
&lt;/h3&gt;
&lt;div&gt;
$block[content]
&lt;/div&gt;
&lt;/div&gt;
EOT;
}
return $output;
}

/**
* This function formats the content of an administrative block.
*
* @param $variables
* An associative array containing:
* - content: An array containing information about the block. It should
* include a 'title', a 'description' and a formatted 'content'.
*
* @ingroup themeable
*/
function theme_admin_block_content($variables) {
$content = $variables['content'];

if (!$content) {
return '';
}

if (system_admin_compact_mode()) {
$output = '&lt;ul&gt;';
foreach ($content as $item) {
$output .= '&lt;li&gt;' . l($item['title'], $item['href'], $item['localized_options']) . '&lt;/li&gt;';
}
$output .= '&lt;/ul&gt;';
}
else {
$output = '&lt;dl&gt;';
foreach ($content as $item) {
$output .= '&lt;dt&gt;' . l($item['title'], $item['href'], $item['localized_options']) . '&lt;/dt&gt;';
$output .= '&lt;dd&gt;' . $item['description'] . '&lt;/dd&gt;';
}
$output .= '&lt;/dl&gt;';
}
return $output;
}

/**
* This function formats an administrative page for viewing.
*
* @param $variables
* An associative array containing:
* - blocks: An array of blocks to display. Each array should include a
* 'title', a 'description', a formatted 'content' and a
* 'position' which will control which container it will be
* in. This is usually 'left' or 'right'.
*
* @ingroup themeable
*/
function theme_admin_page($variables) {
$blocks = $variables['blocks'];

$stripe = 0;
$container = array();

foreach ($blocks as $block) {
if ($block_output = theme('admin_block', array('block' =&gt; $block))) {
if (empty($block['position'])) {
// perform automatic striping.
$block['position'] = ++$stripe % 2 ? 'left' : 'right';
}
if (!isset($container[$block['position']])) {
$container[$block['position']] = '';
}
$container[$block['position']] .= $block_output;
}
}

$output = '&lt;div&gt;';
$output .= theme('system_compact_link');

```
