---
layout: post
title: Silence Him! - XSS
tags:
- Cross-Site Scripting (XSS)
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#xss #vulnerability #owasp'
  _edit_last: '1'
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  _headspace_description: Cross-Site Scripting (XSS) Vulnerability Code Example
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/c3AB9
  _sexybookmarks_permaHash: 484b6e54a9c37307373ac7b8d559c2ac
---
## Details
__Affected Software:__ <strong><a title="WordPress bugs" href="http://spotthevuln.com/category/software/wordpress/" target="_blank">WordPress</a></strong>

<strong>__Fixed in Version:__  2.6</strong>

__Issue Type:__ <strong><a title="Cross Site Scripting" href="http://spotthevuln.com/category/vulnerability/xss/" target="_blank">Cross Site Scripting</a></strong>

<strong>Original Code: <a href="http://spotthevuln.com/2009/10/vulnerable-code-silence-him/">Found Here</a></strong>
## Description
This is a classic Cross-Site Scripting (XSS) vulnerability.  The $_POST and the $_GET parameters passed to the vulnerable file are printed to the WordPress markup without being sanitized, exposing the vulnerability and making exploitation of this issue extremely simple.  The XSS Vulnerability was fixed by the WordPress team by simply removing the vulnerable code.

This particular issue is interesting because it contains valuable lessons for the software project manager.  The vulnerable code serves no purpose to enhance WordPress functionality.  In fact, the vulnerable code was actually checked in as "debugging" code used by the developer to debug a separate issue.  While most XSS vulnerabilities are simply a matter of implementation or coding mistakes, this particular issue (IMHO) is an indication of weaknesses in the software project management of the WordPress project.

The core of the issue stems from the fact that a developer was allowed to check-in debugging code into a production build.  Checking in this vulnerable code was most likely accidental. Problems with code check-in can be solved though the use of an informal buddy code review requirement, formal code review for check-ins to production, or even automation which scans the code quality for certain indicators.  <a href="http://msdn.microsoft.com/en-us/library/bb429476(VS.80).aspx" target="_blank">FxCop</a> and <a href="http://msdn.microsoft.com/en-us/library/ms933794.aspx" target="_blank">PreFast</a> are great examples of free, customizable automation that can provide a baseline indicator of code quality for check-ins, although neither of these can be used against a PHP code base.  The WordPress team should also consider purchasing a commercial source code scanner to provide a baseline analysis of code checked into production.  At minimum, when WordPress developers check-in to production should include "hotspotter" checks which highlight dangerous APIs/common coding errors (e.g. echo, print APIs, exec(), system() ...etc.). This would greatly reduce the amount of vulnerable code that is checked-in to the production builds.

The vulnerable code was discovered over two years ago by the WordPress team.  Hopefully, the check-in process has improved and all new check-in's undergo a baseline security check, either through automation or manual review.
## Developers Solution
[cce lang="diff"]

function convert_all() {
global $wpdb;

$wpdb-&gt;query("UPDATE $wpdb-&gt;term_taxonomy SET taxonomy = 'post_tag', parent = 0 WHERE taxonomy = 'category'");
clean_category_cache($category-&gt;term_id);
}

function init() {
-               echo '&lt;!--'; print_r($_POST); print_r($_GET); echo '--&gt;';

if (isset($_POST['maybe_convert_all_cats'])) {
$step = 3;
} elseif (isset($_POST['yes_convert_all_cats'])) {
$step = 4;
} elseif (isset($_POST['no_dont_do_it'])) {
die('no_dont_do_it');
} else {
$step = (isset($_GET['step'])) ? (int) $_GET['step'] : 1;
}

$this-&gt;header();

if (!current_user_can('manage_categories')) {
print '&lt;div&gt;';
print '&lt;p&gt;' . __('Cheatin' uh?') . '&lt;/p&gt;';
print '&lt;/div&gt;';
} else {
switch ($step) {
case 1 :
$this-&gt;welcome();
break;

case 2 :
$this-&gt;convert_them();
break;

case 3 :
$this-&gt;convert_all_confirm();

[/cce] 
