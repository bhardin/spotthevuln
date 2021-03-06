---
layout: post
title: Anyway - Cross-Site Scripting
tags:
- All
- arbitrary values
- Cross-Site Scripting (XSS)
- markup
- parameters
- PHP
- plugins
- Privilege Escalation
- Solution
- Wordpress
- wordpress plugin
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '1'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/gTLPvV
  _sexybookmarks_permaHash: 7449a576a7c7bef2fb70e3efd2244d74
---
<h3>Details</h3>
__Affected Software:__ The Hackers Diet

__Fixed in Version:__  9.7b

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Anyway" href="http://spotthevuln.com/2010/12/anyway/" target="_blank">Found    Here</a>
<h3>Description</h3>
First, some logistics... the code we're looking at belongs to the "Weight_save.php" file which is part of the "Hackers Diet" WordPress plugin. This plugin was created to "Help you track and predict weight loss using your Wordpress blog". A single change was made to the Weight_save.php for this changelist. The single change simply removes an obvious XSS bug in which POST parameters are printed to HTML markup. This line was likely being used for debugging purposes and was forgotten during release to production. A more robust testing and release process would have caught this.

Although the developers prevented an XSS vulnerability, they completely overlooked several other issues. IndigoMann (via blog comments) noticed XSS and SQL Injection in this code... The one issue that jumps out at me is this line:
<blockquote>$user_id = $_POST["user"];</blockquote>
I always become very suspicious when an application passes user_id's back and forth. Ideally, this data should be stored via session state, otherwise an attacker could pass an arbitrary value and access (and possibly update) another user's data. Looking at the code, it appears this may be the case with the Hacker Diet plugin.
<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="14,15"]
&lt;?
// get our db settings without loading all of wordpress every save
$html = implode('', file(&quot;../../../wp-config.php&quot;));
$html = str_replace (&quot;require_once&quot;, &quot;// &quot;, $html);
$html = str_replace (&quot;&lt;?php&quot;, &quot;&quot;, $html);
$html = str_replace (&quot;?&gt;&quot;, &quot;&quot;, $html);
eval($html);

if (isset($_POST[&quot;id&quot;]) &amp;&amp; isset($_POST[&quot;user&quot;]) &amp;&amp; is_numeric($_POST[&quot;user&quot;]) &amp;&amp; isset($_POST[&quot;content&quot;]) &amp;&amp; is_numeric($_POST[&quot;content&quot;])) {
	$date = substr($_POST[&quot;id&quot;], 7);
	$user_id = $_POST[&quot;user&quot;];
	$weight = round($_POST[&quot;content&quot;], 1);
} else {
-	print_r($_POST);
+	//print_r($_POST);
	echo &quot;Please enter a valid number for your weight.&quot;;
	exit;
}

mysql_connect(DB_HOST, DB_USER, DB_PASSWORD);
mysql_select_db(DB_NAME);

$query = &quot;update &quot;.$table_prefix.&quot;hackdiet_weightlog set weight = $weight where wp_id = $user_id and date = \&quot;&quot;.date(&quot;Y-m-d&quot;, $date).&quot;\&quot;&quot;;
mysql_query($query);
if (mysql_affected_rows() != 1) {
	// record doesn't exist yet, lets create it
	$query = &quot;insert into &quot;.$table_prefix.&quot;hackdiet_weightlog set date = \&quot;&quot;.date(&quot;Y-m-d&quot;, $date).&quot;\&quot;, weight = $weight, wp_id = $user_id&quot;;
	mysql_query($query);
	if (mysql_affected_rows() != 1) {
		echo &quot;Save failed. - &quot; . mysql_error();
		exit();
	} else {
		echo htmlspecialchars($weight);
	}
} else {
	echo htmlspecialchars($weight);
}

$query = &quot;select trend from &quot;.$table_prefix.&quot;hackdiet_weightlog where wp_id = $user_id and date &lt; \&quot;&quot;.date(&quot;Y-m-d&quot;, $date).&quot;\&quot; order by date desc limit 1&quot;;
$result = mysql_query($query);
if (mysql_num_rows($result) == 1) {
	$trend = mysql_result($result, 0);
	$use_first_weight_as_trend = false;
} else {
	// no trends exist below this entry, we must be first. so in next query, we need to grab today's weight to be trend 1
	$use_first_weight_as_trend = true;
}

$query = &quot;select date, weight, trend from &quot;.$table_prefix.&quot;hackdiet_weightlog where wp_id = $user_id and date &gt;= \&quot;&quot;.date(&quot;Y-m-d&quot;, $date).&quot;\&quot; order by date asc&quot;;
$result = mysql_query($query);
while ($entry = mysql_fetch_assoc($result)) {
	if ($use_first_weight_as_trend) {
		$trend = $entry[&quot;weight&quot;];
		$use_first_weight_as_trend = false;
	} else {
		// exponentially smoothed moving average with 10% smoothing
		$trend = $trend + 0.1 * ($entry[&quot;weight&quot;] - $trend);
	}
	$entry[&quot;trend&quot;] = $trend;
	$weights[] = $entry;
}

foreach ($weights as $entry) {
	$query = &quot;update &quot;.$table_prefix.&quot;hackdiet_weightlog set trend = &quot;.round($entry[&quot;trend&quot;], 1).&quot; where wp_id = $user_id and date = \&quot;&quot;.$entry[&quot;date&quot;].&quot;\&quot;&quot;;
	mysql_query($query);
}

// 0 will always be the edited date, since the list contains the edited entry + all the ones after it, sorted asc.
$dif = round($weights[0][&quot;weight&quot;] - $weights[0][&quot;trend&quot;], 1);

echo &quot;&lt;span class=\&quot;trend_dif &quot;.(($dif &lt; 0)?&quot;good_trend&quot;:&quot;bad_trend&quot;).&quot;\&quot;&gt;$dif&lt;/span&gt;&quot;;
?&gt;
[/sourcecode]
