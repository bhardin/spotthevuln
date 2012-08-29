---
layout: post
title: Percentage
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  _edit_last: '2'
  aktt_tweeted: '1'
---
<blockquote><strong>100 per cent of us die, and the percentage cannot be increased.
C.S. Lewis</strong></blockquote>
[sourcecode language="php"]
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
				&lt;a href=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=sermon-browser/new_sermon.php&amp;mid=&lt;?php echo $sermon-&gt;id ?&gt;&quot;&gt;&lt;?php _e('Edit', $sermon_domain) ?&gt;&lt;/a&gt; | &lt;a onclick=&quot;return confirm('Are you sure?')&quot; href=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=sermon-browser/sermon.php&amp;mid=&lt;?php echo $sermon-&gt;id ?&gt;&quot;&gt;&lt;?php _e('Delete', $sermon_domain) ?&gt;&lt;/a&gt;
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
