---
layout: post
title: Leather Straps - XSS
tags:
- Cross-Site Scripting (XSS)
- PHP
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _aktt_hash_meta: ''
  aktt_notify_twitter: 'yes'
  aktt_tweeted: '1'
  _headspace_page_title: Leather Straps - XSS
  _headspace_description: Cross-Site Scripting affecting the Press This WordPress
    plugin
  _sexybookmarks_shortUrl: http://bit.ly/c5h5g2
  _sexybookmarks_permaHash: 9172d2b4f059f68368c7875a22f0a51c
---
## Details
<strong>__Affected Software:__ Press This component for WordPress
</strong>

<strong>__Fixed in Version:__  2.8.6</strong>

<strong>__Issue Type:__ Cross-Site Scripting
</strong>

<strong>Original Code: </strong><a title="Leather Straps" href="http://spotthevuln.com/2009/12/leather-straps/" target="_blank">Found Here</a>
## Description
This was a Cross-Site Scripting bug affecting the Press This component for WordPress. The developers fixed several XSS bugs in this single code change. I find two things that are interesting in this code change. First, all the XSS bugs seem to be fixed using different encoding functions. $title now has strip_tags applied, while $selection has htmlspecialchars() AND html_entity_decode() applied!  Later we see wp_htmledit_pre() used to encode output in one place and wp_richedit_pre() used a few lines down. We also see see esc_html() used to escape output for the same variable in the same code change. These kinds of deltas make code maintenance difficult, as the developer (and tester) has to understand why the particular encoding methods were applied for each situation and what the differences between the encoding methods exist. The second item I find interesting in this fix has to do with the original, vulnerable code. If we take a look at the following line of source that was removed:
<blockquote>&lt;?php if ($selection) echo wp_richedit_pre(htmlspecialchars_decode($selection)); ?&gt;</blockquote>
Here we see that the develop explicitly used the htmlspecialchars_decode() function before echoing the contents of $selection to the user. This could be an indication that this particular developer doesn't understand the code symptoms that introduce XSS as it takes an otherwise safe code segment and makes it more vulnerable to XSS conditions. Conditions like this are good justification to provide some targeted training to this particular group or even individual developer.
## Developer's Solution
```diff

&lt;?php

-$title = isset($_GET['t']) ? esc_html(aposfix(stripslashes($_GET['t']))) : '';
-$selection = isset($_GET['s']) ? trim( aposfix( stripslashes($_GET['s']) ) ) : '';
+$title = isset( $_GET['t'] ) ? trim( strip_tags( aposfix( stripslashes( $_GET['t'] ) ) ) ) : '';
+$selection = isset( $_GET['s'] ) ? trim( htmlspecialchars( html_entity_decode( aposfix( stripslashes( $_GET['s'] ) ) ) ) ) : '';

if ( ! empty($selection) ) {
$selection = preg_replace('/(\r?\n|\r)/', '&lt;/p&gt;&lt;p&gt;', $selection);
}

...

&lt;h2&gt;&lt;label for="embed-code"&gt;&lt;?php _e('Embed Code') ?&gt;&lt;/label&gt;&lt;/h2&gt;
&lt;div&gt;
-        &lt;textarea name="embed-code" id="embed-code" rows="8" cols="40"&gt;&lt;?php echo format_to_edit($selection, true); ?&gt;&lt;/textarea&gt;
+        &lt;textarea name="embed-code" id="embed-code" rows="8" cols="40"&gt;&lt;?php echo wp_htmledit_pre( $selection ); ?&gt;&lt;/textarea&gt;
&lt;p id="options"&gt;&lt;a href="#"&gt;&lt;?php _e('Insert Video'); ?&gt;&lt;/a&gt; &lt;a href="#"&gt;&lt;?php _e('Cancel'); ?&gt;&lt;/a&gt;&lt;/p&gt;
&lt;/div&gt;

...

&lt;div&gt;
&lt;textarea name="content" id="content" style="width:100%;" rows="15"&gt;
-        &lt;?php if ($selection) echo wp_richedit_pre(htmlspecialchars_decode($selection)); ?&gt;
-        &lt;?php if ($url) { echo '&lt;p&gt;'; if($selection) _e('via '); echo "&lt;a href='$url'&gt;$title&lt;/a&gt;."; echo '&lt;/p&gt;'; } ?&gt;
+        &lt;?php if ($selection) echo wp_richedit_pre( $selection ); ?&gt;
+        &lt;?php if ($url) { echo '&lt;p&gt;'; if($selection) _e('via '); printf( "&lt;a href='%s'&gt;%s&lt;/a&gt;.", esc_url( $url ), esc_html( $title ) ); echo '&lt;/p&gt;'; } ?&gt;
&lt;/textarea&gt;
&lt;/div&gt;

```
