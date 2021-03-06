---
layout: post
title: Play Frisbee - XSS
tags:
- Cross-Site Scripting (XSS)
- Drupal
- PHP
- Solution
status: publish
type: post
published: true
meta:
  _sexybookmarks_shortUrl: http://bit.ly/byZcoX
  _sexybookmarks_permaHash: f4b627f2eb12b1fef5cc962d237acdfe
  _aktt_hash_meta: ''
  aktt_notify_twitter: 'yes'
  _edit_last: '1'
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Drupal

__Fixed in Version:__  5.21 and 6.15

__Issue Type:__ XSS

Original Code: <a title="Hippies" href="http://spotthevuln.com/2010/02/play_frisbee/" target="_blank">Found Here</a>
## Description
Simple XSS on Durpal 5.x and 6.x platforms. What's interesting about this particular vulnerability is it shows the advantages of implementing HTML output encoding near the point of consumption and the value of having descriptive function names. In this sample, it's difficult to determine whether the values contained within the $item[] array are already sanitized. In order to determine whether $item[] is tainted, the code reviewer would have to find where $item[] values are set and trace them through the entire code base until it is presented to this particular usage. Additionally, it seems the various values within the $item array: $item['title'], $item['href'], and $item['localized_options'] (all of which appear to be user/attacker controlled) are passed to a function called "l()". "l()" isn't a very descriptive function name :).

The specific issue fixed here was the use of $item['description'] while building HTML markup. The Durpal fixed this issue by passing $item['description'] to the filter_xss_admin(), near the point of consumption. Filter_xss_admin() is a fairly descriptive function name and gives the reviewer a better understanding of what the code is doing. Also, because filter_xss_admin() is used at the point of consumption, the code reviewer can rest assured that the value is encoded before being used, regardless of the previous input sanitization that may/may not have taken place.
## Developer's Solution
```diff
<div id="_mcePaste">

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

-$output .= '&lt;dd&gt;' . $item['description'] . '&lt;/dd&gt;';

+$output .= '&lt;dd&gt;' . filter_xss_admin($item['description']) . '&lt;/dd&gt;'

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

</div>
```
