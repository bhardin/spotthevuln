---
layout: post
title: Wood - SQL Injection
tags:
- All
- dynamic SQL
- PHP
- Solution
- SQL Injection
- SQLi
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
  _sexybookmarks_shortUrl: http://bit.ly/gP94FH
  _sexybookmarks_permaHash: b26d5d93f1b8f63a9ab1a0a76de58a4e
---
<h3>Details</h3>
__Affected Software:__ WordPress Core

__Fixed in Version:__  2.2

__Issue Type:__ SQL Injection

Original Code: <a title="Wood" href="http://spotthevuln.com/2011/01/wood-2/" target="_blank">Found    Here</a>
<h3>Description</h3>
This is a fairly straight forward SQL Injection bug here. First, although we can't see exactly where $args[] is set, we have some strong clues that it contains user/attacker controlled data. For example, the first function on the code snippet wp_newCategory() takes an $args parameter and the first thing it does is escape the values within the array. The names of variables holding various values in the array also provide clues that $args cannot be trusted.

On line 66 we see that $max_results is assigned the value from $args[4]. $max_results is then used to build a portion of a SQL string which is assigned to the $limit variable. $limit is then passed to the end of a SQL statement on line 84, resulting in SQL injection. Some readers may point out that $args is escaped in on line 60, before it is used to build any SQL statement. Unfortunately, escaping values in this case doesn't prevent SQL injection. The attacker controlled value is eventually used to build a LIMIT clause. The LIMIT clause doesn't enclose the attacker supplied values within quotes, so there are no quotes to break out of.

The developers addressed this issue by casting args[4] to int during assignment to $max_results. If args[4] contains any characters that do not qualify as an integer, the value will not be passed to the LIMIT statement.


<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="60,66,67,84"]
&lt;?php
...snip...
	function wp_newCategory($args) {
		$this-&gt;escape($args);

		$blog_id				= (int) $args[0];
		$username				= $args[1];
		$password				= $args[2];
		$category				= $args[3];

		if(!$this-&gt;login_pass_ok($username, $password)) {
			return($this-&gt;error);
		}

		// Set the user context and make sure they are
		// allowed to add a category.
		set_current_user(0, $username);
		if(!current_user_can(&quot;manage_categories&quot;, $page_id)) {
			return(new IXR_Error(401, __(&quot;Sorry, you do not have the right to add a category.&quot;)));
		}

		// We need this to make use of the wp_insert_category()
		// funciton.
		require_once(ABSPATH . &quot;wp-admin/admin-db.php&quot;);

		// If no slug was provided make it empty so that
		// WordPress will generate one.
		if(empty($category[&quot;slug&quot;])) {
			$category[&quot;slug&quot;] = &quot;&quot;;
		}

		// If no parent_id was provided make it empty
		// so that it will be a top level page (no parent).
		if ( !isset($category[&quot;parent_id&quot;]) )
			$category[&quot;parent_id&quot;] = &quot;&quot;;

		// If no description was provided make it empty.
		if(empty($category[&quot;description&quot;])) {
			$category[&quot;description&quot;] = &quot;&quot;;
		}

		$new_category = array(
			&quot;cat_name&quot;				=&gt; $category[&quot;name&quot;],
			&quot;category_nicename&quot;		=&gt; $category[&quot;slug&quot;],
			&quot;category_parent&quot;		=&gt; $category[&quot;parent_id&quot;],
			&quot;category_description&quot;	=&gt; $category[&quot;description&quot;]
		);

		$cat_id = wp_insert_category($new_category);
		if(!$cat_id) {
			return(new IXR_Error(500, __(&quot;Sorry, the new category failed.&quot;)));
		}

		return($cat_id);
	}


	function wp_suggestCategories($args) {
		global $wpdb;

		$this-&gt;escape($args);

		$blog_id				= (int) $args[0];
		$username				= $args[1];
		$password				= $args[2];
		$category				= $args[3];
-		$max_results			= $args[4];
+		$max_results            = (int) $args[4];

		if(!$this-&gt;login_pass_ok($username, $password)) {
			return($this-&gt;error);
		}

		// Only set a limit if one was provided.
		$limit = &quot;&quot;;
		if(!empty($max_results)) {
			$limit = &quot;LIMIT {$max_results}&quot;;
		}

		$category_suggestions = $wpdb-&gt;get_results(&quot;
			SELECT cat_ID category_id,
				cat_name category_name
			FROM {$wpdb-&gt;categories}
			WHERE cat_name LIKE '{$category}%'
			{$limit}
		&quot;);

		return($category_suggestions);
	}


	/* Blogger API functions
	 * specs on http://plant.blogger.com/api and http://groups.yahoo.com/group/bloggerDev/
	 */


	/* blogger.getUsersBlogs will make more sense once we support multiple blogs */
	function blogger_getUsersBlogs($args) {

		$this-&gt;escape($args);

		$user_login = $args[1];
		$user_pass  = $args[2];

		if (!$this-&gt;login_pass_ok($user_login, $user_pass)) {
			return $this-&gt;error;
		}

		set_current_user(0, $user_login);
		$is_admin = current_user_can('level_8');

		$struct = array(
			'isAdmin'  =&gt; $is_admin,
			'url'      =&gt; get_option('home') . '/',
			'blogid'   =&gt; '1',
			'blogName' =&gt; get_option('blogname')
		);

		return array($struct);
	}

...snip...
?&gt;
[/sourcecode]
