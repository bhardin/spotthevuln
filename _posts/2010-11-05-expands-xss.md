---
layout: post
title: Expands - Cross-Site Scripting
tags:
- All
- attribute
- Boinc
- Cross-Site Scripting (XSS)
- disclosure
- Encode
- htmlspecialchars
- open source project
- PHP
- request body
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _aktt_hash_meta: ! '#secure #code #dev'
  _syntaxhighlighter_encoded: '1'
  _sexybookmarks_shortUrl: http://bit.ly/dejILy
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 862d5ada00665af351dea81e08421282
---
## Details
__Affected Software:__ Boinc

__Fixed in Version:__  N/A

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Expect" href="http://spotthevuln.com/2010/11/expands/" target="_blank">Found    Here</a>
## Description
The hint pretty much gave this one away.

This week's bug was an XSS exposure in Boinc. Boinc is an open source project which uses idle time on a computer to perform various scientific tasks. According to the website, its "safe, secure, and easy". Boinc doesn't seem to have version numbers and is updated on a constant basis. This bug was checked into the Boinc source about four weeks ago.

The bug can be found in the else statement in the pm_form() function. The else statement takes a tainted value from the POST request body and uses it to populate a variable. The variable assignment is here:
<blockquote>$writeto = post_str("to", true);
$subject = post_str("subject", true);
$content = post_str("content", true);</blockquote>
The hint lets us know that post_str takes values from the POST request body. Even if a hint wasn't provided, you could have guessed that values coming from post_str() were tainted by looking at the first two lines of code outside of the if/else statement:
<blockquote>$content = htmlspecialchars($content);
$subject = htmlspecialchars($subject);</blockquote>
The two lines above show that the author was careful to encode the $subject and $content variables before using them in markup. Although the $writeto variable was assigned in a similar manner and even in the same code block, it doesn't undergo any encoding. $writeto is then used in the markup in the code provided below, resulting in XSS:
<blockquote>"&lt;input type=\"text\" name=\"to\" value=\"$writeto\" size=\"60\"&gt;"</blockquote>
Some other interesting items. This page is using the .inc extension for includes. Hopefully, the correct extension mappings are done to avoid source code disclosure of includes. While the application logic is open source, I'm sure there are configuration settings like database connection strings which could be exposed. Also, instead of using htmlspecialchars() to encode $writeto, the developers chose to use a function named sanitize_tags(). I hope sanitize tags defends against attribute injection which doesn't require any HTML tags!
## Developers Solution
[sourcecode language="diff"]
&lt;?php

require_once(&quot;boinc_db.inc&quot;);
require_once(&quot;sanitize_html.inc&quot;);
require_once(&quot;bbcode_html.inc&quot;);

function pm_header() {
    echo &quot;&lt;div&gt;\n&quot;;
    echo &quot;    &lt;a href=\&quot;pm.php?action=inbox\&quot;&gt;&quot;.tra(&quot;Inbox&quot;).&quot;&lt;/a&gt;\n&quot;;
    echo &quot;    | &lt;a href=\&quot;pm.php?action=new\&quot;&gt;&quot;.tra(&quot;Write&quot;).&quot;&lt;/a&gt;\n&quot;;
    echo &quot;&lt;/div&gt;\n&quot;;
}

