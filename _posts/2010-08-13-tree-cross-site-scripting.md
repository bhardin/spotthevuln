---
layout: post
title: Tree - Cross Site Scripting
tags:
- All Vulnerabilities
- cross site scripting
- Cross-Site Scripting (XSS)
- Encode
- html markup
- PHP
- Solution
- Wordpress
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 8f13ad95fdc794da4fdc0484a6b685cd
  aktt_tweeted: '1'
  _wp_old_slug: ''
  _sexybookmarks_shortUrl: http://bit.ly/b1GIHi
  aktt_notify_twitter: 'yes'
  _edit_last: '1'
  _aktt_hash_meta: ''
---
## Details
__Affected Software:__ Wordpress-to-lead for Salesforce CRM

__Fixed in Version:__  1.0.5

__Issue Type:__ Cross Site Scripting

Original Code: <a title="Tree" href="http://spotthevuln.com/2010/08/tree/" target="_blank">Found    Here</a>
## Description
This week's vulnerability is an XSS bug in a Salesforce plugin for Wordpress. This bug is a bit interesting as it seems the developer attempted to sanitize an attacker controlled variable, but used the incorrect API. Looking at the vulnerable code, we see the following line:
<blockquote>return '&lt;label for="'.$id.'"&gt;'.$label.':&lt;/label&gt;&lt;br/&gt;&lt;input size="45" name="'.$id.'" value="'.stripslashes($options[$id]).'"/&gt;&lt;br/&gt;&lt;br/&gt;';</blockquote>
In the line above, we see that $options[$id] is placed into the rendered HTML. $options[$id] appears to be attacker controlled and the developers used the stripslashes() API to sanitize $options[$id] before displaying the value in HTML. Unfortunately, stripslashes() doesn't help prevent XSS vulnerabilities and created an opportunity for XSS. The developer fixed this vulnerability by using the correct API to sanitize against XSS vulnerability (htmlentities).
## Developers Solution
[cce lang="diff"]

&lt;?php
if (!class_exists('OV_Plugin_Admin')) {
class OV_Plugin_Admin {

var $hook               = '';
var $filename   = '';
var $longname   = '';
var $shortname  = '';
var $ozhicon    = '';
var $optionname = '';
var $homepage   = '';
var $accesslvl  = 'manage_options';

function Yoast_Plugin_Admin() {
add_action( 'admin_menu', array(&amp;$this, 'register_settings_page') );
add_filter( 'plugin_action_links', array(&amp;$this, 'add_action_link'), 10, 2 );
add_filter( 'ozh_adminmenu_icon', array(&amp;$this, 'add_ozh_adminmenu_icon' ) );

add_action('admin_print_scripts', array(&amp;$this,'config_page_scripts'));
add_action('admin_print_styles', array(&amp;$this,'config_page_styles'));
}

function add_ozh_adminmenu_icon( $hook ) {
if ($hook == $this-&gt;hook)
return WP_CONTENT_URL . '/plugins/' . plugin_basename(dirname($filename)). '/'.$this-&gt;ozhicon;
return $hook;
}

function config_page_styles() {
if (isset($_GET['page']) &amp;&amp; $_GET['page'] == $this-&gt;hook) {
wp_enqueue_style('dashboard');
wp_enqueue_style('thickbox');
wp_enqueue_style('global');
wp_enqueue_style('wp-admin');
wp_enqueue_style('ov-admin-css', WP_CONTENT_URL . '/plugins/' . plugin_basename(dirname(__FILE__)). '/yst_plugin_tools.css');
}
}

function register_settings_page() {
add_options_page($this-&gt;longname, $this-&gt;shortname, $this-&gt;accesslvl, $this-&gt;hook, array(&amp;$this,'config_page'));
}

function plugin_options_url() {
return admin_url( 'options-general.php?page='.$this-&gt;hook );
}

/**
* Add a link to the settings page to the plugins list
*/
function add_action_link( $links, $file ) {
static $this_plugin;
if( empty($this_plugin) ) $this_plugin = $this-&gt;filename;
if ( $file == $this_plugin ) {
$settings_link = '&lt;a href="' . $this-&gt;plugin_options_url() . '"&gt;' . __('Settings') . '&lt;/a&gt;';
array_unshift( $links, $settings_link );
}
return $links;
}

function config_page() {

}

function config_page_scripts() {
if (isset($_GET['page']) &amp;&amp; $_GET['page'] == $this-&gt;hook) {
wp_enqueue_script('postbox');
wp_enqueue_script('dashboard');
wp_enqueue_script('thickbox');
wp_enqueue_script('media-upload');
}
}

/**
* Create a Checkbox input field
*/
function checkbox($id, $label) {
$options = get_option($this-&gt;optionname);
return '&lt;input type="checkbox" id="'.$id.'" name="'.$id.'"'. checked($options[$id],true,false).'/&gt; &lt;label for="'.$id.'"&gt;'.$label.'&lt;/label&gt;&lt;br/&gt;';
}

/**
* Create a Text input field
*/
function textinput($id, $label) {
$options = get_option($this-&gt;optionname);
-                       return '&lt;label for="'.$id.'"&gt;'.$label.':&lt;/label&gt;&lt;br/&gt;&lt;input size="45" type="text" id="'.$id.'" name="'.$id.'" value="'.stripslashes($options[$id]).'"/&gt;&lt;br/&gt;&lt;br/&gt;';
+                       return '&lt;label for="'.$id.'"&gt;'.$label.':&lt;/label&gt;&lt;br/&gt;&lt;input size="45" type="text" id="'.$id.'" name="'.$id.'" value="'.htmlentities(stripslashes($options[$id])).'"/&gt;&lt;br/&gt;&lt;br/&gt;';
}

[/cce]
