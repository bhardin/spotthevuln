---
layout: post
title: Haircut - Cross-Site Scripting
tags:
- All
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- markup
- PHP
- plugins
- Solution
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
  _sexybookmarks_shortUrl: http://bit.ly/fykNOq
  _sexybookmarks_permaHash: 6a80bd2d0006f6cb1918e841004124ec
---
## Details
__Affected Software:__ WP-Slimbox 2

__Fixed in Version:__  1.0.1

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Haircut" href="http://spotthevuln.com/2010/11/haircut/" target="_blank">Found    Here</a>
## Description
A bit of a head fake here. There are a lot of variable assignments in this code. Lots of variable assignments results in a lot of tracing during security code audits. As a variable is set with an untrusted value, it becomes tainted. Following that variable until you find exactly where its being used is crucial in understanding whether a security bug exists or not. Any one of those variable assignments could easily result in a major security vulnerability. In this week's example, the vulnerable line came before the massive set of variable assignments. Once again, we see PHP_SELF being used to create a URL. Instead of trying to encode the value before using it in markup, the developer chose to remove the reference to PHP_SELF.

## Developer's Solution
[sourcecode language="diff" highlight="8,9"]
&lt;?php
	$easingArray = array(swing,easeInQuad,easeOutQuad,easeInOutQuad,easeInCubic,easeOutCubic,easeInOutCubic,easeInQuart,easeOutQuart,easeInOutQuart,easeInQuint,easeOutQuint,easeInOutQuint,easeInSine,easeOutSine,easeInOutSine,easeInExpo,easeOutExpo,easeInOutExpo,easeInCirc,easeOutCirc,easeInOutCirc,easeInElastic,easeOutElastic,easeInOutElastic,easeInBack,easeOutBack,easeInOutBack,easeInBounce,easeOutBounce,easeInOutBounce);
	$overlayOpacity = array(0,0.1,0.2,0.3,0.4,0.5,0.6,0.7,0.8,0.9,1);
	$msArray = array(1,100,200,300,400,500,600,700,800,900,1000);
	$captions = array('a-title','img-alt','img-title','href','None');
