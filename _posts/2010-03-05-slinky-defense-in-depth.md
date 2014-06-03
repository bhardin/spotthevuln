---
layout: post
title: Slinky - Defense in Depth
tags:
- Apache
- Code Snippet
- Defense In Depth
- Solution
status: publish
type: post
published: true
meta:
  bfa_ata_body_title_multi: Slinky - Defense in Depth
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  aktt_tweeted: '1'
  bfa_ata_display_body_title: ''
  bfa_ata_body_title: Slinky - Defense in Depth
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  _oembed_d12d3b9db1c0cf62a2e977d9ddb3b2d2: ! '{{unknown}}'
  _sexybookmarks_permaHash: 2b87e4c43b0c995b9d55e37d0c0a1952
  _sexybookmarks_shortUrl: http://bit.ly/axHfoi
  _oembed_c9d399e131a26ef505befbaf7d26fc14: ! '{{unknown}}'
---
## Details
__Affected Software:__ Apache webservices

__Fixed in Version:__  revision 697031

__Issue Type:__ Defense in Depth

Original Code: <a title="Slinky" href="http://spotthevuln.com/2010/03/slinky/" target="_blank">Found Here</a>
## Description
<div>

This is a defense in depth fix checked into the Apache SVN under web services. This particular issue prevented a username enumeration/disclosure bug that would occur if a null password was sent with a value username. The developers fixed this particular issue by logging the failure in the debug log and displaying a generic authentication failure message back to the user.
<blockquote>throw new WSSecurityException(WSSecurityException.FAILURE,
"noPassword", <span style="color: #ff0000;">new Object[]{user}</span>)</blockquote>
The defense in depth fix is pretty straightforward... what's interesting however, is the code surrounding the fix. A quick examination shows that there is a real possibility that clear text passwords are being logged in the debug log file. Take the following code for example:
<blockquote>log.debug("UsernameToken callback password " + <span style="color: #ff0000;">origPassword</span>);</blockquote>
I hope origPassword doesn't actually represent a user's password!

</div>
## Developer's Solution
```diff

http://svn.apache.org/viewvc/webservices/wss4j/trunk/src/org/apache/ws/security/processor/UsernameTokenProcessor.java?p2=/webservices/wss4j/trunk/src/org/apache/ws/security/processor/UsernameTokenProcessor.java&amp;p1=/webservices/wss4j/trunk/src/org/apache/ws/security/processor/UsernameTokenProcessor.java&amp;r1=697031&amp;r2=697030&amp;pathrev=697031&amp;view=diff&amp;diff_format=l

People are like slinkys, they get on your nerves, but its still fun to push them down some stairs.
By Anonymous

public WSUsernameTokenPrincipal handleUsernameToken(Element token, CallbackHandler cb)
throws WSSecurityException {
ut = new UsernameToken(token);
String user = ut.getName();
String password = ut.getPassword();
String nonce = ut.getNonce();
String createdTime = ut.getCreated();
String pwType = ut.getPasswordType();
if (log.isDebugEnabled()) {
log.debug("UsernameToken user " + user);
log.debug("UsernameToken password " + password);
}

Callback[] callbacks = new Callback[1];
String origPassword = null;

//
// If the UsernameToken is hashed, then retrieve the password from the callback handler
// and compare directly. If the UsernameToken is in plaintext or of some unknown type,
// then delegate authentication to the callback handler
//
if (ut.isHashed()) {
if (cb == null) {
throw new WSSecurityException(WSSecurityException.FAILURE, "noCallback");
}

WSPasswordCallback pwCb = new WSPasswordCallback(user, WSPasswordCallback.USERNAME_TOKEN);
callbacks[0] = pwCb;
try {
cb.handle(callbacks);
} catch (IOException e) {
if (log.isDebugEnabled()) {
log.debug(e);
}
throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);
} catch (UnsupportedCallbackException e) {
if (log.isDebugEnabled()) {
log.debug(e);
}
throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);
}
origPassword = pwCb.getPassword();
if (log.isDebugEnabled()) {
log.debug("UsernameToken callback password " + origPassword);
}
if (origPassword == null) {
+                if (log.isDebugEnabled()) {
+                    log.debug("Callback supplied no password for: " + user);
+                }
+                throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);

-                throw new WSSecurityException(WSSecurityException.FAILURE,
-                        "noPassword", new Object[]{user});
}
String passDigest = UsernameToken.doPasswordDigest(nonce, createdTime, origPassword);
if (!passDigest.equals(password)) {
throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);
}
ut.setRawPassword(origPassword);
} else {
if (cb == null) {
throw new WSSecurityException(WSSecurityException.FAILURE, "noCallback");
} else if (!WSConstants.PASSWORD_TEXT.equals(pwType) &amp;&amp; !handleCustomPasswordTypes) {
if (log.isDebugEnabled()) {
log.debug("Authentication failed as handleCustomUsernameTokenTypes is false");
}
throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);
}
WSPasswordCallback pwCb = new WSPasswordCallback(user, password,
pwType, WSPasswordCallback.USERNAME_TOKEN_UNKNOWN);
callbacks[0] = pwCb;
try {
cb.handle(callbacks);
} catch (IOException e) {
if (log.isDebugEnabled()) {
log.debug(e);
}
throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);
} catch (UnsupportedCallbackException e) {
if (log.isDebugEnabled()) {
log.debug(e);
}
throw new WSSecurityException(WSSecurityException.FAILED_AUTHENTICATION);
}
ut.setRawPassword(password);
}
WSUsernameTokenPrincipal principal = new WSUsernameTokenPrincipal(user, ut.isHashed());
principal.setNonce(nonce);
principal.setPassword(password);
principal.setCreatedTime(createdTime);
principal.setPasswordType(pwType);

return principal;
}

```
