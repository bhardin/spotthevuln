---
layout: post
title: Reasoning - Cross-Site Scripting
tags:
- Consumption
- Cross-Site Scripting (XSS)
- FreePBX
- html encoding
- PHP
- Solution
status: publish
type: post
published: true
meta:
  _syntaxhighlighter_encoded: '1'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  aktt_notify_twitter: 'yes'
  _sexybookmarks_shortUrl: http://bit.ly/fy2Oik
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: c3c009f6685fd78a4325fc409779580c
---
<h3>Details</h3>
__Affected Software:__ FreePBX

__Fixed in Version:__  2.9

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Radical" href="http://spotthevuln.com/2011/02/reasoning/" target="_blank">Found    Here</a>
<h3>Description</h3>
To be honest, I was a little confused by this week's patch. There are several XSS bugs in this code. Originally, the vulnerable code would take a tainted $_REQUEST value (a value from a GET, POST, or cookie) and assign the tainted value to a couple of different PHP variables ($description and $notes in particular). The application then uses of these tainted values on lines 136 and 140, resulting in XSS. The developer addressed these XSS issues by html encoding the $_REQUEST values before assigning them to PHP variables. In the code mentioned above, the developer decided to encode/sanitize at the point of assignment (as opposed to the point of consumption). There are differing perspectives as to whether one should encode/sanitize upon assignment or consumption, but the truth is both methods work.

What's confusing is the code sample contains many symptoms that are exactly like the vulnerable code patched by this security patch. $type, $action, $old_custom_dest, and $custom_dest are all set in exactly the same way the patched assignments were. For some reason, the developer chose to ignore these assignments even though they are only a few lines away. Also, instead of encoding at the point of assignment (like they did for $description and $notes), the developer chose to change styles and encode at the point of consumption for one of the tainted variables (see line 96 and 97). What's even more confusing is only 4 lines later, we see the developer missed the same tainted variable used in an echo and failed to encode the tainted $custom_dest variable resulting in XSS. Lines 77 - 79 also contain XSS vulnerabilities that were missed in this patch.

<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="11-14,77,78,79,96,97,136,140"]
&lt;?php
$tabindex = 0;
$display = 'customdests';

$type   = isset($_REQUEST['type']) ? $_REQUEST['type'] : 'tool';
$action = isset($_REQUEST['action']) ? $_REQUEST['action'] : '';
if (isset($_REQUEST['delete'])) $action = 'delete';

$old_custom_dest = isset($_REQUEST['old_custom_dest']) ? $_REQUEST['old_custom_dest'] :  '';
$custom_dest     = isset($_REQUEST['extdisplay']) ? $_REQUEST['extdisplay'] :  '';
-$description     = isset($_REQUEST['description']) ? $_REQUEST['description'] :  '';
-$notes           = isset($_REQUEST['notes']) ? $_REQUEST['notes'] :  '';
+$description     = isset($_REQUEST['description']) ? htmlentities($_REQUEST['description']) :  '';
+$notes           = isset($_REQUEST['notes']) ? htmlentities($_REQUEST['notes']) :  '';

switch ($action) {
	case 'add':
		if (customappsreg_customdests_add($custom_dest, $description, $notes)) {
			needreload();
			redirect_standard();
		} else {
			$custom_dest='';
		}
	break;
	case 'edit':
		if (customappsreg_customdests_edit($old_custom_dest, $custom_dest, $description, $notes)) {
			needreload();
			redirect_standard('extdisplay');
		}
	break;
	case 'delete':
		customappsreg_customdests_delete($custom_dest);
		needreload();
		redirect_standard();
	break;
}

?&gt;
&lt;/div&gt;

&lt;div class=&quot;rnav&quot;&gt;&lt;ul&gt;
&lt;?php

echo '&lt;li&gt;&lt;a href=&quot;config.php?display='.$display.'&amp;amp;type='.$type.'&quot;&gt;'._('Add Custom Destination').'&lt;/a&gt;&lt;/li&gt;';

foreach (customappsreg_customdests_list() as $row) {
	$descr = $row['description'] != '' ? $row['description'] : '('.$row['custom_dest'].')';
	echo '&lt;li&gt;&lt;a href=&quot;config.php?display='.$display.'&amp;amp;type='.$type.'&amp;amp;extdisplay='.$row['custom_dest'].'&quot; class=&quot;&quot;&gt;'.$descr.'&lt;/a&gt;&lt;/li&gt;';
}

