---
layout: post
title: Price - Cross-Site Scripting
tags:
- All
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- html markup
- input field
- input results
- PHP
- PunBB
- root cause analysis
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/ifb5cy
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: c17b7f36e0d0d347a80e399f5415bd21
---
<h3>Details</h3>
__Affected Software:__ PunBB

__Fixed in Version:__  2.1

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Price" href="http://spotthevuln.com/2010/12/price/" target="_blank">Found    Here</a>
<h3>Description</h3>
This week's vulnerability was a XSS bug in PunBB. PunBB was taking an un-trusted value directly from the POST parameter ($_POST['prune_sticky']) and echoing the un-trusted value directly into a value attribute for a hidden form input field. You can see the XSS bug in line 98. This echoing of un-trusted input results in XSS.

The PunBB developers did something I really like here. Instead of fixing the single instance of XSS and moving on, the PunBB developers went a step further and hardened the use of $_POST['prune_sticky']. Instead of allowing users/attacker to provide arbitrary values for $_POST['prune_sticky'] they restricted the acceptable values to 1 or 0. You can see this fix in line 11. This is a perfect example of root cause analysis in action. The PunBB developers took a few minutes to understand how the application uses $_POST[' prune_sticky'] and adjusted the application behavior to protect against other attacks while being transparent to the user. This patch submitted by the PunBB developers goes a long way in protecting their customers and is a great example of being smart about security fixes.
<h3>Developers Solution</h3>
[sourcecode language="diff" highlight="11,26,27,34,35,64,65,98,99"]
&lt;?php
... &lt;snip&gt; ...

