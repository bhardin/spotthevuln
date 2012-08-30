---
layout: post
title: Invincible
tags:
- Code Snippet
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
<blockquote><strong>In ancient times skillful warriors first made themselves invincible, and then watched for vulnerability in their opponents.
Sun Tzu</strong></blockquote>
[sourcecode language="php"]
&lt;?php

# Visit this file in your browser to simulate a mobile device's screensize via an &lt;iframe&gt;

$devices = array(
	'iphone_p' =&gt; array(
		'type'   =&gt; 'iPhone: portrait (320x480)',
		'width'  =&gt; 320,
		'height' =&gt; 480
	),
	'iphone_l' =&gt; array(
		'type'   =&gt; 'iPhone: landscape (480x320)',
		'width'  =&gt; 480,
		'height' =&gt; 320
	),
	'moto'	   =&gt; array(
		'type'   =&gt; 'Motorola phone/browser (RAZR, v551, etc)',
		'width'  =&gt; 176,
		'height' =&gt; 220
	),
	'n80'	   =&gt; array(
		'type'   =&gt; 'Nokia N80 (N60WebKit)',
		'width'  =&gt; 352,
		'height' =&gt; 416
	)
);

if ( (int) $_REQUEST['w'] &amp;&amp; (int) $_REQUEST['h'] ) {
	$choice = array(
		'type'   =&gt; &quot;Custom size ({$_REQUEST['w']}x{$_REQUEST['h']})&quot;,
		'width'  =&gt; $_REQUEST['w'],
		'height' =&gt; $_REQUEST['h']
	);
}

elseif ( $devices[$_REQUEST['d']] )
	$choice = $devices[$_REQUEST['d']];

else $choice = $devices['iphone_p'];

?&gt;
&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD HTML 4.01 Transitional//EN&quot; &quot;http://www.w3.org/TR/html4/loose.dtd&quot;&gt;
&lt;html&gt;
&lt;head&gt;
	&lt;meta http-equiv=&quot;Content-Type&quot; content=&quot;text/html; charset=UTF-8&quot;&gt;
	&lt;title&gt;WPhone iFramer test tool: &lt;?php echo $choice['type']; ?&gt;&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
	&lt;form action=&quot;&lt;?php echo $_SERVER['PHP_SELF']; ?&gt;&quot; method=&quot;get&quot;&gt;
		&lt;label for=&quot;h&quot;&gt;CHOOSE&lt;/label&gt;
		&lt;select name=&quot;d&quot; id=&quot;d&quot;&gt;
			&lt;option&gt;&lt;/option&gt;
&lt;?php
			foreach ( $devices as $this_d_key =&gt; $this_d ) {
				$selected = ( $_REQUEST['d'] == $this_d_key ) ? 'selected' : '';
				echo '&lt;option value=&quot;' . $this_d_key . '&quot; ' . $selected . '&gt;' . $this_d['type'] . '&lt;/option&gt;' . &quot;\n\t\t\t&quot;;
			}
			echo &quot;\n&quot;;
?&gt;		
		&lt;/select&gt;
		&lt;br /&gt;OR INPUT
		&lt;label for=&quot;w&quot;&gt;Width&lt;/label&gt;
		&lt;input type=&quot;text&quot; name=&quot;w&quot; id=&quot;w&quot; value=&quot;&quot; size=&quot;5&quot; /&gt;
		x
		&lt;label for=&quot;h&quot;&gt;Height&lt;/label&gt;
		&lt;input type=&quot;text&quot; name=&quot;h&quot; id=&quot;h&quot; value=&quot;&quot; size=&quot;5&quot; /&gt;
		&lt;br /&gt;
		&lt;input type=&quot;submit&quot; name=&quot;submit&quot; value=&quot;view&quot; /&gt;
	&lt;/form&gt;
	&lt;h2&gt;&lt;?php echo $choice['type']; ?&gt;&lt;/h2&gt;
	&lt;iframe src=&quot;../../../wp-login.php&quot; width=&quot;&lt;?php echo $choice['width']; ?&gt;&quot; height=&quot;&lt;?php echo $choice['height']; ?&gt;&quot;&gt;your browser does not support iframes.&lt;/iframe&gt;
&lt;/body&gt;
&lt;/html&gt;
[/sourcecode] 