function pm_form($error = null) {
    global $bbcode_html, $bbcode_js;
    global $g_logged_in_user;
    page_head(tra(&quot;Send private message&quot;),'','','', $bbcode_js);

    if (post_str(&quot;preview&quot;, true) == tra(&quot;Preview&quot;)) {
        $options = new output_options;
        echo &quot;&lt;div id=\&quot;preview\&quot;&gt;\n&quot;;
        echo &quot;&lt;div class=\&quot;header\&quot;&gt;&quot;.tra(&quot;Preview&quot;).&quot;&lt;/div&gt;\n&quot;;
        echo output_transform(post_str(&quot;content&quot;, true), $options);
        echo &quot;&lt;/div&gt;\n&quot;;
    }

    $replyto = get_int(&quot;replyto&quot;, true);
    $userid = get_int(&quot;userid&quot;, true);

    $subject = null;
    $content = null;
    if ($replyto) {
        $message = BoincPrivateMessage::lookup_id($replyto);
        if (!$message || $message-&gt;userid != $g_logged_in_user-&gt;id) {
            error_page(&quot;no such message&quot;);
        }
        $content = &quot;[quote]&quot;.$message-&gt;content.&quot;[/quote]\n&quot;;
        $userid = $message-&gt;senderid;
        $user = BoincUser::lookup_id($userid);
        if ($user != null) {
            $writeto = $userid.&quot; (&quot;.$user-&gt;name.&quot;)&quot;;
        }
        $subject = $message-&gt;subject;
        if (substr($subject, 0, 3) != &quot;re:&quot;) {
            $subject = &quot;re: &quot;.$subject;
        }
    } elseif ($userid) {
        $user = BoincUser::lookup_id($userid);
        if ($user) {
            $writeto = $userid.&quot; (&quot;.$user-&gt;name.&quot;)&quot;;
        }
    } else {
-       $writeto = post_str(&quot;to&quot;, true);
+		$writeto = sanitize_tags(post_str(&quot;to&quot;, true));
        $subject = post_str(&quot;subject&quot;, true);
        $content = post_str(&quot;content&quot;, true);
    }

    $content = htmlspecialchars($content);
    $subject = htmlspecialchars($subject);

    if ($error != null) {
        echo &quot;&lt;div class=\&quot;error\&quot;&gt;&quot;.$error.&quot;&lt;/div&gt;\n&quot;;
    }

    echo &quot;&lt;form action=\&quot;pm.php\&quot; method=\&quot;post\&quot; name=\&quot;post\&quot; onsubmit=\&quot;return checkForm(this)\&quot;&gt;\n&quot;;
    echo &quot;&lt;input type=\&quot;hidden\&quot; name=\&quot;action\&quot; value=\&quot;send\&quot;&gt;\n&quot;;
    echo form_tokens($g_logged_in_user-&gt;authenticator);
    start_table();
    row2(tra(&quot;To&quot;).&quot;&lt;br /&gt;&lt;span class=\&quot;smalltext\&quot;&gt;&quot;.tra(&quot;User IDs or unique usernames, separated with commas&quot;).&quot;&lt;/span&gt;&quot;,
        &quot;&lt;input type=\&quot;text\&quot; name=\&quot;to\&quot; value=\&quot;$writeto\&quot; size=\&quot;60\&quot;&gt;&quot;
    );
    row2(tra(&quot;Subject&quot;), &quot;&lt;input type=\&quot;text\&quot; name=\&quot;subject\&quot; value=\&quot;$subject\&quot; size=\&quot;60\&quot;&gt;&quot;);
    row2(tra(&quot;Message&quot;).&quot;&lt;span class=\&quot;smalltext\&quot;&gt;&quot;.html_info().&quot;&lt;/span&gt;&quot;,
        $bbcode_html.&quot;&lt;textarea name=\&quot;content\&quot; rows=\&quot;18\&quot; cols=\&quot;80\&quot;&gt;$content&lt;/textarea&gt;&quot;
    );
    echo &quot;&lt;tr&gt;&lt;td&gt;&lt;/td&gt;&lt;td&gt;&lt;input type=\&quot;submit\&quot; name=\&quot;preview\&quot; value=\&quot;&quot;.tra(&quot;Preview&quot;).&quot;\&quot;&gt; &lt;input type=\&quot;submit\&quot; value=\&quot;&quot;.tra(&quot;Send message&quot;).&quot;\&quot;&gt;&lt;/td&gt;&lt;/tr&gt;\n&quot;;
    end_table();

    page_tail();
    exit();
}

function send_pm_notification_email(
    $logged_in_user, $to_user, $subject, $content
) {
    $message  = &quot;
You have received a new private message at &quot;.PROJECT.&quot;.

From: $logged_in_user-&gt;name (ID $logged_in_user-&gt;id)
Subject: $subject

$content

--------------------------
To delete or respond to this message, visit:
&quot;.URL_BASE.&quot;pm.php

To change email preferences, visit:
&quot;.URL_BASE.&quot;edit_forum_preferences_form.php
Do not reply to this message.
&quot; ;
    send_email($to_user, &quot;[&quot;.PROJECT.&quot;] &quot;.tra(&quot;- private message&quot;), $message);
}

?&gt;

[/sourcecode]