if (isset($_GET['action']) || isset($_POST['prune']) || isset($_POST['prune_comply']))
{
	if (isset($_POST['prune_comply']))
	{
		confirm_referrer('admin_prune.php');

		$prune_from = $_POST['prune_from'];
+		$prune_sticky = isset($_POST['prune_sticky']) ? '1' : '0';
		$prune_days = intval($_POST['prune_days']);
		$prune_date = ($prune_days) ? time() - ($prune_days*86400) : -1;

		@set_time_limit(0);

		if ($prune_from == 'all')
		{
			$result = $db-&gt;query('SELECT id FROM '.$db-&gt;prefix.'forums') or error('Unable to fetch forum list', __FILE__, __LINE__, $db-&gt;error());
			$num_forums = $db-&gt;num_rows($result);

			for ($i = 0; $i &lt; $num_forums; ++$i)
			{
				$fid = $db-&gt;result($result, $i);

-				prune($fid, $_POST['prune_sticky'], $prune_date);
+				prune($fid, $prune_sticky, $prune_date);
				update_forum($fid);
			}
		}
		else
		{
			$prune_from = intval($prune_from);
-			prune($prune_from, $_POST['prune_sticky'], $prune_date);
+			prune($fid, $prune_sticky, $prune_date);
			update_forum($prune_from);
		}

		// Locate any &quot;orphaned redirect topics&quot; and delete them
		$result = $db-&gt;query('SELECT t1.id FROM '.$db-&gt;prefix.'topics AS t1 LEFT JOIN '.$db-&gt;prefix.'topics AS t2 ON t1.moved_to=t2.id WHERE t2.id IS NULL AND t1.moved_to IS NOT NULL') or error('Unable to fetch redirect topics', __FILE__, __LINE__, $db-&gt;error());
		$num_orphans = $db-&gt;num_rows($result);

		if ($num_orphans)
		{
			for ($i = 0; $i &lt; $num_orphans; ++$i)
				$orphans[] = $db-&gt;result($result, $i);

			$db-&gt;query('DELETE FROM '.$db-&gt;prefix.'topics WHERE id IN('.implode(',', $orphans).')') or error('Unable to delete redirect topics', __FILE__, __LINE__, $db-&gt;error());
		}

		redirect('admin_prune.php', 'Posts pruned. Redirecting &amp;hellip;');
	}


	$prune_days = $_POST['req_prune_days'];
	if (!@preg_match('#^\d+$#', $prune_days))
		message('Days to prune must be a positive integer.');

	$prune_date = time() - ($prune_days*86400);
	$prune_from = $_POST['prune_from'];

	// Concatenate together the query for counting number or topics to prune
	$sql = 'SELECT COUNT(id) FROM '.$db-&gt;prefix.'topics WHERE last_post&lt;'.$prune_date.' AND moved_to IS NULL';

-	if ($_POST['prune_sticky'] == '0')
+	if (!$prune_sticky)
		$sql .= ' AND sticky=\'0\'';

	if ($prune_from != 'all')
	{
		$prune_from = intval($prune_from);
		$sql .= ' AND forum_id='.$prune_from;

		// Fetch the forum name (just for cosmetic reasons)
		$result = $db-&gt;query('SELECT forum_name FROM '.$db-&gt;prefix.'forums WHERE id='.$prune_from) or error('Unable to fetch forum name', __FILE__, __LINE__, $db-&gt;error());
		$forum = '&quot;'.pun_htmlspecialchars($db-&gt;result($result)).'&quot;';
	}
	else
		$forum = 'all forums';

	$result = $db-&gt;query($sql) or error('Unable to fetch topic prune count', __FILE__, __LINE__, $db-&gt;error());
	$num_topics = $db-&gt;result($result);

	if (!$num_topics)
		message('There are no topics that are '.$prune_days.' days old. Please decrease the value of &quot;Days old&quot; and try again.');


	$page_title = pun_htmlspecialchars($pun_config['o_board_title']).' / Admin / Prune';
	require PUN_ROOT.'header.php';

	generate_admin_menu('prune');

?&gt;
	&lt;div class=&quot;blockform&quot;&gt;
		&lt;h2&gt;&lt;span&gt;Prune&lt;/span&gt;&lt;/h2&gt;
		&lt;div class=&quot;box&quot;&gt;
			&lt;form method=&quot;post&quot; action=&quot;admin_prune.php?action=foo&quot;&gt;
				&lt;div class=&quot;inform&quot;&gt;
					&lt;input type=&quot;hidden&quot; name=&quot;prune_days&quot; value=&quot;&lt;?php echo $prune_days ?&gt;&quot; /&gt;
-					&lt;input type=&quot;hidden&quot; name=&quot;prune_sticky&quot; value=&quot;&lt;?php echo $_POST['prune_sticky'] ?&gt;&quot; /&gt;
+					&lt;input type=&quot;hidden&quot; name=&quot;prune_sticky&quot; value=&quot;&lt;?php echo $prune_sticky ?&gt;&quot; /&gt;
					&lt;input type=&quot;hidden&quot; name=&quot;prune_from&quot; value=&quot;&lt;?php echo $prune_from ?&gt;&quot; /&gt;
					&lt;fieldset&gt;
						&lt;legend&gt;Confirm prune posts&lt;/legend&gt;
						&lt;div class=&quot;infldset&quot;&gt;
							&lt;p&gt;Are you sure that you want to prune all topics older than &lt;?php echo $prune_days ?&gt; days from &lt;?php echo $forum ?&gt;? (&lt;?php echo $num_topics ?&gt; topics)&lt;/p&gt;
							&lt;p&gt;WARNING! Pruning posts deletes them permanently.&lt;/p&gt;
						&lt;/div&gt;
					&lt;/fieldset&gt;
				&lt;/div&gt;
				&lt;p&gt;&lt;input type=&quot;submit&quot; name=&quot;prune_comply&quot; value=&quot;Prune&quot; /&gt;&lt;a href=&quot;javascript:history.go(-1)&quot;&gt;Go back&lt;/a&gt;&lt;/p&gt;
			&lt;/form&gt;
		&lt;/div&gt;
	&lt;/div&gt;
	&lt;div class=&quot;clearer&quot;&gt;&lt;/div&gt;
&lt;/div&gt;
... &lt;snip&gt; ...
[/sourcecode]
