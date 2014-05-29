---
layout: post
title: Misunderstood
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 63353b8b7ec14e992e619afe6caf04a5
  _sexybookmarks_shortUrl: http://bit.ly/d3hFVV
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  _headspace_page_title: Misunderstood
  _headspace_description: Misunderstood
  aktt_tweeted: '1'
  bfa_ata_body_title: Misunderstood
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Misunderstood
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
---
<blockquote><strong>To be great is to be misunderstood.
- Ralph Waldo Emerson</strong></blockquote>
```php
<?
// show the products of a specified manufacturer
if (isset($HTTP_GET_VARS['manufacturers_id'])) {
if (isset($HTTP_GET_VARS['filter_id']) &amp;&amp; tep_not_null($HTTP_GET_VARS['filter_id'])) {
// We are asked to show only a specific category
$listing_sql = "select " . $select_column_list . " p.products_id, p.manufacturers_id, p.products_price, p.products_tax_class_id, IF(s.status, s.specials_new_products_price, NULL) as specials_new_products_price, IF(s.status, s.specials_new_products_price, p.products_price) as final_price from " . TABLE_PRODUCTS . " p left join " . TABLE_SPECIALS . " s on p.products_id = s.products_id, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_MANUFACTURERS . " m, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_status = '1' and p.manufacturers_id = m.manufacturers_id and m.manufacturers_id = '" . (int)$HTTP_GET_VARS['manufacturers_id'] . "' and p.products_id = p2c.products_id and pd.products_id = p2c.products_id and pd.language_id = '" . (int)$languages_id . "' and p2c.categories_id = '" . (int)$HTTP_GET_VARS['filter_id'] . "'";
} else {
// We show them all
$listing_sql = "select " . $select_column_list . " p.products_id, p.manufacturers_id, p.products_price, p.products_tax_class_id, IF(s.status, s.specials_new_products_price, NULL) as specials_new_products_price, IF(s.status, s.specials_new_products_price, p.products_price) as final_price from " . TABLE_PRODUCTS . " p left join " . TABLE_SPECIALS . " s on p.products_id = s.products_id, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_MANUFACTURERS . " m where p.products_status = '1' and pd.products_id = p.products_id and pd.language_id = '" . (int)$languages_id . "' and p.manufacturers_id = m.manufacturers_id and m.manufacturers_id = '" . (int)$HTTP_GET_VARS['manufacturers_id'] . "'";
}
} else {
// show the products in a given categorie
if (isset($HTTP_GET_VARS['filter_id']) &amp;&amp; tep_not_null($HTTP_GET_VARS['filter_id'])) {
// We are asked to show only specific catgeory
$listing_sql = "select " . $select_column_list . " p.products_id, p.manufacturers_id, p.products_price, p.products_tax_class_id, IF(s.status, s.specials_new_products_price, NULL) as specials_new_products_price, IF(s.status, s.specials_new_products_price, p.products_price) as final_price from " . TABLE_PRODUCTS . " p left join " . TABLE_SPECIALS . " s on p.products_id = s.products_id, " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_MANUFACTURERS . " m, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_status = '1' and p.manufacturers_id = m.manufacturers_id and m.manufacturers_id = '" . (int)$HTTP_GET_VARS['filter_id'] . "' and p.products_id = p2c.products_id and pd.products_id = p2c.products_id and pd.language_id = '" . (int)$languages_id . "' and p2c.categories_id = '" . (int)$current_category_id . "'";
} else {
// We show them all
$listing_sql = "select " . $select_column_list . " p.products_id, p.manufacturers_id, p.products_price, p.products_tax_class_id, IF(s.status, s.specials_new_products_price, NULL) as specials_new_products_price, IF(s.status, s.specials_new_products_price, p.products_price) as final_price from " . TABLE_PRODUCTS_DESCRIPTION . " pd, " . TABLE_PRODUCTS . " p left join " . TABLE_MANUFACTURERS . " m on p.manufacturers_id = m.manufacturers_id left join " . TABLE_SPECIALS . " s on p.products_id = s.products_id, " . TABLE_PRODUCTS_TO_CATEGORIES . " p2c where p.products_status = '1' and p.products_id = p2c.products_id and pd.products_id = p2c.products_id and pd.language_id = '" . (int)$languages_id . "' and p2c.categories_id = '" . (int)$current_category_id . "'";
}
}

if ( (!isset($HTTP_GET_VARS['sort'])) || (!ereg('[1-8][ad]', $HTTP_GET_VARS['sort'])) || (substr($HTTP_GET_VARS['sort'], 0, 1) &gt; sizeof($column_list)) ) {
for ($i=0, $n=sizeof($column_list); $i&lt;$n; $i++) {
if ($column_list[$i] == 'PRODUCT_LIST_NAME') {
$HTTP_GET_VARS['sort'] = $i+1 . 'a';
} else {
$sort_col = substr($HTTP_GET_VARS['sort'], 0 , 1);
$sort_order = substr($HTTP_GET_VARS['sort'], 1);
$listing_sql .= ' order by ';

switch ($column_list[$sort_col-1]) {
case 'PRODUCT_LIST_MODEL':
$listing_sql .= "p.products_model " . ($sort_order == 'd' ? 'desc' : '') . ", pd.products_name";
break;
case 'PRODUCT_LIST_NAME':
$listing_sql .= " order by pd.products_name " . ($sort_order == 'd' ? 'desc' : '');
break;
case 'PRODUCT_LIST_MANUFACTURER':
$listing_sql .= " order by m.manufacturers_name " . ($sort_order == 'd' ? 'desc' : '') . ", pd.products_name";
break;
case 'PRODUCT_LIST_QUANTITY':
$listing_sql .= " order by p.products_quantity " . ($sort_order == 'd' ? 'desc' : '') . ", pd.products_name";
break;
case 'PRODUCT_LIST_IMAGE':

```
