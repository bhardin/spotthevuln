---
layout: post
title: Dont Know - SQL Injection
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _sexybookmarks_shortUrl: http://bit.ly/cPf6Pd
  _sexybookmarks_permaHash: 9238e6c89d4931f3bcf38530137149dc
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '2'
  bfa_ata_body_title: Dont Know - SQL Injection
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Dont Know - SQL Injection
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Yak for Wordpress

__Fixed in Version:__  2.1.0

__Issue Type:__ SQL Injection

Original Code: <a title="Dont Know" href="http://spotthevuln.com/2010/05/dont-know/" target="_blank">Found  Here</a>
## Description
This example covers a SQL injection in the YAK for Wordpress plugin. YAK is a shopping cart plugin for WordPress which associates products with weblog entries. Shopping carts are the favorite of many web hackers as they typically contain security flaws due to the nature and exposed functionality of shopping carts. The <a title="Web Application Hackers Handbook" href="http://www.amazon.com/dp/0470170778" target="_blank">Web Application Hackers Handbook</a> (by Dafydd Stuttard aka <a title="Portswigger" href="http://portswigger.net/" target="_blank">Portswigger</a>) has some excellent examples of typically flaws that can be associated with online shopping carts as well as some techniques to exploit those flaws... highly recommended if you don't have it already!

Looking at the code in this example, we see a classic SQL Injection through SQL statement string building. There are several places where the YAK plugin developer takes user controlled values and uses those values as part of a dynamic SQL query. The bright spot in this code fix is once the vulnerabilities were discovered, the developer made use of prepared statements (they utilized the wordpress prepared statement functions) to address the vulnerabilities. An advantage to using prepared statements (as opposed to input validation before using a value in a dynamic SQL statement) is the developer doesn't have to worry about missing some malicious character sequence or being too restrictive with legitimate data.
## Developer's Solution
```diff

&lt;?php
if (!function_exists('yak_calc_discount_price')) {
/**
* Calculate a price discount, or return 0 if not a pricing promotion
*/
function yak_calc_price_discount($item_quantity, $price, $total_items, $total_price, $promo=null) {
if ($promo != null) {
if ($promo-&gt;promo_type == 'pricing_perc' || $promo-&gt;promo_type == 'pricing_perc_threshold') {
return ($promo-&gt;value / 100.0) * $price;
}
else if ($promo-&gt;promo_type == 'pricing_val' || $promo-&gt;promo_type == 'pricing_val_threshold') {
if ($total_price &gt; 0) {
$item_perc_of_total = $price / $total_price;
return $promo-&gt;value * $item_perc_of_total;
}
}
}

return 0;
}
}

if (!function_exists('yak_calc_discount_shipping')) {
/**
* Calculate a shipping discount, or return 0 if not a shipping discount
*/
function yak_calc_shipping_discount($shipping_cost, $promo=null) {
if ($promo != null) {
if ($promo-&gt;promo_type == 'shipping_perc'  || $promo-&gt;promo_type == 'shipping_perc_threshold') {
return ($promo-&gt;value / 100.0) * $shipping_cost;
}
else if ($promo-&gt;promo_type == 'shipping_val' || $promo-&gt;promo_type == 'shipping_val_threshold') {
return $promo-&gt;value;
}
}

return 0;
}
}


if (!function_exists('yak_get_promotion')) {
function yak_get_promotion($code) {
$promos = yak_get_promotions($code, true);
if (sizeof($promos) &gt;= 1) {
return $promos[0];
}
else {
return new YakPromotion('', null, null, null, 0, '', '', null);
}
}
}


if (!function_exists('yak_get_promotion_by_threshold')) {
function yak_get_promotion_by_threshold($threshold) {
$promos = yak_get_promotions(null, true, $threshold);
if (sizeof($promos) &gt;= 1) {
return $promos[0];
}
else {
return null;
}
}
}


if (!function_exists('yak_get_promotions')) {
function yak_get_promotions($code = null, $valid = false, $threshold = null) {
global $wpdb;

$registry =&amp; Registry::getInstance();
$promo_table =&amp; $registry-&gt;get('promo_table');
$promo_users_table =&amp; $registry-&gt;get('promo_users_table');

+    $args = array();
$sql = "select promo_id, code, promo_type, description, threshold, value, expiry_date
from $promo_table
where 1 = 1 ";

// search by code
if (!empty($code)) {
-           $sql .= "and code = '$code' ";
+        $sql .= "and code = %s ";
$sql .= "and promo_type in ('shipping_perc', 'shipping_val', 'pricing_perc', 'pricing_val') ";
+        $args[] = $code;
}
// or search by threshold
else if (!empty($threshold)) {
-           $sql .= "and threshold is not null and $threshold &gt; threshold ";
+        $sql .= "and threshold is not null and %f &gt; threshold ";
$sql .= "and promo_type in ('shipping_perc_threshold', 'shipping_val_threshold', 'pricing_perc_threshold',

'pricing_val_threshold') ";
+        $args[] = $threshold;
}

// check for validity
if ($valid) {
$sql .= "and (expiry_date is null or expiry_date &gt;= current_date) ";
}

// final sql
if (!empty($threshold)) {
$sql .= "order by threshold desc limit 1";
}
else {
$sql .= "order by promo_id asc";
}

+    $sql = $wpdb-&gt;prepare($sql, $args);
$results = $wpdb-&gt;get_results($sql);

$promos = array();
foreach ($results as $result) {
-           $rows = $wpdb-&gt;get_results("select user_nicename
-                                       from $wpdb-&gt;users u
-                                       where exists (select 1
-                                                      from $promo_users_table pu
-                                                      where pu.promo_id = $result-&gt;promo_id
-                                                      and pu.user_id = u.ID)");
+           $sql = $wpdb-&gt;prepare("select user_nicename
+                                       from $wpdb-&gt;users u
+                                       where exists (select 1
+                                                      from $promo_users_table pu
+                                                      where pu.promo_id = %d
+                                                     and pu.user_id = u.ID)", $result-&gt;promo_id);
+           $rows = $wpdb-&gt;get_results($sql);
$users = array();
foreach ($rows as $row) {
$users[] = $row-&gt;user_nicename;
}
$promo = new YakPromotion($result-&gt;promo_id, $result-&gt;code, $result-&gt;promo_type, $result-&gt;threshold, $result-

&gt;value,
$result-&gt;description, $result-&gt;expiry_date, $users);
$promos[] = $promo;
}

return $promos;
}
}
?&gt;

```
