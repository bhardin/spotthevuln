---
layout: post
title: Tougher - SQL Injection
tags:
- All
- array
- dynamic SQL
- injection bug
- PHP
- PunBB
- punbb
- Solution
- SQL Injection
- sql statement
- SQLi
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/hUDV7Q
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: 3e788b730e85a0057b3c91b7116897bb
---
<h3>Details</h3>
__Affected Software:__ PunBB

__Fixed in Version:__  1.3

__Issue Type:__ SQL Injection (SQLi)

Original Code: <a title="Tougher" href="http://spotthevuln.com/2010/12/tougher/" target="_blank">Found    Here</a>
<h3>Description</h3>
This week's bug was an old SQL injection bug that affected PunBB versions &lt; 1.3. In short, a value is taken from an attacker/user controlled POST request and is used to build a SQL statement. This bug actually requires a small amount of tracing, so here we go!  First, we see that PunBB takes the attacker/user supplied content here (line 11)
<br><br>
[sourcecode language="diff" highlight="11,19,26,27" firstline="11"]$form = array_map('trim', $_POST['form']);[/sourcecode]
<br><br>
The line above uses the value passed via $_POST['form'] to populate the $form variable. The value goes through a trim() function, but is (for the most part) un-sanitized. It's interesting that PHP allows for the submission of arrays through POST parameters. This behavior is mentioned in the comments on this <a title="Arrays" href="http://php.net/manual/en/reserved.variables.post.php" target="_blank">page</a>

Next, the $form variable (which contains our attacker supplied values from $_POST['form']) is used in a foreach statement and each index of the $form variable value is used in some application logic. You can see this in the following line (line 19)
<br><br>
[sourcecode language="diff" highlight="11,19,26,27" firstline="19"]foreach ($form as $key =&gt; $input)[/sourcecode]
<br><br>
The foreach extracts the various values from the $form variable, does a quick comparison to a configuration value of some sort. If the comparison returns the correct value, the application uses the tainted value to populate a $query array variable. The tainted value is used here (line 26)
<br><br>
[sourcecode language="diff" highlight="11,19,26,27" firstline="26"]'SET'		=&gt; 'conf_value='.$input,[/sourcecode]
<br><br>
The name of the variable ($query), along with the names of the indexes in the array (UPDATE, SET, WHERE), and finally the names of variables/functions close-by ($forum_db, query_build) are dead giveaways that the untainted value will eventually be used in a SQL query. Use of a tainted value in this manner leads to SQL injection.
The developers addressed this issue by casting the tainted $input value to an int before using it to build a SQL statement.

<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="11,19,26,27"]
...&lt;snip&gt;...

// Load the admin.php language file
require FORUM_ROOT.'lang/'.$forum_user['language'].'/admin_common.php';
require FORUM_ROOT.'lang/'.$forum_user['language'].'/admin_settings.php';

$section = isset($_GET['section']) ? $_GET['section'] : null;

if (isset($_POST['form_sent']))
{
	$form = array_map('trim', $_POST['form']);

	($hook = get_hook('aop_form_submitted')) ? eval($hook) : null;

...&lt;snip&gt;...

	($hook = get_hook('aop_pre_update_configuration')) ? eval($hook) : null;

	foreach ($form as $key =&gt; $input)
	{
		// Only update permission values that have changed
		if (array_key_exists('p_'.$key, $forum_config) &amp;&amp; $forum_config['p_'.$key] != $input)
		{
			$query = array(
				'UPDATE'	=&gt; 'config',
-				'SET'		=&gt; 'conf_value='.$input,
+				'SET'       =&gt; 'conf_value='.intval($input),
				'WHERE'		=&gt; 'conf_name=\'p_'.$forum_db-&gt;escape($key).'\''
			);

			($hook = get_hook('aop_qr_update_permission_conf')) ? eval($hook) : null;
			$forum_db-&gt;query_build($query) or error(__FILE__, __LINE__);
		}

		// Only update option values that have changed
		if (array_key_exists('o_'.$key, $forum_config) &amp;&amp; $forum_config['o_'.$key] != $input)
		{
			if ($input != '' || is_int($input))
				$value = '\''.$forum_db-&gt;escape($input).'\'';
			else
				$value = 'NULL';

			$query = array(
				'UPDATE'	=&gt; 'config',
				'SET'		=&gt; 'conf_value='.$value,
				'WHERE'		=&gt; 'conf_name=\'o_'.$forum_db-&gt;escape($key).'\''
			);

			($hook = get_hook('aop_qr_update_permission_option')) ? eval($hook) : null;
			$forum_db-&gt;query_build($query) or error(__FILE__, __LINE__);
		}
	}

	// Regenerate the config cache
	if (!defined('FORUM_CACHE_FUNCTIONS_LOADED'))
		require FORUM_ROOT.'include/cache.php';

	generate_config_cache();

	($hook = get_hook('aop_pre_redirect')) ? eval($hook) : null;

	redirect(forum_link($forum_url['admin_settings_'.$section]), $lang_admin_settings['Settings updated'].' '.$lang_admin_common['Redirect']);
}

if (!$section || $section == 'setup')
{
	// Setup the form
	$forum_page['group_count'] = $forum_page['item_count'] = $forum_page['fld_count'] = 0;

	// Setup breadcrumbs
	$forum_page['crumbs'] = array(
		array($forum_config['o_board_title'], forum_link($forum_url['index'])),
		array($lang_admin_common['Forum administration'], forum_link($forum_url['admin_index'])),
		array($lang_admin_common['Settings'], forum_link($forum_url['admin_settings_setup'])),
		array($lang_admin_common['Setup'], forum_link($forum_url['admin_settings_setup']))
	);

	($hook = get_hook('aop_setup_pre_header_load')) ? eval($hook) : null;

	define('FORUM_PAGE_SECTION', 'settings');
	define('FORUM_PAGE', 'admin-settings-setup');
	require FORUM_ROOT.'header.php';

	// START SUBST - &lt;!-- forum_main --&gt;
	ob_start();

	($hook = get_hook('aop_setup_output_start')) ? eval($hook) : null;

?&gt;
	&lt;div class=&quot;main-content main-frm&quot;&gt;
		&lt;form class=&quot;frm-form&quot; method=&quot;post&quot; accept-charset=&quot;utf-8&quot; action=&quot;&lt;?php echo forum_link($forum_url['admin_settings_setup']) ?&gt;&quot;&gt;
			&lt;div class=&quot;hidden&quot;&gt;
				&lt;input type=&quot;hidden&quot; name=&quot;csrf_token&quot; value=&quot;&lt;?php echo generate_form_token(forum_link($forum_url['admin_settings_setup'])) ?&gt;&quot; /&gt;
				&lt;input type=&quot;hidden&quot; name=&quot;form_sent&quot; value=&quot;1&quot; /&gt;
			&lt;/div&gt;
				&lt;div class=&quot;content-head&quot;&gt;
					&lt;h2 class=&quot;hn&quot;&gt;&lt;span&gt;&lt;?php echo $lang_admin_settings['Setup personal'] ?&gt;&lt;/span&gt;&lt;/h2&gt;
				&lt;/div&gt;

...&lt;snip&gt;...
[/sourcecode]
