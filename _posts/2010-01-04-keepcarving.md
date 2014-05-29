---
layout: post
title: Keep Carving
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#code #development'
  _edit_last: '1'
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 89e74920856154c57352e58868d6e8fc
  _sexybookmarks_shortUrl: http://bit.ly/5gYGTw
  bfa_ata_body_title: Keep Carving
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Keep Carving
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
---
<blockquote>I saw the angel in the marble and carved until I set him free.

- Michelangelo</blockquote>
```php
<?php

function display( &amp;$rows, $params, $pageNav, $limitstart, $limit, $total, $totalRows, $searchword ) {
global $mosConfig_hideCreateDate;
global $mosConfig_live_site, $option, $Itemid;

$c             = count ($rows);
$image         = mosAdminMenus::ImageCheck( 'google.png', '/images/M_images/', NULL, NULL, 'Google', 'Google', 1 );
$searchword = urldecode( $searchword );
$searchword    = htmlspecialchars($searchword, ENT_QUOTES);

// number of matches found
echo '&lt;br/&gt;';
eval ('echo "'._CONCLUSION.'";');

?&gt;
&lt;a href="http://www.google.com/search?q=&lt;?php echo $searchword; ?&gt;" target="_blank"&gt;
&lt;?php echo $image; ?&gt;&lt;/a&gt;
&lt;/td&gt;
&lt;/tr&gt;
&lt;/table&gt;

...

&lt;?php
function conclusion( $searchword, $pageNav ) {
global $mosConfig_live_site, $option, $Itemid;

$ordering         = strtolower( strval( mosGetParam( $_REQUEST, 'ordering', 'newest' ) ) );
$searchphrase     = strtolower( strval( mosGetParam( $_REQUEST, 'searchphrase', 'any' ) ) );

$searchphrase    = htmlspecialchars($searchphrase);

$link             = $mosConfig_live_site ."/index.php?option=$option&amp;Itemid=$Itemid&amp;searchword=$searchword&amp;searchphrase=$searchphrase&amp;ordering=$ordering";
?&gt;
&lt;tr&gt;
&lt;td colspan="3"&gt;
&lt;div align="center"&gt;
&lt;?php echo $pageNav-&gt;writePagesLinks( $link ); ?&gt;
&lt;/div&gt;
&lt;/td&gt;
&lt;/tr&gt;
&lt;/table&gt;
&lt;?php
}

```