?&gt;
&lt;/ul&gt;&lt;/div&gt;

&lt;div class=&quot;content&quot;&gt;

&lt;?php

if ($custom_dest != '') {
	// load
	$usage_list = framework_display_destination_usage(customappsreg_customdests_getdest($custom_dest));

	$row = customappsreg_customdests_get($custom_dest);

	$description = $row['description'];
	$notes       = $row['notes'];

	$disp_description = $row['description'] != '' ? $row['description'] : '('.$row['custom_dest'].')';
	echo &quot;&lt;h2&gt;&quot;._(&quot;Edit: &quot;).&quot;$disp_description&quot;.&quot;&lt;/h2&gt;&quot;;
} else {
	echo &quot;&lt;h2&gt;&quot;._(&quot;Add Custom Destination&quot;).&quot;&lt;/h2&gt;&quot;;
}

$helptext = _(&quot;Custom Destinations allows you to register your custom destinations that point to custom dialplans and will also 'publish' these destinations as available destinations to other modules. This is an advanced feature and should only be used by knowledgeable users. If you are getting warnings or errors in the notification panel about CUSTOM destinations that are correct, you should include them here. The 'Unknown Destinations' chooser will allow you to choose and insert any such destinations that the registry is not aware of into the Custom Destination field.&quot;);
echo $helptext;
?&gt;

