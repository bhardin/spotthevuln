---
layout: post
title: Expands - Cross-Site Scripting
tags:
- Cross-Site Scripting (XSS)
- PHP
- query string
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
---
## Details
__Affected Software:__ Wordpress Core

__Fixed in Version:__  2.8

__Issue Type:__ Cross-Site Scripting

Original Code: <a href="http://spotthevuln.com/2011/05/expands-2/">Found Here</a>
## Details
This week's bug was subtle. The patch submitted by the developer addresses an XSS bug. Looking at the diff, we see that $title and $selection come from the query string. These values are fixed up before being assigned to a variable. The developers changed the way $title is assigned in the diff. It's difficult to see why $title needs to be changed, so we'll ignore that change for now. $selection gives some hints towards XSS. $selection is assigned from the query string value $_GET['s'] which is sent through the trim() and aposfix() functions. Immediately following the $selection variable assignment we see the $selection being manipulated with some HTML tags. This is a good indication that $selection will eventually be used to build HTML markup. I usually recommend developers encode/sanitize values as close to the point of consumption as possible, but this case is different. $selection contains HTML tags (the &lt;p&gt; and &lt;/p&gt; tags), so encoding the whole value at the point of consumption will also encode those tags. The &lt;p&gt; tags are probably designed to be rendered by the browser. Due to the mixing of HTML tags and regular data, the developer will have to encode the user controlled values during variable assignment. This is exactly what the Wordpress developers did in the patch.


## Developer's Solution
[sourcecode language="diff" highlight="68,69,70,71"]
&lt;?php

require_once('admin.php');
header('Content-Type: ' . get_option('html_type') . '; charset=' . get_option('blog_charset'));

if ( ! current_user_can('edit_posts') )
	wp_die( __( 'Cheatin&amp;#8217; uh?' ) );

function aposfix($text) {
	$translation_table[chr(34)] = '&amp;quot;';
	$translation_table[chr(38)] = '&amp;';
	$translation_table[chr(39)] = '&amp;apos;';
	return preg_replace(&quot;/&amp;(?![A-Za-z]{0,4}\w{2,3};|#[0-9]{2,3};)/&quot;,&quot;&amp;amp;&quot; , strtr($text, $translation_table));
}


function press_it() {
	// define some basic variables
	$quick['post_status'] = 'draft'; // set as draft first
	$quick['post_category'] = isset($_REQUEST['post_category']) ? $_REQUEST['post_category'] : null;
	$quick['tax_input'] = isset($_REQUEST['tax_input']) ? $_REQUEST['tax_input'] : '';
	$quick['post_title'] = isset($_REQUEST['title']) ? $_REQUEST['title'] : '';
	$quick['post_content'] = '';

	// insert the post with nothing in it, to get an ID
	$post_ID = wp_insert_post($quick, true);
	$content = isset($_REQUEST['content']) ? $_REQUEST['content'] : '';

	$upload = false;
	if( !empty($_REQUEST['photo_src']) &amp;&amp; current_user_can('upload_files') )
		foreach( (array) $_REQUEST['photo_src'] as $key =&gt; $image)
			// see if files exist in content - we don't want to upload non-used selected files.
			if( strpos($_REQUEST['content'], $image) !== false ) {
				$desc = isset($_REQUEST['photo_description'][$key]) ? $_REQUEST['photo_description'][$key] : '';
				$upload = media_sideload_image($image, $post_ID, $desc);

				// Replace the POSTED content &lt;img&gt; with correct uploaded ones. Regex contains fix for Magic Quotes
				if( !is_wp_error($upload) ) $content = preg_replace('/&lt;img ([^&gt;]*)src=\\\?(\&quot;|\')'.preg_quote($image, '/').'\\\?(\2)([^&gt;\/]*)\/*&gt;/is', $upload, $content);
			}

	// set the post_content and status
	$quick['post_status'] = isset($_REQUEST['publish']) ? 'publish' : 'draft';
	$quick['post_content'] = $content;
	// error handling for $post
	if ( is_wp_error($post_ID)) {
		wp_die($id);
		wp_delete_post($post_ID);
	// error handling for media_sideload
	} elseif ( is_wp_error($upload)) {
		wp_die($upload);
		wp_delete_post($post_ID);
	} else {
		$quick['ID'] = $post_ID;
		wp_update_post($quick);
	}
	return $post_ID;
}

// For submitted posts.
if ( isset($_REQUEST['action']) &amp;&amp; 'post' == $_REQUEST['action'] ) {
	check_admin_referer('press-this');
	$post_ID = press_it();
	$posted =  $post_ID;
} else {
	$post_ID = 0;
}

// Set Variables
-$title = isset($_GET['t']) ? esc_html(aposfix(stripslashes($_GET['t']))) : '';
-$selection = isset($_GET['s']) ? trim( aposfix( stripslashes($_GET['s']) ) ) : '';
+$title = isset( $_GET['t'] ) ? trim( strip_tags( aposfix( stripslashes( $_GET['t'] ) ) ) ) : '';
+$selection = isset( $_GET['s'] ) ? trim( htmlspecialchars( html_entity_decode( aposfix( stripslashes( $_GET['s'] ) ) ) ) ) : '';
if ( ! empty($selection) ) {
	$selection = preg_replace('/(\r?\n|\r)/', '&lt;/p&gt;&lt;p&gt;', $selection);
	$selection = '&lt;p&gt;'.str_replace('&lt;p&gt;&lt;/p&gt;', '', $selection).'&lt;/p&gt;';
}
$url = isset($_GET['u']) ? esc_url($_GET['u']) : '';
$image = isset($_GET['i']) ? $_GET['i'] : '';
...snip...
[/sourcecode]
