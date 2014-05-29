---
layout: post
title: Leather Straps
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#code #development'
  _edit_last: '1'
  aktt_tweeted: '1'
  _wp_old_slug: leather-straps-week-19
  bfa_ata_display_body_title: ''
  bfa_ata_body_title: Leather Straps
  bfa_ata_body_title_multi: Leather Straps
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  _sexybookmarks_permaHash: b12f13cf65bd9b0bc982adc42eae5e71
  _sexybookmarks_shortUrl: http://bit.ly/aHkxTJ
---
<blockquote>Some mornings, it's just not worth chewing through the leather straps

- Emo Phillips</blockquote>
```php
<%php

...

$title = isset($_GET['t']) ? esc_html(aposfix(stripslashes($_GET['t']))) : '';

$selection = isset($_GET['s']) ? trim( aposfix( stripslashes($_GET['s']) ) ) : '';

if ( ! empty($selection) ) {

$selection = preg_replace('/(\r?\n|\r)/', '&lt;/p&gt;&lt;p&gt;', $selection);

}

...

&lt;h2&gt;&lt;label for="embed-code"&gt;&lt;?php _e('Embed Code') ?&gt;&lt;/label&gt;&lt;/h2&gt;

&lt;div class="inside"&gt;

&lt;textarea name="embed-code" id="embed-code" rows="8" cols="40"&gt;&lt;?php echo format_to_edit($selection, true); ?&gt;&lt;/textarea&gt;

&lt;p id="options"&gt;&lt;a href="#" class="select button"&gt;&lt;?php _e('Insert Video'); ?&gt;&lt;/a&gt; &lt;a href="#" class="close button"&gt;&lt;?php _e('Cancel'); ?&gt;&lt;/a&gt;&lt;/p&gt;

&lt;/div&gt;

...

&lt;div class="editor-container"&gt;

&lt;textarea name="content" id="content" style="width:100%;" class="mceEditor" rows="15"&gt;

&lt;?php if ($selection) echo wp_richedit_pre(htmlspecialchars_decode($selection)); ?&gt;

&lt;?php if ($url) { echo '&lt;p&gt;'; if($selection) _e('via '); echo "&lt;a href='$url'&gt;$title&lt;/a&gt;."; echo '&lt;/p&gt;'; } ?&gt;

&lt;/textarea&gt;

&lt;/div&gt;

```
