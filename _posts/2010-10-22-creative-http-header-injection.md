---
layout: post
title: Creative - HTTP Header Injection
tags:
- addslashes
- All
- attacker
- bugs
- Carriage Return/Line Feed (CRLF) Injection
- Encode
- HTTP header injection
- PHP
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/9duePL
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_permaHash: 91d82e68df7d2d840999922d092d8b6a
---
## Details
__Affected Software:__ WordPress

__Fixed in Version:__  2.1

__Issue Type:__ HTTP Header Injection

Original Code: <a title="Creative" href="http://spotthevuln.com/2010/10/creative/" target="_blank">Found    Here</a>
## Description
This week's bug was a Set-Cookie/HTTP Header injection bug in WordPress Core. Looking at the code, we see that the WordPress Developers assign several variables directly from the $_POST body. These assignments are listed below:
<blockquote>$comment_author       = trim($_POST['author']);

$comment_author_email = trim($_POST['email']);

$comment_author_url   = trim($_POST['url']);

$comment_content      = trim($_POST['comment']);</blockquote>
Each of these variables should now be considered tainted and should be sanitized/encoded before using their values in any operations. It seems that the WordPress developers were aware that the values assigned to the variables listed above could not be trusted and proceeded to escape the values before using them in database related operations. Some examples of the sanitization can are provided below:
<blockquote>$comment_author       = $wpdb-&gt;escape($user_identity);

$comment_author_email = $wpdb-&gt;escape($user_email);

$comment_author_url   = $wpdb-&gt;escape($user_url);</blockquote>
Unfortunately, the developers utilized the incorrect sanitizing routines for the tainted values in the setcookie API. The developers used stripslashes() to sanitize the user controlled value before passing it to the setcookie() API. Although stripslashes can help prevent other types of vulnerabilities, it was not intended to defend against HTTP HEADER injection or cookie poisoning. By injecting a series of CRLF (%0d%0a) characters, an attacker could use this bug to inject arbitrary headers into the HTTP response and in some cases use this header injection bug for XSS. The WordPress developers addressed this issue by passing the attacker controlled value to the clean_url() function before allowing it in the setcookie. The full source for clean_url() can be found <a title="clean_url" href="http://core.trac.wordpress.org/browser/tags/2.9/wp-includes/formatting.php" target="_blank">here</a>, however the more interesting sanitizing routines are provided below:
<blockquote>$url = preg_replace('|[^a-z0-9-~+_.?#=!&amp;;,/:%@$\|*\'()\\x80-\\xff]|i', '', $url);

$strip = array('%0d', '%0a', '%0D', '%0A');

$url = _deep_replace($strip, $url);

$url = str_replace(';//', '://', $url);</blockquote>
## Developer's Solution<pre>
[sourcecode language="diff"]
&lt;?php
require( dirname(__FILE__) . '/wp-config.php' );

nocache_headers();

$comment_post_ID = (int) $_POST['comment_post_ID'];

$status = $wpdb-&gt;get_row(&quot;SELECT post_status, comment_status FROM $wpdb-&gt;posts WHERE ID = '$comment_post_ID'&quot;);

if ( empty($status-&gt;comment_status) ) {
	do_action('comment_id_not_found', $comment_post_ID);
	exit;
} elseif ( 'closed' ==  $status-&gt;comment_status ) {
	do_action('comment_closed', $comment_post_ID);
	die( __('Sorry, comments are closed for this item.') );
} elseif ( 'draft' == $status-&gt;post_status ) {
	do_action('comment_on_draft', $comment_post_ID);
	exit;
}

$comment_author       = trim($_POST['author']);
$comment_author_email = trim($_POST['email']);
$comment_author_url   = trim($_POST['url']);
$comment_content      = trim($_POST['comment']);

// If the user is logged in
get_currentuserinfo();
if ( $user_ID ) :
	$comment_author       = $wpdb-&gt;escape($user_identity);
	$comment_author_email = $wpdb-&gt;escape($user_email);
	$comment_author_url   = $wpdb-&gt;escape($user_url);
else :
	if ( get_option('comment_registration') )
		die( __('Sorry, you must be logged in to post a comment.') );
endif;

$comment_type = '';

if ( get_settings('require_name_email') &amp;&amp; !$user_ID ) {
	if ( 6 &gt; strlen($comment_author_email) || '' == $comment_author )
		die( __('Error: please fill the required fields (name, email).') );
	elseif ( !is_email($comment_author_email))
		die( __('Error: please enter a valid email address.') );
}

if ( '' == $comment_content )
	die( __('Error: please type a comment.') );

$commentdata = compact('comment_post_ID', 'comment_author', 'comment_author_email', 'comment_author_url', 'comment_content', 'comment_type', 'user_ID');

wp_new_comment( $commentdata );

if ( !$user_ID ) :
	setcookie('comment_author_' . COOKIEHASH, stripslashes($comment_author), time() + 30000000, COOKIEPATH, COOKIE_DOMAIN);
	setcookie('comment_author_email_' . COOKIEHASH, stripslashes($comment_author_email), time() + 30000000, COOKIEPATH, COOKIE_DOMAIN);
-	setcookie('comment_author_url_' . COOKIEHASH, stripslashes($comment_author_url), time() + 30000000, COOKIEPATH, COOKIE_DOMAIN);
+	setcookie('comment_author_url_' . COOKIEHASH, stripslashes(clean_url($comment_author_url)), time() + 30000000, COOKIEPATH, COOKIE_DOMAIN);
endif;

$location = ( empty( $_POST['redirect_to'] ) ) ? get_permalink( $comment_post_ID ) : $_POST['redirect_to'];

wp_redirect( $location );

?&gt;
[/sourcecode]</pre>