?&gt;
&lt;div class=&quot;wrap&quot;&gt;
-	&lt;form method=&quot;post&quot; action=&quot;&lt;?php echo $_SERVER['PHP_SELF']?&gt;?page=slimbox2options&quot; id=&quot;options&quot;&gt;&lt;?php	echo wp_nonce_field('update-options','wp_slimbox_wpnonce'); ?&gt;&lt;h2&gt;&lt;?php _e('WP Slimbox2 Plugin', 'wp-slimbox2'); ?&gt;&lt;/h2&gt;
+ 	&lt;form method=&quot;post&quot; action=&quot;&quot; id=&quot;options&quot;&gt;&lt;?php      echo wp_nonce_field('update-options','wp_slimbox_wpnonce'); ?&gt;&lt;h2&gt;&lt;?php _e('WP Slimbox2 Plugin', 'wp-slimbox2'); ?&gt;&lt;/h2&gt;
&lt;?php
	if(isset($_POST['action']) &amp;&amp; wp_verify_nonce($_POST['wp_slimbox_wpnonce'], 'update-options')) {
		$options-&gt;update_option(
			array(
				'autoload'   =&gt; $_POST['wp_slimbox_autoload'],
				'loop' =&gt; $_POST['wp_slimbox_loop'],
				'overlayOpacity'   =&gt; $_POST['wp_slimbox_overlayOpacity'],
				'overlayColor' =&gt; $_POST['wp_slimbox_overlayColor'],
				'overlayFadeDuration'   =&gt; $_POST['wp_slimbox_overlayFadeDuration'],
				'resizeDuration' =&gt; $_POST['wp_slimbox_resizeDuration'],
				'resizeEasing'   =&gt; $_POST['wp_slimbox_resizeEasing'],
				'initialWidth' =&gt; $_POST['wp_slimbox_initialWidth'],
				'initialHeight'   =&gt; $_POST['wp_slimbox_initialHeight'],
				'imageFadeDuration' =&gt; $_POST['wp_slimbox_imageFadeDuration'],
				'captionAnimationDuration'   =&gt; $_POST['wp_slimbox_captionAnimationDuration'],
				'caption' =&gt; array($_POST['wp_slimbox_caption1'],$_POST['wp_slimbox_caption2'],$_POST['wp_slimbox_caption3'],$_POST['wp_slimbox_caption4']),
				'url' =&gt; $_POST['wp_slimbox_url'],
				'selector' =&gt; $_POST['wp_slimbox_selector'],
				'counterText' =&gt; $_POST['wp_slimbox_counterText'],
				'closeKeys'   =&gt; $_POST['wp_slimbox_closeKeys'],
				'previousKeys' =&gt; $_POST['wp_slimbox_previousKeys'],
				'nextKeys'   =&gt;  $_POST['wp_slimbox_nextKeys'],
				'picasaweb' =&gt; $_POST['wp_slimbox_picasaweb'],
				'flickr'   =&gt; $_POST['wp_slimbox_flickr'],
				'mobile' =&gt; $_POST['wp_slimbox_mobile'],
				'maintenance' =&gt; $_POST['wp_slimbox_maintenance'],
				'cache'   =&gt; $_POST['wp_slimbox_cache']
			)
		);
		echo '&lt;div id=&quot;message&quot; class=&quot;updated fade&quot;&gt;&lt;p&gt;&lt;strong&gt;'.__('Settings Saved', 'wp-slimbox2').'.&lt;/strong&gt;&lt;/p&gt;&lt;/div&gt;';
	}
	$caption = $options-&gt;get_option('caption');

	function selectionGen(&amp;$option,&amp;$array) {
		foreach($array as $key=&gt; $ms) {
			$selected = ($option != $ms)? '' : ' selected';
			echo &quot;&lt;option value='$ms'$selected&gt;&quot;.(($ms=='1'&amp;&amp;$array[0]!='0')?__('Disabled', 'wp-slimbox2'):$ms).&quot;&lt;/option&gt;\n&quot;;
		}
	}
