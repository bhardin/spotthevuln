---
layout: post
tags:
- Cross-Site Scripting (XSS)
- Joomla
- PHP
- Solution
---

# Solution: Keep Carving

## Details

* __Affected Software:__ Joomla
* __Fixed in Version:__  1.x
* __Issue Type:__ Cross-Site Scripting

## Description

This was an XSS bug which affected 1.x versions of Joomla. Looking at the provided code sample, we see two different functions (function display and function conclusion). Both functions take various parameters including a parameter named $searchword. Looking at the function display(), we see a hint that exposes the Cross-Site Scripting vulnerability. Function display() takes the $searchword value, urldecodes the value, then passes the urldecoded value to htmlspecialchars(). This sanitization is conducted in the following lines of code:
<blockquote>$searchword = <a href="http://www.php.net/urldecode">urldecode</a>( <span style="color: #ff0000;">$searchword</span> );
$searchword    = <a href="http://www.php.net/htmlspecialchars">htmlspecialchars</a>(<span style="color: #ff0000;">$searchword</span>, ENT_QUOTES);</blockquote>
Once the $searchword variable has been sanitized, it is ultimately used in creating a link (href) to a Google search query.

```html+php
<a href="http://www.google.com/search?q=<?php echo <span style="color: #ff0000;">$searchword</span>; ?></blockquote>
```

This gives us some indication that values passed as $searchword are likely to be untrusted and could contain bad characters.

Now, if we take a look at the conclusion() function, we see that it also takes a $searchword parameter. In this case however $searchword is NOT sanitized and is passed to what appears to be a querystring. Oddly enough, it seems the Joomla developers understood some of the dangers of passing un-sanitized values to a href/link as they were careful to sanitize the $ordering and $searchphrase variables (mosGetParam provides some sanitization against XSS, but we didn't expect you to know that). In fact, $searchphrase actually undergoes two rounds of sanitization before being used to build the querystring!  Eventually, the un-sanitized $searchword is used to build a querystring which is assigned to the $link variable. The $link variable is then used to write a link (href) on the page. If the $searchword variable had contained quotes, an attacker could easily inject arbitrary HTML attributes into the href, resulting in XSS.

Adding sanitization routines similar to those found in the display() function can help prevent this exposure.

From the code sample provided, we can't be certain whether the globals $option and $Itemid are sanitized either, it would probably be a good idea for a Joomla dev to check.

## Developer's Solution

```diff
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
+$searchword = urldecode( $searchword );
+$searchword    = htmlspecialchars($searchword, ENT_QUOTES);

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
