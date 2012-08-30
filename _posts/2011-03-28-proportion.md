---
layout: post
title: Proportion
tags:
- Code Snippet
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
<blockquote><strong>Rocket science has been mythologized all out of proportion to its true difficulty.
John Carmack</strong></blockquote>
[sourcecode language="php"]
&lt;?php

	// Don't remove this lines:
	require_once('../../../wp-blog-header.php');
	global $lg_gallery;

?&gt;

&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.1//EN&quot; &quot;http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd&quot;&gt;

&lt;html xmlns=&quot;http://www.w3.org/1999/xhtml&quot;&gt;

	&lt;head&gt;

		&lt;meta http-equiv=&quot;Content-Type&quot; content=&quot;&lt;?php bloginfo('html_type'); ?&gt;; charset=&lt;?php bloginfo('charset'); ?&gt;&quot; /&gt;
		&lt;meta name=&quot;generator&quot; content=&quot;WordPress &lt;?php bloginfo('version'); ?&gt;&quot; /&gt;

		&lt;title&gt;&lt;?php echo $_GET['image'] ?&gt;&lt;/title&gt;
		
		&lt;style type=&quot;text/css&quot;&gt;
			body {
				text-align:center;
				margin:0;
				padding:0;
			}
			img {
				border:none;
			}
		&lt;/style&gt;
		&lt;script type=&quot;text/javascript&quot;&gt;
		function WinWidth()	{
			if (window.innerWidth!=window.undefined) return window.innerWidth; 
			if (document.compatMode=='CSS1Compat') return document.documentElement.clientWidth; 
			if (document.body) return document.body.clientWidth; 
			return window.undefined; 
		}
		
		function WinHeight() {
			if (window.innerHeight!=window.undefined) return window.innerHeight; 
			if (document.compatMode=='CSS1Compat') return document.documentElement.clientHeight; 
			if (document.body) return document.body.clientHeight; 
			return window.undefined; 
		}
		
		function FitPic() { 
			iWidth=WinWidth();
			iHeight=WinHeight();
			iWidth = document.images[0].width - iWidth; 
			iHeight = document.images[0].height - iHeight; 
			window.resizeBy((iWidth), (iHeight))
			self.focus(); 
		} 

		&lt;/script&gt;
	&lt;/head&gt;

	&lt;body onload=&quot;FitPic()&quot;&gt;
		&lt;a href=&quot;javascript:self.close()&quot; title=&quot;&lt;?php _e('Click to close', $lg_text_domain); ?&gt;&quot;&gt;
			&lt;img src=&quot;&lt;?php echo str_replace(&quot; &quot;, &quot;%20&quot;, $lg_gallery-&gt;address.$_GET['folder'].$_GET['image']); ?&gt;&quot; alt=&quot;&lt;?php echo $_GET['image']; ?&gt;&quot; /&gt;
		&lt;/a&gt;
	&lt;/body&gt;
&lt;/html&gt;

&lt;?php

?&gt;
[/sourcecode] 
