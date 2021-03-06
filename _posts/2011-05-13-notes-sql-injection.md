---
layout: post
title: Notes - SQL Injection
tags:
- Encode
- PHP
- Solution
- SQL Injection
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

Original Code: <a href="http://spotthevuln.com/2011/05/notes/">Found Here</a>
## Details
There are a couple of different issues here, but let's focus on what the developers patched. On line 27, the developer uses the $_GET['getid3'] value to build a dynamic SQL statement. This is classic SQL injection. The patch seems straight forward, escape the $_GET['getid3'] value before using it in the SQL statement. Normally, SQL injection involves breaking out of a predefined SQL statement by closing off a quoted string and injecting your own SQL statement. Most escaping functions escape quotes and other special characters so that an attacker cannot escape out of a quoted string. There is a problem in this patch though. The tainted value is NOT enclosed within quotes, so the attacker does not need to escape out of a quoted string. The attacker is free to build a SQL injection payload as long as the payload doesn't contain any special characters. So, despite escaping the $_GET['getid3'] value, SQL injection is still possible.
<br><br>
Anyone spot the unpatched XSS?


## Developer's Solution
[sourcecode language="diff" highlight="27,28"]
&lt;?php
...snip...
		// tags
		$tags = explode(',', $_POST['tags']);
		$wpdb-&gt;query(&quot;DELETE FROM {$wpdb-&gt;prefix}sb_sermons_tags WHERE sermon_id = $id;&quot;);
		foreach ($tags as $tag) {
			$clean_tag = trim(mysql_real_escape_string($tag));
			$existing_id = $wpdb-&gt;get_var(&quot;SELECT id FROM {$wpdb-&gt;prefix}sb_tags WHERE name='$clean_tag'&quot;);
			if (is_null($existing_id)) {
				$wpdb-&gt;query(&quot;INSERT  INTO {$wpdb-&gt;prefix}sb_tags VALUES (null, '$clean_tag')&quot;);
				$existing_id = $wpdb-&gt;insert_id;
			}
			$wpdb-&gt;query(&quot;INSERT INTO {$wpdb-&gt;prefix}sb_sermons_tags VALUES (null, $id, $existing_id)&quot;);
		}
		sb_delete_unused_tags();
		// everything is fine, get out of here!
		if(!isset($error)) {
			sb_ping_gallery();
			echo &quot;&lt;script&gt;document.location = '&quot;.$_SERVER['PHP_SELF'].&quot;?page=sermon-browser/sermon.php&amp;saved=true';&lt;/script&gt;&quot;;
			die();
		}
	}

	$id3_tags = array();
	if (isset($_GET['getid3'])) {
		require_once('getid3/getid3.php');
-		$file_data = $wpdb-&gt;get_row(&quot;SELECT name, type FROM {$wpdb-&gt;prefix}sb_stuff WHERE id = &quot;.$_GET['getid3']);
+		$file_data = $wpdb-&gt;get_row(&quot;SELECT name, type FROM {$wpdb-&gt;prefix}sb_stuff WHERE id = &quot;.$wpdb-&gt;escape($_GET['getid3']));
		if ($file_data !== NULL) {
			$getID3 = new getID3;
			if ($file_data-&gt;type == 'url') {
				$filename = substr($file_data-&gt;name, strrpos ($file_data-&gt;name, '/')+1);
				$sermonUploadDir = SB_ABSPATH.sb_get_option('upload_dir');
				$tempfilename = $sermonUploadDir.preg_replace('/([ ])/e', 'chr(rand(97,122))', '		').'.mp3';
				if ($tempfile = @fopen($tempfilename, 'wb'))
					if ($remote_file = @fopen($file_data-&gt;name, 'r')) {
						$remote_contents = '';
						while (!feof($remote_file)) {
							$remote_contents .= fread($remote_file, 8192);
							if (strlen($remote_contents) &gt; 65536)
							   break;
						}
						fwrite($tempfile, $remote_contents);
						fclose($remote_file);
						fclose($tempfile);
						$id3_raw_tags = $getID3-&gt;analyze(realpath($tempfilename));
						unlink ($tempfilename);
					}
			} else {
				$filename = $file_data-&gt;name;
				$id3_raw_tags = $getID3-&gt;analyze(realpath(SB_ABSPATH.sb_get_option('upload_dir').$filename));
			}
			if (!isset($id3_raw_tags['tags'])) {
				echo '&lt;div id=&quot;message&quot; class=&quot;updated fade&quot;&gt;&lt;p&gt;&lt;b&gt;'.__('No ID3 tags found.', $sermon_domain);
				if ($file_data-&gt;type == 'url')
					 echo ' Remote files must have id3v2 tags.';
				echo '&lt;/b&gt;&lt;/div&gt;';
			}
...snip...
?&gt;
[/sourcecode]
