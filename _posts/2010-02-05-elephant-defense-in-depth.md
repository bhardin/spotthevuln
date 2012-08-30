---
layout: post
title: Elephant - Defense in Depth
tags:
- Defense In Depth
- Drupal
- Solution
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: bc1c77fc59c958d92c63d66eb68305d3
  _sexybookmarks_shortUrl: http://bit.ly/dxhldL
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Drupal

__Fixed in Version:__  1.2.0

__Issue Type:__ Defense in Depth

Original Code: <a title="Elephant" href="http://spotthevuln.com/2010/02/elephants/" target="_blank">Found Here</a>
## Description
This was a simple change to a security message shown to a user.  Although not a critical security fix, this can help users make better decisions with regards to choosing an appropriate password.  The code sample provided below shows that Durpal checks for lowercase, uppercase, numbers, and punctuation in the user's password.
<blockquote>var weaknesses = 0, strength = 100, msg = [];

var hasLowercase = password.match(/[a-z]+/);

var hasUppercase = password.match(/[A-Z]+/);

var hasNumbers = password.match(/[0-9]+/);

var hasPunctuation = password.match(/[^a-zA-Z0-9]+/);</blockquote>
In addition to the checks above, the passwords length is also checked.  Durpal assigns the password a "score" of 100 and subtracts from this score if weaknesses are detected.  In this case, the Durpal decided that if the password was less than six characters, they would subtract 10 points from the password strength for every character short of 6.
<blockquote>strength -= (6 - password.length) * 10;</blockquote>
Later, the password is put through a "strength scale" where every password that has a score above "80" gets a rating of "strong".
<blockquote>

} else if (strength &lt; 80) {

indicatorText = translate.good;

} else if (strength &lt; 100) {

indicatorText = translate.strong;</blockquote>
Using this logic, it was possible for a user to select a 4 character password that was rated as strong.  To combat this, the Durpal developers penalized the user 30 points if they selected a password with less than 6 characters, guaranteeing that a password with less than 6 characters would never be rated "strong".

The safer way to go about this might be to assign a password strength of zero and build up the strength value as conditions are met.
<h2>Developers Solution</h2>
[cce lang="diff"]

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

-// Lose 10 points for every character less than 6.
+// Lose 5 points for every character less than 6, plus a 30 point penalty.
if (password.length &lt; 6) {
msg.push(translate.tooShort);
-  strength -= (6 - password.length) * 10;
+  strength -= ((6 - password.length) * 5) + 30;
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
-} else if (strength &lt; 100) {
+} else if (strength &lt;= 100) {
indicatorText = translate.strong;
}

// Assemble the final message.
msg = translate.hasWeaknesses + '&lt;ul&gt;&lt;li&gt;' + msg.join('&lt;/li&gt;&lt;li&gt;') + '&lt;/li&gt;&lt;/ul&gt;';
return { strength: strength, message: msg, indicatorText: indicatorText }

[/cce] 
