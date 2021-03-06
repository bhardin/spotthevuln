---
layout: post
title: Madman - File Include
tags:
- File Inclusion
- Solution
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: eb3ebc9cad7c08cee9639a1fb3fd2207
  _sexybookmarks_shortUrl: http://bit.ly/aaWiOL
  _aktt_hash_meta: ''
  aktt_notify_twitter: 'yes'
  _edit_last: '1'
  aktt_tweeted: '1'
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Madman - File Include
  bfa_ata_body_title: Madman - File Include
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  _headspace_page_title: File Include Vulnerability Code Example
---
## Details
<strong>__Affected Software:__ Joomla
</strong>

<strong>__Fixed in Version:__  Directory Revision 10041
</strong>

<strong>__Issue Type:__ File Include Vulnerability
</strong>

<strong>Original Code: </strong><a title="Madman" href="http://spotthevuln.com/2009/12/madman/" target="_blank">Found Here</a>
## Description
This particular vulnerability affected Joomla. The vulnerable code had symptoms which could allow for a file inclusion vulnerability (under certain circumstances). PHP based applications are especially vulnerable to remote file due to extensive use of file includes and common PHP server configurations.

File include vulnerabilities give the attacker the ability to execute arbitrary commands on the web server, resulting in the complete compromise of the Joomla installation. The PHP include(), require(), include_once() and require_once() functions are great candidates for remote file include attacks and in this case we see that the Joomla code base makes use of require_once() function.

Although the Joomla developers checked in a code change which changed the require_once() function to a require(), the real fix will be a configuration change for the PHP server (turning register_globals off). What's a bit surprising is the Joomla checked in a code change which was designed to prevent a file include vulnerability but changed the require_once() function to a require() function. Typically, file inclusion vulnerabilities are fixed by changing a require() function call to an require_once() function call and explicitly loading the required library before the attacker has a chance to influence the file inclusion. Once again, the authors of this blog are not responsible for the code fixes checked into the software branch :)

It is also interesting that we see a call to the include() function, which remained unchanged:
<blockquote>include ($mosConfig_absolute_path .'/offline.php');</blockquote>
## Developer's Solution
```diff

// Set flag that this is a parent file
define( '_VALID_MOS', 1 );

if (!file_exists( '../configuration.php' )) {
header( 'Location: ../installation/index.php' );
exit();
}

require( '../globals.php' );
-require_once( '../configuration.php' );
+require( '../configuration.php' );

// SSL check - $http_host returns &lt;live site url&gt;:&lt;port number if it is 443&gt;
$http_host = explode(':', $_SERVER['HTTP_HOST'] );
if( (!empty( $_SERVER['HTTPS'] ) &amp;&amp; strtolower( $_SERVER['HTTPS'] ) != 'off' || isset( $http_host[1] ) &amp;&amp; $http_host[1] == 443) &amp;&amp; substr( $mosConfig_live_site, 0, 8 ) != 'https://' ) {
$mosConfig_live_site = 'https://'.substr( $mosConfig_live_site, 7 );
}

require_once( '../includes/joomla.php' );
include_once ( $mosConfig_absolute_path . '/language/'. $mosConfig_lang .'.php' );

//Installation sub folder check, removed for work with SVN
if (file_exists( '../installation/index.php' ) &amp;&amp; $_VERSION-&gt;SVN == 0) {
define( '_INSTALL_CHECK', 1 );
include ($mosConfig_absolute_path .'/offline.php');
exit();
}

$option = strtolower( strval( mosGetParam( $_REQUEST, 'option', NULL ) ) );

```
