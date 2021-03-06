---
layout: post
title: Character - Cross-Site Scripting
tags:
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- html markup
- javascript eval
- PhotoSmash
- PHP
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
---
<h3>Details</h3>
__Affected Software:__ PhotoSmash

__Fixed in Version:__  1.0.5

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Character" href="http://spotthevuln.com/2011/02/character/" target="_blank">Found    Here</a>
<h3>Description</h3>
Once again, we see the familiar pattern of the developer taking user/attacker controlled values and using those values to build HTML markup. Line 76 is the start of a large echo statement which writes a couple input fields to markup. The developer uses the $_REQUEST['bwbps_galname'] variable to populate the value attribute for one of the input form fields. Although not completely clear from the code snippet, the developers addressed this issue by placing an encoded version of $_REQUEST['bwbps_galname'] into a variable named $gallery_name and using the newly encoded value to build the HTML markup.

Although not addressed by this patch, there are a couple of areas that deserve deeper inspection. For example, on line 113 the application is calling a javascript eval on an unknown function. If this function contains user/attacker supplied content, this could result in XSS. Additionally, on line 136 it seems the user/attacker has some influence on variables passed to a SWF object. If the SWF doesn't have the appropriate logic to handle the tainted data, this could result in a security vulnerability.



<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="76-81,111,134-136"]
&lt;?php
...snip...
	//Get a link for the Start Slideshow for PicLens
	function getPicLensLink($g, $atts){
		if($atts['link_text']){
			$link_text = $atts['link_text'];
		} else {
			$link_text = 'Start Slideshow
  &lt;img src=&quot;http://lite.piclens.com/images/PicLensButton.png&quot;
  alt=&quot;PicLens&quot; width=&quot;16&quot; height=&quot;12&quot; border=&quot;0&quot; align=&quot;absmiddle&quot;&gt;';
		}

		$picatts['id'] = $g['gallery_id'];
		$picatts['thumb_width'] = $g['thumb_width'];
		$picatts['thumb_height'] = $g['thumb_height'];
		$picatts['gallery_type'] = $g['gallery_type'];
		$picatts['images'] = $g['images'];
		$picatts['page'] = $g['page'];


		if($g['tags'] == 'post_tags'){
			$picatts['tags'] = $this-&gt;getPostTags(0);
		} else {
			$picatts['tags'] = $g['tags'];
		}

		$param_array = $this-&gt;filterMRSSAttsFromArray($picatts, &quot;&quot;);

		if( is_array($param_array)){
			$params = implode(&quot;&amp;&quot;, $param_array);
			//$params = urlencode($params);
		}

		$ret = '&lt;a class=&quot;piclenselink&quot; href=&quot;javascript:PicLensLite.start({feedUrl:\''
			. plugins_url() . '/photosmash-galleries/bwbps-media-rss.php?'
			. $params . '\'});&quot;&gt;
			' . $link_text . ' &lt;/a&gt;
			';

		return $ret;
	}

	function getPostTags($post_id){

		if(!$post_id ){
			global $wp_query;
			$post_id = $wp_query-&gt;post-&gt;ID;
		}
		$terms = wp_get_object_terms( $post_id, 'post_tag', $args ) ;

		if(is_array($terms)){

			foreach( $terms as $term ){

				$_terms[] = $term-&gt;name;

			}

			unset($terms);
			if( is_array($_terms)){
				$ret = implode(&quot;,&quot; , $_terms);
			} else {
				$ret = &quot;&quot;;
			}
		}

		return $ret;
	}


	/*		SECTION:  Media Uploader Integration
	 * 		Media Uploader Integration for Admin -&gt; Photo Manager uploading images
	 *
	*/
	function mediaUAddGalleryFieldToMediaUploader(){
		if(isset($_REQUEST['bwbps_galid']) &amp;&amp; (int)$_REQUEST['bwbps_galid']){

			echo &quot;&lt;input type='hidden' id='bwbps_mediau_galid' name='bwbps_mediau_galid' value='&quot; . (int)$_REQUEST['bwbps_galid'] . &quot;' /&gt;
			&lt;input type='hidden' id='bwbps_galid' name='bwbps_galid' value='&quot; . (int)$_REQUEST['bwbps_galid'] . &quot;' /&gt;
-			&lt;input type='hidden' name='bwbps_galname' value='&quot; . $_REQUEST['bwbps_galname'] . &quot;' /&gt;
-			&lt;div style='background-color: #eaffdf; padding: 5px; border: 1px solid #a0a0a0; margin: 3px; font-size: 14px; color: #333;'&gt;Adding to PhotoSmash: &quot; . $_REQUEST['bwbps_galname'] . &quot;&lt;/div&gt;
+			&lt;input type='hidden' name='bwbps_galname' value='&quot; . $gallery_name . &quot;' /&gt;
+			&lt;div style='background-color: #eaffdf; padding: 5px; border: 1px solid #a0a0a0; margin: 3px; font-size: 14px; color: #333;'&gt;Adding to PhotoSmash: &quot; . $gallery_name . &quot;&lt;/div&gt;
			&quot;;

		} else {

			$gid = isset($_REQUEST['bwbps_mediau_galid']) ? (int)$_REQUEST['bwbps_mediau_galid'] : 0;

			$galleryDDL = $this-&gt;getGalleryDDL($gid, &quot;select gallery&quot;, &quot;&quot;, &quot;bwbps_mediau_galid&quot;, 30, true, true);
			echo &quot;&lt;div style='padding: 5px; margin: 3px; font-size: 14px; color: #333;'&gt;Add to PhotoSmash: $galleryDDL&lt;/div&gt;&quot;;
		}
	}

	function mediaUAddGalleryFieldToFlashUploader(){

			?&gt;
			&lt;script type=&quot;text/javascript&quot;&gt;

			if (typeof flashStartUploadFunctions == 'undefined'){

				var flashStartUploadFunctions = [];
				function addFlashStartUploadFunction( funct_name ){
					flashStartUploadFunctions.push( funct_name );

				}

				function runFlashStartUploadFunctions(){
					if( flashStartUploadFunctions.length &gt; 0 ){
						var bwbfunc;
						for( bwbfunc in flashStartUploadFunctions){

								eval(flashStartUploadFunctions[ bwbfunc ]);

						}
					}
				}

			}

			addFlashStartUploadFunction( 'bwbpsAddGalleryToFlashUploader();' );

				jQuery(window).load( function() {
					swfu.settings.upload_start_handler = function(){
						runFlashStartUploadFunctions();
					}
				});

				function bwbpsAddGalleryToFlashUploader(){
					jQuery('#bwbps_uploaded_images', top.document).show().append('&lt;h4&gt;Flash upload...preview not available.&lt;/h4&gt;');
					var gid = jQuery(&quot;#bwbps_mediau_galid_flash&quot;).val() + &quot;&quot;;

					if( gid ){
						swfu.addPostParam('bwbps_mediau_galid', gid);
						&lt;?php
						if(isset($_REQUEST['bwbps_galid']) ){
						?&gt;
						swfu.addPostParam('bwbps_galid', gid);
						&lt;?php
						}
						?&gt;
					}
				}

			&lt;/script&gt;
			&lt;?php

		if(isset($_REQUEST['bwbps_galid']) &amp;&amp; (int)$_REQUEST['bwbps_galid']){

			$this-&gt;count++;

			echo &quot;
			&lt;script type='text/javascript'&gt;
				jQuery(window).load( function() {
				//Hide the other Media Tabs
					jQuery('#tab-type_url').hide();
					jQuery('#tab-library').hide();&quot;;

...snip...
?&gt;
[/sourcecode]
