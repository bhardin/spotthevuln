---
layout: post
tags:
- Code Snippet
- PHP
---

# Silence Him!

> You have not converted a man because you have silenced him.
> - John Viscount Morley

## Vulnerable Code

```php
<?
function convert_all() {
  global $wpdb;

  $wpdb->query("UPDATE $wpdb->term_taxonomy SET taxonomy = 'post_tag', parent = 0 WHERE taxonomy = 'category'");
  clean_category_cache($category->term_id);
}

function init() {
  echo '<!--'; print_r($_POST); print_r($_GET); echo '-->;';

  if (isset($_POST['maybe_convert_all_cats'])) {
    $step = 3;
  } elseif (isset($_POST['yes_convert_all_cats'])) {
    $step = 4;
  } elseif (isset($_POST['no_dont_do_it'])) {
    die('no_dont_do_it');
  } else {
    $step = (isset($_GET['step'])) ? (int) $_GET['step'] : 1;
  }

  $this->header();

  if (!current_user_can('manage_categories')) {
    print '<div>';
    print '<p>' . __("Cheatin uh?") . '</p>';
    print '</div>';
  } else {
    switch ($step) {
    case 1 :
      $this->welcome();
      break;

    case 2 :
      $this->convert_them();
      break;

    case 3 :
      $this->convert_all_confirm();
```
