---
layout: post
title: Percentage - Cross-Site Scripting
tags:
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- parameterized queries
- PHP
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Sermon Browser Wordpress Plugin

__Fixed in Version:__  .44

__Issue Type:__ Cross-Site Scripting

Original Code: <a href="http://spotthevuln.com/2011/05/percentage/">Found Here</a>
## Details
There is a lot going on here in this code snippet. First, let's talk about the patch. The patch adds a check to ensure the user requesting has the rights to edit a post. The added functionality only displays a link (A HREF) if the user has the correct permissions. Let's hope there are additional checks in place to prevent the execution of this functionality, as opposed to just trying to obscure the link.
<br><br>
Second, there are a few SQL queries. The SQL queries actually seem to be well handled; most values are cast to int, which should work. Of course, Neal Poole and Jacob astutely point out that casts to int cannot always be trusted (<a href="http://spotthevuln.com/2011/03/invincible-cross-site-scripting/">http://spotthevuln.com/2011/03/invincible-cross-site-scripting/</a> &lt;-- see comments section). Abraham Kang also points out that some escaping functions can be defeated by using alternate charsets. The better solution is to use parameterized queries.
<br><br>
What about XSS?  There is XSS everywhere on this page. In fact, there is so much XSS I'm not even going to try to list all of the exposures. Instead, I'll focus on two different patterns. Let's start with a classic XSS bug seen in many different PHP products. On line XX, the developer echoes back $_SERVER['PHP_SELF']. This results in XSS. Many developers think that PHP_SELF will only echo back the current URL path with no user controlled input, but sadly this is not the case. PHP_SELF can almost always be tainted by a user/attacker. This comment on the reserved variables page for PHP sums it up nicely: <a href="http://www.php.net/manual/en/reserved.variables.php#55068">http://www.php.net/manual/en/reserved.variables.php#55068</a>.
<br><br>
Now, looking at lines 30-35, we see the developer uses stripslashes before echoing back a user controlled value. I'm guessing this is because the developer was worried about echoing back data that was stored with a database escaping function. Unfortunately, stripslashes does not prevent XSS. Pretty much all of the echo statements on this page are vulnerable to XSS (unless the data is being stored  in an HTML encoded state). The echo statements on line 68 and line 70 are 100% XSS.
<br><br>
From the looks of this code, it's likely the developer doesn't know what XSS is. Any security training should probably first focus on this deficiency. Other code written by this developer should also be audited for XSS.


## Developer's Solution
[sourcecode language="diff" highlight="37-42,68,70"]
&lt;?php
...snip...
} elseif (isset($_POST['fetch'])) { // ajax pagination
	if (function_exists('wp_timezone_override_offset'))
		wp_timezone_override_offset();
	$st = (int) $_POST['fetch'] - 1;
	if (!empty($_POST['title'])) {
		$cond = &quot;and m.title LIKE '%&quot; . mysql_real_escape_string($_POST['title']) . &quot;%' &quot;;
	} else
		$cond = '';
	if ($_POST['preacher'] != 0) {
		$cond .= 'and m.preacher_id = ' . (int) $_POST['preacher'] . ' ';
	}
	if ($_POST['series'] != 0) {
		$cond .= 'and m.series_id = ' . (int) $_POST['series'] . ' ';
	}
	$m = $wpdb-&gt;get_results(&quot;SELECT SQL_CALC_FOUND_ROWS m.id, m.title, m.datetime, p.name as pname, s.name as sname, ss.name as ssname
	FROM {$wpdb-&gt;prefix}sb_sermons as m
	LEFT JOIN {$wpdb-&gt;prefix}sb_preachers as p ON m.preacher_id = p.id
	LEFT JOIN {$wpdb-&gt;prefix}sb_services as s ON m.service_id = s.id
	LEFT JOIN {$wpdb-&gt;prefix}sb_series as ss ON m.series_id = ss.id
	WHERE 1=1 {$cond}
	ORDER BY m.datetime desc, s.time desc LIMIT {$st}, &quot;.sb_get_option('sermons_per_page'));

	$cnt = $wpdb-&gt;get_var(&quot;SELECT FOUND_ROWS()&quot;);
	?&gt;
	&lt;?php foreach ($m as $sermon): ?&gt;
		&lt;tr class=&quot;&lt;?php echo ++$i % 2 == 0 ? 'alternate' : '' ?&gt;&quot;&gt;
			&lt;th style=&quot;text-align:center&quot; scope=&quot;row&quot;&gt;&lt;?php echo $sermon-&gt;id ?&gt;&lt;/th&gt;
			&lt;td&gt;&lt;?php echo stripslashes($sermon-&gt;title) ?&gt;&lt;/td&gt;
			&lt;td&gt;&lt;?php echo stripslashes($sermon-&gt;pname) ?&gt;&lt;/td&gt;
			&lt;td&gt;&lt;?php echo ($sermon-&gt;datetime == '1970-01-01 00:00:00') ? __('Unknown', $sermon_domain) : strftime('%d %b %y', strtotime($sermon-&gt;datetime)); ?&gt;&lt;/td&gt;
			&lt;td&gt;&lt;?php echo stripslashes($sermon-&gt;sname) ?&gt;&lt;/td&gt;
			&lt;td&gt;&lt;?php echo stripslashes($sermon-&gt;ssname) ?&gt;&lt;/td&gt;
			&lt;td&gt;&lt;?php echo sb_sermon_stats($sermon-&gt;id) ?&gt;&lt;/td&gt;
			&lt;td style=&quot;text-align:center&quot;&gt;
-				&lt;a href=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=sermon-browser/new_sermon.php&amp;mid=&lt;?php echo $sermon-&gt;id ?&gt;&quot;&gt;&lt;?php _e('Edit', $sermon_domain) ?&gt;&lt;/a&gt; | &lt;a onclick=&quot;return confirm('Are you sure?')&quot; href=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=sermon-browser/sermon.php&amp;mid=&lt;?php echo $sermon-&gt;id ?&gt;&quot;&gt;&lt;?php _e('Delete', $sermon_domain) ?&gt;&lt;/a&gt;
+				&lt;?php //Security check
+							if (current_user_can('edit_posts')) { ?&gt;
+							&lt;a href=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=sermon-browser/new_sermon.php&amp;mid=&lt;?php echo $sermon-&gt;id ?&gt;&quot;&gt;&lt;?php _e('Edit', $sermon_domain) ?&gt;&lt;/a&gt; | &lt;a onclick=&quot;return confirm('Are you sure?')&quot; href=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=sermon-browser/sermon.php&amp;mid=&lt;?php echo $sermon-&gt;id ?&gt;&quot;&gt;&lt;?php _e('Delete', $sermon_domain); ?&gt;&lt;/a&gt; |
+				&lt;?php } ?&gt;
+				&lt;a href=&quot;&lt;?php echo sb_display_url().sb_query_char(true).'sermon_id='.$sermon-&gt;id;?&gt;&quot;&gt;View&lt;/a&gt;
			&lt;/td&gt;
		&lt;/tr&gt;
	&lt;?php endforeach ?&gt;
	&lt;script type=&quot;text/javascript&quot;&gt;
	&lt;?php if($cnt&lt;sb_get_option('sermons_per_page') || $cnt &lt;= $st+sb_get_option('sermons_per_page')): ?&gt;
		jQuery('#right').css('display','none');
	&lt;?php elseif($cnt &gt; $st+sb_get_option('sermons_per_page')): ?&gt;
		jQuery('#right').css('display','');
	&lt;?php endif ?&gt;
	&lt;/script&gt;
	&lt;?php
} elseif (isset($_POST['fetchU']) || isset($_POST['fetchL']) || isset($_POST['search'])) { // ajax pagination (uploads)
	if (isset($_POST['fetchU'])) {
		$st = (int) $_POST['fetchU'] - 1;
		$abc = $wpdb-&gt;get_results(&quot;SELECT f.*, s.title FROM {$wpdb-&gt;prefix}sb_stuff AS f LEFT JOIN {$wpdb-&gt;prefix}sb_sermons AS s ON f.sermon_id = s.id WHERE f.sermon_id = 0 AND f.type = 'file' ORDER BY f.name LIMIT {$st}, &quot;.sb_get_option('sermons_per_page'));
	} elseif (isset($_POST['fetchL'])) {
		$st = (int) $_POST['fetchL'] - 1;
		$abc = $wpdb-&gt;get_results(&quot;SELECT f.*, s.title FROM {$wpdb-&gt;prefix}sb_stuff AS f LEFT JOIN {$wpdb-&gt;prefix}sb_sermons AS s ON f.sermon_id = s.id WHERE f.sermon_id &lt;&gt; 0 AND f.type = 'file' ORDER BY f.name LIMIT {$st}, &quot;.sb_get_option('sermons_per_page'));
	} else {
		$s = mysql_real_escape_string($_POST['search']);
		$abc = $wpdb-&gt;get_results(&quot;SELECT f.*, s.title FROM {$wpdb-&gt;prefix}sb_stuff AS f LEFT JOIN {$wpdb-&gt;prefix}sb_sermons AS s ON f.sermon_id = s.id WHERE f.name LIKE '%{$s}%' AND f.type = 'file' ORDER BY f.name;&quot;);
	}
?&gt;
&lt;?php if (count($abc) &gt;= 1): ?&gt;
	&lt;?php foreach ($abc as $file): ?&gt;
		&lt;tr class=&quot;file &lt;?php echo (++$i % 2 == 0) ? 'alternate' : '' ?&gt;&quot; id=&quot;&lt;?php echo $_POST['fetchU'] ? '' : 's' ?&gt;file&lt;?php echo $file-&gt;id ?&gt;&quot;&gt;
			&lt;th style=&quot;text-align:center&quot; scope=&quot;row&quot;&gt;&lt;?php echo $file-&gt;id ?&gt;&lt;/th&gt;
			&lt;td id=&quot;&lt;?php echo $_POST['fetchU'] ? '' : 's' ?&gt;&lt;?php echo $file-&gt;id ?&gt;&quot;&gt;&lt;?php echo substr($file-&gt;name, 0, strrpos($file-&gt;name, '.')) ?&gt;&lt;/td&gt;
			&lt;td style=&quot;text-align:center&quot;&gt;&lt;?php echo isset($filetypes[substr($file-&gt;name, strrpos($file-&gt;name, '.') + 1)]['name']) ? $filetypes[substr($file-&gt;name, strrpos($file-&gt;name, '.') + 1)]['name'] : strtoupper(substr($file-&gt;name, strrpos($file-&gt;name, '.') + 1)) ?&gt;&lt;/td&gt;
			&lt;?php if (!isset($_POST['fetchU'])) { ?&gt;&lt;td&gt;&lt;?php echo stripslashes($file-&gt;title) ?&gt;&lt;/td&gt;&lt;?php } ?&gt;
			&lt;td style=&quot;text-align:center&quot;&gt;
				&lt;script type=&quot;text/javascript&quot; language=&quot;javascript&quot;&gt;
				function deletelinked_&lt;?php echo $file-&gt;id;?&gt;(filename, filesermon) {
					if (confirm('Do you really want to delete '+filename+'?')) {
						if (filesermon != '') {
							return confirm('This file is linked to the sermon called ['+filesermon+']. Are you sure you want to delete it?');
						}
						return true;
					}
					return false;
				}
				&lt;/script&gt;
				&lt;?php if (isset($_POST['fetchU'])) { ?&gt;&lt;a id=&quot;&quot; href=&quot;&lt;?php echo $_SERVER['PHP_SELF'].&quot;?page=sermon-browser/new_sermon.php&amp;amp;getid3={$file-&gt;id}&quot;; ?&gt;&quot;&gt;&lt;?php _e('Create sermon', $sermon_domain) ?&gt;&lt;/a&gt; | &lt;?php } ?&gt;
				&lt;a id=&quot;link&lt;?php echo $file-&gt;id ?&gt;&quot; href=&quot;javascript:rename(&lt;?php echo $file-&gt;id ?&gt;, '&lt;?php echo $file-&gt;name ?&gt;')&quot;&gt;&lt;?php _e('Rename', $sermon_domain) ?&gt;&lt;/a&gt; | &lt;a onclick=&quot;return deletelinked_&lt;?php echo $file-&gt;id;?&gt;('&lt;?php echo str_replace(&quot;'&quot;, '', $file-&gt;name) ?&gt;', '&lt;?php echo str_replace(&quot;'&quot;, '', $file-&gt;title) ?&gt;');&quot; href=&quot;javascript:kill(&lt;?php echo $file-&gt;id ?&gt;, '&lt;?php echo $file-&gt;name ?&gt;');&quot;&gt;&lt;?php _e('Delete', $sermon_domain) ?&gt;&lt;/a&gt;
			&lt;/td&gt;
		&lt;/tr&gt;
	&lt;?php endforeach ?&gt;
&lt;?php else: ?&gt;
	&lt;tr&gt;
		&lt;td&gt;&lt;?php _e('No results', $sermon_domain) ?&gt;&lt;/td&gt;
	&lt;/tr&gt;
&lt;?php endif ?&gt;
&lt;?php
}
die();

?&gt;
[/sourcecode]
