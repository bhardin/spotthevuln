---
layout: post
title: Errors - Cross-Site Scripting
tags:
- All
- attacker
- classic symptom
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- html markup
- markup
- PHP
- POST
- PunBB
- Solution
status: publish
type: post
published: true
meta:
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_notify_twitter: 'yes'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/cYrL80
  _sexybookmarks_permaHash: 7462de80a4a9677dd2baf049c88b67a3
---
## Details
__Affected Software:__ PunBB

__Fixed in Version:__  1.2

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Errors" href="http://spotthevuln.com/2010/08/errors/" target="_blank">Found    Here</a>
## Description
This weeks' example was a XSS bug that affected PunBB. The vulnerable code took attacker controlled variables directly from POST parameters and used those values for various operations. Specifically, the $_POST['prune_sticky'] value was used in several places without any form of sanitization. Looking through the patch submitted by the PunBB developers we see that the unsantized value was passed to a function named prune() and also echoed in HTML markup. PHP echo of a $_POST variable is a classic symptom of XSS. The PunBB developers addressed this issue by sanitizing the $_POST['prune_sticky'] value before echoing its value in HTML markup. The developer forces the $_POST['prune_sticky'] value to either 1 or 0, which eliminates the possibility or arbitrary script being injected via the $prune_sticky variable. The $_POST['prune_sticky'] value is sanitized here:
<blockquote>$prune_sticky = isset($_POST['prune_sticky']) ? '1' : '0';</blockquote>
In addition to the changes made to PHP echo there were other changes checked in the by the PunBB developers, most notably the sanitizing of values passed to the prune() function. Using the code snippet provided here, it's difficult to understand exactly what is accomplished by the prune() function. Further variable tracing will be needed in order to determine the danger associated with passing a tainted value to prune(). The developers however felt it was necessary to sanitize the $prune_sticky value before passing it to prune().
## Developer's Solution
```diff
if (isset($_GET['action']) || isset($_POST['prune']) || isset($_POST['prune_comply']))
{
if (isset($_POST['prune_comply']))
{
confirm_referrer('admin_prune.php');

$prune_from = $_POST['prune_from'];
+$prune_sticky = isset($_POST['prune_sticky']) ? '1' : '0';
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

-prune($fid, $_POST['prune_sticky'], $prune_date);
+prune($fid, $prune_sticky, $prune_date);
update_forum($fid);
}
}
else
{
$prune_from = intval($prune_from);
-prune($prune_from, $_POST['prune_sticky'], $prune_date);
+prune($prune_from, $prune_sticky, $prune_date);
update_forum($prune_from);
}

// Locate any "orphaned redirect topics" and delete them
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

...&lt;snip&gt;...

&lt;div&gt;
&lt;h2&gt;&lt;span&gt;Prune&lt;/span&gt;&lt;/h2&gt;
&lt;div&gt;
&lt;form method="post" action="admin_prune.php?action=foo"&gt;
&lt;div&gt;
&lt;input type="hidden" name="prune_days" value="&lt;?php echo $prune_days ?&gt;" /&gt;
-&lt;input type="hidden" name="prune_sticky" value="&lt;?php echo $_POST['prune_sticky'] ?&gt;" /&gt;
+&lt;input type="hidden" name="prune_sticky" value="&lt;?php echo $prune_sticky ?&gt;" /&gt;
&lt;input type="hidden" name="prune_from" value="&lt;?php echo $prune_from ?&gt;" /&gt;
&lt;fieldset&gt;
&lt;legend&gt;Confirm prune posts&lt;/legend&gt;
&lt;div&gt;
&lt;p&gt;Are you sure that you want to prune all topics older than &lt;?php echo $prune_days ?&gt; days from &lt;?php echo $forum ?&gt;? (&lt;?php echo $num_topics ?&gt; topics)&lt;/p&gt;
&lt;p&gt;WARNING! Pruning posts deletes them permanently.&lt;/p&gt;
&lt;/div&gt;
&lt;/fieldset&gt;
&lt;/div&gt;
&lt;p&gt;&lt;input type="submit" name="prune_comply" value="Prune" /&gt;&lt;a href="javascript:history.go(-1)"&gt;Go back&lt;/a&gt;&lt;/p&gt;
```
