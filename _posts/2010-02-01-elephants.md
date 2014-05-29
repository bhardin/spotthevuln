---
layout: post
title: Elephants
tags:
- Code Snippet
- PHP
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  bfa_ata_body_title: Elephants
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Elephants
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  _sexybookmarks_shortUrl: http://bit.ly/boAzYB
  _sexybookmarks_permaHash: d727f45bd203472e49741dc7fa3b8176
---
<blockquote><strong>Yesterday I shot an elephant in my pajamas. How he got in my pajamas I don't know</strong><strong>.
- Groucho Marx</strong></blockquote>
```php
<?
/**
* Evaluate the strength of a user's password.
*
* Returns the estimated strength and the relevant output message.
*/

Drupal.evaluatePasswordStrength = function (password, translate) {

var weaknesses = 0, strength = 100, msg = [];
var hasLowercase = password.match(/[a-z]+/);
var hasUppercase = password.match(/[A-Z]+/);
var hasNumbers = password.match(/[0-9]+/);
var hasPunctuation = password.match(/[^a-zA-Z0-9]+/);

// If there is a username edit box on the page, compare password to that, otherwise
// use value from the database.

var usernameBox = $('input.username');
var username = (usernameBox.length &gt; 0) ? usernameBox.val() : translate.username;

// Lose 10 points for every character less than 6.

if (password.length &lt; 6) {
msg.push(translate.tooShort);
strength -= (6 - password.length) * 10;
}

// Count weaknesses.
if (!hasLowercase) {
msg.push(translate.addLowerCase);
weaknesses++;
}

if (!hasUppercase) {
msg.push(translate.addUpperCase);
weaknesses++;
}
if (!hasNumbers) {
msg.push(translate.addNumbers);
weaknesses++;
}

if (!hasPunctuation) {
msg.push(translate.addPunctuation);
weaknesses++;
}

...

// Based on the strength, work out what text should be shown by the password strength meter.
if (strength &lt; 60) {
indicatorText = translate.weak;
} else if (strength &lt; 70) {
indicatorText = translate.fair;
} else if (strength &lt; 80) {
indicatorText = translate.good;
} else if (strength &lt; 100) {
indicatorText = translate.strong;
}

// Assemble the final message.
msg = translate.hasWeaknesses + '&lt;ul&gt;&lt;li&gt;' + msg.join('&lt;/li&gt;&lt;li&gt;') + '&lt;/li&gt;&lt;/ul&gt;';
return { strength: strength, message: msg, indicatorText: indicatorText }

```
