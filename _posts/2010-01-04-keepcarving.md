---
layout: post
tags:
- Code Snippet
- PHP
---

# Keep Carving

> I saw the angel in the marble and carved until I set him free.

> - Michelangelo

```html+php
<?php

function display( &amp;$rows, $params, $pageNav, $limitstart, $limit, $total, $totalRows, $searchword ) {
global $mosConfig_hideCreateDate;
global $mosConfig_live_site, $option, $Itemid;

$c             = count ($rows);
$image         = mosAdminMenus::ImageCheck( 'google.png', '/images/M_images/', NULL, NULL, 'Google', 'Google', 1 );
$searchword = urldecode( $searchword );
$searchword    = htmlspecialchars($searchword, ENT_QUOTES);

// number of matches found
echo '<br/>';
eval ('echo "'._CONCLUSION.'";');

?>
<a href="http://www.google.com/search?q=<?php echo $searchword; ?>" target="_blank">
<?php echo $image; ?></a>
</td>
</tr>
</table>

...

<?php
function conclusion( $searchword, $pageNav ) {
global $mosConfig_live_site, $option, $Itemid;

$ordering         = strtolower( strval( mosGetParam( $_REQUEST, 'ordering', 'newest' ) ) );
$searchphrase     = strtolower( strval( mosGetParam( $_REQUEST, 'searchphrase', 'any' ) ) );

$searchphrase    = htmlspecialchars($searchphrase);

$link             = $mosConfig_live_site ."/index.php?option=$option&amp;Itemid=$Itemid&amp;searchword=$searchword&amp;searchphrase=$searchphrase&amp;ordering=$ordering";
?>
<tr>
<td colspan="3">
<div align="center">
<?php echo $pageNav->writePagesLinks( $link ); ?>
</div>
</td>
</tr>
</table>
<?php
}

```
