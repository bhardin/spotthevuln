---
layout: post
title: Sacred Facts - XSS
tags:
- Cross-Site Scripting (XSS)
- Persistent
- PHP
- Solution
- strip_tags
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '2'
  _headspace_page_title: Cross-Site Scripting (XSS) Vulnerability Code Example
  _headspace_description: Cross-Site Scripting (XSS) Vulnerability Code Example
  _sexybookmarks_permaHash: 70fc6ce5036dece2715d76c3e0b15e65
  _sexybookmarks_shortUrl: http://bit.ly/8Ln0A7
---
## Details
<strong>__Affected Software:__ WordPress</strong>

<strong>__Fixed in Version:__  2.8</strong>

<strong>__Issue Type:__ Cross Site Scripting</strong>

<strong>Original Code: </strong><a href="http://spotthevuln.com/2009/11/vulnerable-code-sacred-facts/">Found Here</a>
## Description
The WordPress developers fixed a persistent Cross Site Scripting vulnerability with this code fix.  Examining the vulnerable code, we see that the $comment_post_title variable is assigned an un-sanitized value from get_the_title( $comment-&gt;comment_post_ID ).  $comment_post_title is then immediately used in to build HTML markup (used as the text for a HREF tag) and assigned to the $comment_post_link variable.  The $comment_post_link variable with the tainted value is eventually used in the HTML markup in a WordPress page.  By placing script into a blog post title, a contributor could use the persistent cross site scripting vulnerability to elevate to WordPress Administrator.

 

The WordPress team implemented the PHP strip_tags() function to strip HTML tags from the post title before assigning to the $comment_post_title variable.  More information related to the PHP strip_tags() API can be found <a title="PHP strip_tags" href="http://us2.php.net/manual/en/function.strip-tags.php" target="_blank">here</a>.  It should be noted that the documentation for strip_tags() provides the following warning:
<blockquote>Because <strong>strip_tags()</strong> does not actually validate the HTML, partial, or broken tags can result in the removal of more text/data than expected.</blockquote>
Interesting indeed…
<h2>Developers Solution</h2>
[cce lang="diff"]

function _wp_dashboard_recent_comments_row( &amp;$comment, $show_date = true ) {
$GLOBALS['comment'] =&amp; $comment;

$comment_post_url = get_edit_post_link( $comment-&gt;comment_post_ID );
-       $comment_post_title = get_the_title( $comment-&gt;comment_post_ID );
+       $comment_post_title = strip_tags(get_the_title( $comment-&gt;comment_post_ID ));
$comment_post_link = "&lt;a href='$comment_post_url'&gt;$comment_post_title&lt;/a&gt;";
$comment_link = '&lt;a href="' . get_comment_link() . '"&gt;#&lt;/a&gt;';

$delete_url = clean_url( wp_nonce_url( "comment.php?action=deletecomment&amp;p=$comment-&gt;comment_post_ID&amp;c=$comment-&gt;comment_ID", "delete-comment_$comment-&gt;comment_ID" ) );
$approve_url = clean_url( wp_nonce_url( "comment.php?action=approvecomment&amp;p=$comment-&gt;comment_post_ID&amp;c=$comment-&gt;comment_ID", "approve-comment_$comment-&gt;comment_ID" ) );
$unapprove_url = clean_url( wp_nonce_url( "comment.php?action=unapprovecomment&amp;p=$comment-&gt;comment_post_ID&amp;c=$comment-&gt;comment_ID", "unapprove-comment_$comment-&gt;comment_ID" ) );
$spam_url = clean_url( wp_nonce_url( "comment.php?action=deletecomment&amp;dt=spam&amp;p=$comment-&gt;comment_post_ID&amp;c=$comment-&gt;comment_ID", "delete-comment_$comment-&gt;comment_ID" ) );

$actions = array();

$actions_string = '';
if ( current_user_can('edit_post', $comment-&gt;comment_post_ID) ) {
$actions['approve'] = "&lt;a href='$approve_url' class='dim:the-comment-list:comment-$comment-&gt;comment_ID:unapproved:e7e7d3:e7e7d3:new=approved vim-a' title='" . __( 'Approve this comment' ) . "'&gt;" . __( 'Approve' ) . '&lt;/a&gt;';
$actions['unapprove'] = "&lt;a href='$unapprove_url' class='dim:the-comment-list:comment-$comment-&gt;comment_ID:unapproved:e7e7d3:e7e7d3:new=unapproved vim-u' title='" . __( 'Unapprove this comment' ) . "'&gt;" . __( 'Unapprove' ) . '&lt;/a&gt;';
$actions['edit'] = "&lt;a href='comment.php?action=editcomment&amp;amp;c={$comment-&gt;comment_ID}' title='" . __('Edit comment') . "'&gt;". __('Edit') . '&lt;/a&gt;';
//$actions['quickedit'] = '&lt;a onclick="commentReply.open(''.$comment-&gt;comment_ID.'',''.$comment-&gt;comment_post_ID.'','edit');return false;" title="'.__('Quick Edit').'" href="#"&gt;' . __('Quick&amp;nbsp;Edit') . '&lt;/a&gt;';
$actions['reply'] = '&lt;a onclick="commentReply.open(''.$comment-&gt;comment_ID.'',''.$comment-&gt;comment_post_ID.'');return false;" title="'.__('Reply to this comment').'" href="#"&gt;' . __('Reply') . '&lt;/a&gt;';
$actions['spam'] = "&lt;a href='$spam_url' class='delete:the-comment-list:comment-$comment-&gt;comment_ID::spam=1 vim-s vim-destructive' title='" . __( 'Mark this comment as spam' ) . "'&gt;" . /* translators: mark as spam link */  _x( 'Spam', 'verb' ) . '&lt;/a&gt;';
$actions['delete'] = "&lt;a href='$delete_url' class='delete:the-comment-list:comment-$comment-&gt;comment_ID delete vim-d vim-destructive'&gt;" . __('Delete') . '&lt;/a&gt;';

$actions = apply_filters( 'comment_row_actions', $actions, $comment );

$i = 0;
[/cce] 
