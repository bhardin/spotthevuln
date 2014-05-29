---
layout: post
title: Working Clothes
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 0edf3c5a47968f52445d5d7950db59b9
  _sexybookmarks_shortUrl: http://bit.ly/dD7IvS
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#code #development'
  _edit_last: '2'
  bfa_ata_body_title: Working Clothes
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Working Clothes
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
<blockquote><strong>Common sense is genius dressed in its working clothes.</strong>

<strong>- Ralph Waldo Emerson</strong></blockquote>
```php
<?php

function wp_dashboard_recent_drafts( $drafts = false ) {
        if ( !$drafts ) {
                $drafts_query = new WP_Query( array(
                        'post_type' =&gt; 'post',
                        'post_status' =&gt; 'draft',
                        'author' =&gt; $GLOBALS['current_user']-&gt;ID,
                        'posts_per_page' =&gt; 5,
                        'orderby' =&gt; 'modified',
                        'order' =&gt; 'DESC'
                ) );
                $drafts =&amp; $drafts_query-&gt;posts;
        }

        if ( $drafts &amp;&amp; is_array( $drafts ) ) {
                $list = array();
                foreach ( $drafts as $draft ) {
                        $url = get_edit_post_link( $draft-&gt;ID );
                        $title = _draft_or_post_title( $draft-&gt;ID );
                        $item = "&lt;h4&gt;&lt;a href='$url' title='" . sprintf( __( 'Edit &amp;#8220;%s&amp;#8221;' ), esc_attr( $title ) ) . "'&gt;$title&lt;/a&gt; &lt;abbr title='" . get_the_time(__('Y/m/d g:i:s A'), $draft) . "'&gt;" . get_the_time( get_option( 'date_format' ), $draft ) . '&lt;/abbr&gt;&lt;/h4&gt;';
                        if ( $the_content = preg_split( '#\s#', strip_tags( $draft-&gt;post_content ), 11, PREG_SPLIT_NO_EMPTY ) )
                                $item .= '&lt;p&gt;' . join( ' ', array_slice( $the_content, 0, 10 ) ) . ( 10 &lt; count( $the_content ) ? '&amp;hellip;' : '' ) . '&lt;/p&gt;';
                        $list[] = $item;
                }
?&gt;
        &lt;ul&gt;
                &lt;li&gt;&lt;?php echo join( "&lt;/li&gt;\n&lt;li&gt;", $list ); ?&gt;&lt;/li&gt;
        &lt;/ul&gt;
        &lt;p&gt;&lt;a href="edit.php?post_status=draft"&gt;&lt;?php _e('View all'); ?&gt;&lt;/a&gt;&lt;/p&gt;
&lt;?php
        } else {
                _e('There are no drafts at the moment');
        }
}

/**
 * Display recent comments dashboard widget content.
 *
 * @since unknown
 */
function wp_dashboard_recent_comments() {
        global $wpdb;

        if ( current_user_can('edit_posts') )
                $allowed_states = array('0', '1');
        else
                $allowed_states = array('1');

        // Select all comment types and filter out spam later for better query performance.
        $comments = array();
        $start = 0;

        while ( count( $comments ) &lt; 5 &amp;&amp; $possible = $wpdb-&gt;get_results( "SELECT * FROM $wpdb-&gt;comments c LEFT JOIN $wpdb-&gt;posts p ON c.comment_post_ID = p.ID WHERE p.post_status != 'trash' ORDER BY c.comment_date_gmt DESC LIMIT $start, 50" ) ) {

                foreach ( $possible as $comment ) {
                        if ( count( $comments ) &gt;= 5 )
                                break;
                        if ( in_array( $comment-&gt;comment_approved, $allowed_states ) )
                                $comments[] = $comment;
                }

                $start = $start + 50;
        }

        if ( $comments ) :
?&gt;
```
