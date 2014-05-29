---
layout: post
title: Three Days
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _edit_last: '2'
  _aktt_hash_meta: ''
  bfa_ata_body_title: Three Days
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Three Days
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 08fc77e0a7174347a6d45f421aa9a63f
  _sexybookmarks_shortUrl: http://bit.ly/cB7Rjb
---
<blockquote><strong>For three days after death, hair and fingernails continue to grow but phone calls taper off</strong><strong>.</strong><strong>
- Johnny Carson
</strong></blockquote>
```php
<?
$user_login = $HTTP_POST_VARS['user_login'];
$pass1 = $HTTP_POST_VARS['pass1'];
$pass2 = $HTTP_POST_VARS['pass2'];
$user_email = $HTTP_POST_VARS['user_email'];

/* checking login has been typed */
if ($user_login == '') {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: Please enter a login.');
}

/* checking the password has been typed twice */
if ($pass1 == '' || $pass2 == '') {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: Please enter your password twice.');
}

/* checking the password has been typed twice the same */
if ($pass1 != $pass2)    {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: Please type the same password in the two password fields.');
}
$user_nickname = $user_login;

/* checking e-mail address */
if ($user_email == '') {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: Please type your e-mail address.');
} else if (!is_email($user_email)) {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: The email address isn\'t correct.');
}

/* checking the login isn't already used by another user */
$result = $wpdb-&gt;get_results("SELECT user_login FROM $tableusers WHERE user_login = '$user_login'");
if (count($result) &gt;= 1) {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: This login is already registered, please choose another one.');
}

$user_ip = $HTTP_SERVER_VARS['REMOTE_ADDR'] ;
$user_domain = gethostbyaddr($HTTP_SERVER_VARS['REMOTE_ADDR'] );
$user_browser = $HTTP_SERVER_VARS['HTTP_USER_AGENT'];

$user_login = addslashes($user_login);
$pass1 = addslashes($pass1);
$user_nickname = addslashes($user_nickname);
$now = current_time('mysql');

$result = $wpdb-&gt;query("INSERT INTO $tableusers
(user_login, user_pass, user_nickname, user_email, user_ip, user_domain, user_browser, dateYMDhour, user_level, user_idmode)
VALUES
('$user_login', '$pass1', '$user_nickname', '$user_email', '$user_ip', '$user_domain', '$user_browser', '$now', '$new_users_can_blog', 'nickname')");

if ($result == false) {
die ('&lt;strong&gt;ERROR&lt;/strong&gt;: Couldn\'t register you... please contact the &lt;a href="mailto:'.$admin_email.'"&gt;webmaster&lt;/a&gt; !');
}

$stars = '';
for ($i = 0; $i &lt; strlen($pass1); $i = $i + 1) {
$stars .= '*';
}

$message  = "New user registration on your blog $blogname:\r\n\r\n";
$message .= "Login: $user_login\r\n\r\nE-mail: $user_email";

@mail($admin_email, "[$blogname] New User Registration", $message);

```