?&gt;
	&lt;div style=&quot;clear:both;padding-top:5px;&quot;&gt;&lt;/div&gt;
		&lt;h2&gt;&lt;?php _e('Settings', 'wp-slimbox2');?&gt;&lt;/h2&gt;
		&lt;table class=&quot;widefat&quot; cellspacing=&quot;0&quot; id=&quot;inactive-plugins-table&quot;&gt;
			&lt;thead&gt;
			&lt;tr&gt;
				&lt;th scope=&quot;col&quot; colspan=&quot;2&quot;&gt;&lt;?php _e('Setting', 'wp-slimbox2'); ?&gt;&lt;/th&gt;
				&lt;th scope=&quot;col&quot;&gt;&lt;?php _e('Description', 'wp-slimbox2'); ?&gt;&lt;/th&gt;
			&lt;/tr&gt;
			&lt;/thead&gt;

			&lt;tfoot&gt;
			&lt;tr&gt;
				&lt;th scope=&quot;col&quot; colspan=&quot;3&quot;&gt;&lt;?php _e('Use the various options above to control some of the advanced settings of the plugin', 'wp-slimbox2'); ?&gt;&lt;/th&gt;
			&lt;/tr&gt;
			&lt;/tfoot&gt;

			&lt;tbody class=&quot;plugins&quot;&gt;

			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Autoload?', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;input type=&quot;checkbox&quot; name=&quot;wp_slimbox_autoload&quot;&lt;?php if ($options-&gt;get_option('autoload') == 'on') echo ' checked=&quot;yes&quot;';?&gt; /&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to automatically activate Slimbox on all links pointing to &quot;.jpg&quot;, &quot;.jpeg&quot;, &quot;.png&quot;, &quot;.bmp&quot; or &quot;.gif&quot;. All image links will automatically be grouped together in a gallery according to the selector chosen below. If this isn\'t activated you will need to manually add &lt;b&gt;&lt;code&gt;rel=&quot;lightbox&quot;&lt;/code&gt;&lt;/b&gt; for individual images or &lt;b&gt;&lt;code&gt;rel=&quot;lightbox-imagesetname&quot;&lt;/code&gt;&lt;/b&gt; for groups on all links you wish to use the Slimbox effect. &lt;b&gt;Default is Disabled.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Enable Picasaweb Integration?', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;input type=&quot;checkbox&quot; name=&quot;wp_slimbox_picasaweb&quot;&lt;?php if ($options-&gt;get_option('picasaweb') == 'on') echo ' checked=&quot;yes&quot;';?&gt; /&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to automatically add the Slimbox effect to Picasaweb links when provided an appropriate url (this is separate from the autoload script which only functions on direct image links). &lt;b&gt;Default is Disabled.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Enable Flickr Integration?', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;input type=&quot;checkbox&quot; name=&quot;wp_slimbox_flickr&quot;&lt;?php if ($options-&gt;get_option('flickr') == 'on') echo ' checked=&quot;yes&quot;';?&gt; /&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to automatically add the Slimbox effect to Flickr links when provided an appropriate url (this is separate from the autoload script which only functions on direct image links). &lt;b&gt;Default is Disabled.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Loop?', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;input type=&quot;checkbox&quot; name=&quot;wp_slimbox_loop&quot;&lt;?php if ($options-&gt;get_option('loop') == 'on') echo ' checked=&quot;yes&quot;';?&gt; /&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to navigate between the first and last images of a Slimbox gallery group when there is more than one image to display. &lt;b&gt;Default is Disabled.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Overlay Opacity', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;select name=&quot;wp_slimbox_overlayOpacity&quot;&gt;
					&lt;?php selectionGen($options-&gt;get_option('overlayOpacity'),$overlayOpacity); ?&gt;
					&lt;/select&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to adjust the opacity of the background overlay. 1 is completely opaque, 0 is completely transparent. &lt;b&gt;Default is 0.8.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Overlay Color', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;input type=&quot;text&quot; id=&quot;wp_slimbox_overlayColor&quot; name=&quot;wp_slimbox_overlayColor&quot; value=&quot;&lt;?php echo $options-&gt;get_option('overlayColor'); ?&gt;&quot; size=&quot;7&quot; maxlength=&quot;7&quot;/&gt;&lt;div id=&quot;picker&quot;&gt;&lt;/div&gt;

				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to set the color of the overlay by selecting your hue from the circle and color gradient from the square. Alternatively you may manually enter a valid HTML color code. The color of the entry field will change to reflect your selected color. &lt;b&gt;Default is #000000.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Overlay Fade Duration', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;select name=&quot;wp_slimbox_overlayFadeDuration&quot;&gt;
					&lt;?php selectionGen($options-&gt;get_option('overlayFadeDuration'),$msArray); ?&gt;
					&lt;/select&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to adjust the duration of the overlay fade-in and fade-out animations, in milliseconds. &lt;b&gt;Default is 400.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
			&lt;tr class='inactive'&gt;
				&lt;td class='name'&gt;&lt;?php _e('Resize Duration', 'wp-slimbox2'); ?&gt;&lt;/td&gt;
				&lt;th scope='row' class='check-column'&gt;
					&lt;select name=&quot;wp_slimbox_resizeDuration&quot;&gt;
					&lt;?php selectionGen($options-&gt;get_option('resizeDuration'),$msArray); ?&gt;
					&lt;/select&gt;
				&lt;/th&gt;
				&lt;td class='desc'&gt;
					&lt;p&gt; &lt;?php _e('This option allows the user to adjust the duration of the resize animation for width and height, in milliseconds. &lt;b&gt;Default is 400.&lt;/b&gt;', 'wp-slimbox2'); ?&gt;
					&lt;/p&gt;
				&lt;/td&gt;
			&lt;/tr&gt;
[/sourcecode]
