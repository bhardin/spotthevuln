---
layout: post
title: Proportion - Cross-Site Scripting
tags:
- Cross-Site Scripting
- Cross-Site Scripting (XSS)
- PHP
- plugins
- query string
- querystring
- Solution
- Wordpress
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
## Details
__Affected Software:__ Lazyest-Gallery

__Fixed in Version:__  0.9

__Issue Type:__ Cross-Site Scripting

Original Code: <a href="http://spotthevuln.com/2011/03/proportion/">Found Here</a>
## Details
For most security issues, I give the developer the benefit of the doubt. It's tough to keep track of all the corner cases and security nuances. For this diff however, there is no excuse.

First, let's cover what the patch fixes. On line 18, the developer was taking a tainted value passed via query string parameter and using that value to build HTML markup. This is XSS in its most classic form. Also, on line 58 the same tainted input is used to build the SRC attribute for an image tag, also resulting in XSS. The developer chose to encode both of these tainted values before using them in the HTML output.

Now, let's talk about the problems with this patch. First, the tainted value used to build the SRC attribute for an image tag needs additional validation. SRC attributes are tricky as they usually cause the browser to issue a request. Escaping the tainted SRC value only prevents the attacker from breaking out of the attribute and injecting their own HTML. Escaping doesn't prevent the attacker from passing a well formed URI like javascript:javascript-payload-here. I can let the developer slide on this one... chalk it up as a lesson on corner cases. Now, if you look at the patched line, you'll see that the ALT attribute for the same image tag also contains a XSS vulnerability. Yes, the developer missed a XSS vulnerability that is less than 5 characters away from a fixed XSS vulnerability. This also shows that the developer never tested the patch. The tainted query string parameter is the same for all the vulnerable sections. If the developer tried to test this patch, they would have discovered they were still exposed...

## Developer's Solution
[sourcecode language="diff" highlight="18,19,59,60"]
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

-		&lt;title&gt;&lt;?php echo $_GET['image'] ?&gt;&lt;/title&gt;
+		&lt;title&gt;&lt;?php echo esc_html($_GET['image']) ?&gt;&lt;/title&gt;
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
-			&lt;img src=&quot;&lt;?php echo str_replace(&quot; &quot;, &quot;%20&quot;, $lg_gallery-&gt;address.$_GET['folder'].$_GET['image']); ?&gt;&quot; alt=&quot;&lt;?php echo $_GET['image']; ?&gt;&quot; /&gt;
+			&lt;img src=&quot;&lt;?php echo str_replace(&quot; &quot;, &quot;%20&quot;, $lg_gallery-&gt;address.esc_attr($_GET['folder']).esc_attr($_GET['image'])); ?&gt;&quot; alt=&quot;&lt;?php echo $_GET['image']; ?&gt;&quot; /&gt;
		&lt;/a&gt;
	&lt;/body&gt;
&lt;/html&gt;

&lt;?php

?&gt;
[/sourcecode]