&lt;form name=&quot;editCustomDest&quot; action=&quot;&lt;?php  $_SERVER['PHP_SELF'] ?&gt;&quot; method=&quot;post&quot; onsubmit=&quot;return checkCustomDest(editCustomDest);&quot;&gt;
	&lt;input type=&quot;hidden&quot; name=&quot;extdisplay&quot; value=&quot;&lt;?php echo $custom_dest; ?&gt;&quot;&gt;
	&lt;input type=&quot;hidden&quot; name=&quot;old_custom_dest&quot; value=&quot;&lt;?php echo $custom_dest; ?&gt;&quot;&gt;
	&lt;input type=&quot;hidden&quot; name=&quot;action&quot; value=&quot;&lt;?php echo ($custom_dest != '' ? 'edit' : 'add'); ?&gt;&quot;&gt;
	&lt;table&gt;
	&lt;tr&gt;&lt;td colspan=&quot;2&quot;&gt;&lt;h5&gt;&lt;?php  echo ($custom_dest ? _(&quot;Edit Custom Destination&quot;) : _(&quot;Add Custom Destination&quot;)) ?&gt;&lt;hr&gt;&lt;/h5&gt;&lt;/td&gt;&lt;/tr&gt;
	&lt;tr&gt;
		&lt;td&gt;&lt;a href=&quot;#&quot; class=&quot;info&quot;&gt;&lt;?php echo _(&quot;Custom Destination&quot;)?&gt;:
			&lt;span&gt;
				&lt;?php
				echo _(&quot;This is the Custom Destination to be published. It should be formatted exactly as you would put it in a goto statement, with context, exten, priority all included. An example might look like:&lt;br /&gt;mycustom-app,s,1&quot;);
				if (!empty($usage_list)) {
					echo &quot;&lt;br /&gt;&quot;._(&quot;READONLY WARNING: Because this destination is being used by other module objects it can not be edited. You must remove those dependencies in order to edit this destination, or create a new destination to use&quot;);
				}
				?&gt;
			&lt;/span&gt;&lt;/a&gt;&lt;/td&gt;
	&lt;?php
	if (!empty($usage_list)) {
	?&gt;
-		&lt;td&gt;&lt;b&gt;&lt;?php echo $custom_dest; ?&gt;&lt;/b&gt;&lt;/td&gt;
+	    &lt;td&gt;&lt;b&gt;&lt;?php echo htmlentities($custom_dest); ?&gt;&lt;/b&gt;&lt;/td&gt;
	&lt;?php
	} else {
	?&gt;
		&lt;td&gt;&lt;input size=&quot;30&quot; type=&quot;text&quot; name=&quot;extdisplay&quot; id=&quot;extdisplay&quot; value=&quot;&lt;?php  echo $custom_dest; ?&gt;&quot; tabindex=&quot;&lt;?php echo ++$tabindex;?&gt;&quot;&gt;&lt;/td&gt;
	&lt;?php
	}
	?&gt;
	&lt;/tr&gt;

	&lt;?php
	if (empty($usage_list)) {
	?&gt;
	&lt;tr&gt;
		&lt;td&gt;
		&lt;a href=# class=&quot;info&quot;&gt;&lt;?php echo _(&quot;Destination Quick Pick&quot;)?&gt;
			&lt;span&gt;
				&lt;?php echo _(&quot;Choose un-identified destinations on your system to add to the Custom Destination Registry. This will insert the chosen entry into the Custom Destination box above.&quot;)?&gt;
			&lt;/span&gt;
		&lt;/a&gt;
		&lt;/td&gt;
		&lt;td&gt;
			&lt;select onChange=&quot;insertDest();&quot; id=&quot;insdest&quot; tabindex=&quot;&lt;?php echo ++$tabindex;?&gt;&quot;&gt;
				&lt;option value=&quot;&quot;&gt;&lt;?php echo _(&quot;(pick destination)&quot;)?&gt;&lt;/option&gt;
	&lt;?php
				$results = customappsreg_customdests_getunknown();
				foreach ($results as $thisdest) {
					echo &quot;&lt;option value='$thisdest'&gt;$thisdest&lt;/option&gt;\n&quot;;
				}
	?&gt;
			&lt;/select&gt;
		&lt;/td&gt;
	&lt;/tr&gt;
	&lt;?php
	}
	?&gt;

	&lt;tr&gt;
		&lt;td&gt;&lt;a href=&quot;#&quot; class=&quot;info&quot;&gt;&lt;?php echo _(&quot;Description&quot;)?&gt;:&lt;span&gt;&lt;?php echo _(&quot;Brief Description that will be published to modules when showing destinations. Example: My Weather App&quot;)?&gt;&lt;/span&gt;&lt;/a&gt;&lt;/td&gt;
		&lt;td&gt;&lt;input size=&quot;30&quot; type=&quot;text&quot; name=&quot;description&quot; value=&quot;&lt;?php  echo $description; ?&gt;&quot; tabindex=&quot;&lt;?php echo ++$tabindex;?&gt;&quot;&gt;&lt;/td&gt;
	&lt;/tr&gt;
	&lt;tr&gt;
		&lt;td valign=&quot;top&quot;&gt;&lt;a href=&quot;#&quot; class=&quot;info&quot;&gt;&lt;?php echo _(&quot;Notes&quot;)?&gt;:&lt;span&gt;&lt;?php echo _(&quot;More detailed notes about this destination to help document it. This field is not used elsewhere.&quot;)?&gt;&lt;/span&gt;&lt;/a&gt;&lt;/td&gt;
		&lt;td&gt;&lt;textarea name=&quot;notes&quot; cols=&quot;23&quot; rows=&quot;6&quot; tabindex=&quot;&lt;?php echo ++$tabindex;?&gt;&quot;&gt;&lt;?php echo $notes; ?&gt;&lt;/textarea&gt;&lt;/td&gt;
	&lt;/tr&gt;

	&lt;tr&gt;
		&lt;td colspan=&quot;2&quot;&gt;&lt;br&gt;&lt;input name=&quot;Submit&quot; type=&quot;submit&quot; value=&quot;&lt;?php echo _(&quot;Submit Changes&quot;)?&gt;&quot; tabindex=&quot;&lt;?php echo ++$tabindex;?&gt;&quot;&gt;
		&lt;?php if ($custom_dest != '') { echo '&amp;nbsp;&lt;input name=&quot;delete&quot; type=&quot;submit&quot; value=&quot;'._(&quot;Delete&quot;).'&quot;&gt;'; } ?&gt;
		&lt;/td&gt;

		&lt;?php
		if ($custom_dest != '') {
			if (!empty($usage_list)) {
			?&gt;
				&lt;tr&gt;&lt;td colspan=&quot;2&quot;&gt;
				&lt;a href=&quot;#&quot; class=&quot;info&quot;&gt;&lt;?php echo $usage_list['text']?&gt;:&lt;span&gt;&lt;?php echo $usage_list['tooltip']?&gt;&lt;/span&gt;&lt;/a&gt;
				&lt;/td&gt;&lt;/tr&gt;
			&lt;?php
			}
		}
		?&gt;
	&lt;/tr&gt;
	&lt;/table&gt;
	&lt;/form&gt;
...snip...
&lt;/script&gt;
[/sourcecode]
