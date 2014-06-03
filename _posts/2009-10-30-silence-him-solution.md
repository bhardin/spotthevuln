---
layout: post
tags:
- Cross-Site Scripting (XSS)
- Solution
- Wordpress
---

# Solution: Silence Him!

## Details

* __Affected Software:__ WordPress
* __Fixed in Version:__  2.6
* __Issue Type:__ Cross-Site Scripting

## Description

This is a classic Cross-Site Scripting (XSS) vulnerability. The `$_POST` and the `$_GET` parameters passed to the vulnerable file are printed to the WordPress markup without being sanitized, exposing the vulnerability and making exploitation of this issue extremely simple. The vulnerability was fixed by the WordPress team by simply removing the vulnerable code.

This particular issue is interesting because it contains valuable lessons for the software project manager. The vulnerable code serves no purpose to enhance WordPress functionality. In fact, the vulnerable code was actually committed as "debugging" code used by the developer to debug a separate issue. While most XSS vulnerabilities are simply a matter of implementation or coding mistakes, this particular issue (IMHO) is an indication of weaknesses in the software project management of the WordPress project.

The core of the issue stems from the fact that a developer was allowed to commit debugging code into a production build. Committing this vulnerable code was most likely accidental. Problems with code committing can be solved though the use of an informal buddy code review requirement, formal code review for commits to production, or even automation which scans the code quality for certain indicators. [FxCop](http://msdn.microsoft.com/en-us/library/bb429476.aspx) and [PreFast](http://msdn.microsoft.com/en-us/library/ms933794.aspx) are great examples of free, customizable automation that can provide a baseline indicator of code quality for commits, although neither of these can be used against a PHP code base. The WordPress team should also consider purchasing a commercial source code scanner to provide a baseline analysis of code committed into production. At minimum, when WordPress developers commits to production should include "hotspotter" checks which highlight dangerous APIs and common coding errors (e.g. echo, print APIs, `exec()`, `system()`, etc.). This small change would reduce the amount of vulnerable code that is committed to production builds.

The vulnerable code was discovered over two years ago by the WordPress team. Hopefully, the commit process has improved and all new commits undergo a baseline security check, either through automation or manual review.

## Developer's Solution

```diff
<?
function convert_all() {
  global $wpdb;

  $wpdb->query("UPDATE $wpdb->term_taxonomy SET taxonomy = 'post_tag', parent = 0 WHERE taxonomy = 'category'");
  clean_category_cache($category->term_id);
}

function init() {
- echo '<!--'; print_r($_POST); print_r($_GET); echo '-->;';

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