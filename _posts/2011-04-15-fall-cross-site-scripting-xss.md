---
layout: post
title: Fall - Cross-Site Scripting
tags:
- Cross-Site Scripting (XSS)
- Cubed
- false sense of security
- PHP
- sanitization
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
## Details
__Affected Software:__ Cubed

__Fixed in Version:__  1.0 RC2

__Issue Type:__ Cross-Site Scripting

Original Code: <a href="http://spotthevuln.com/2011/04/Fall/">Found Here</a>
## Details
This week's patch is a good one. The code sample was basically a library that only contained functions. While there isn't a blatant vulnerability in the library, there is a startling function called "PrepDataForScript". Looking at PrepDataForScript, it's obvious this function is meant to provide some sanitization. Unfortunately, the routine isn't very robust. When you see things like the code snippet below, you know the developer is headed in the wrong direction:

$strData = str_replace("</script>", "&lt/script&gt", $strData);

Fortunately, the Cubed developers were smart enough to realize that this function is dangerous and will probably lead to a false sense of security. Instead of trying to fix it up, they just removed the function entirely.

## Developer's Solution
[sourcecode language="diff" highlight="94,95,96,97,98,99,100,101,102,103"]
&lt;?php
...snip...

	function QcodoHandleError($__exc_errno, $__exc_errstr, $__exc_errfile, $__exc_errline, $blnExit = true) {
		// If a command is called with &quot;@&quot;, then we should return
		if (error_reporting() == 0)
			return;

		if (class_exists('QApplicationBase'))
			QApplicationBase::$ErrorFlag = true;

		global $__exc_strType;
		if (isset($__exc_strType))
			return;

		$__exc_strType = &quot;Error&quot;;
		$__exc_strMessage = $__exc_errstr;

		switch ($__exc_errno) {
			case E_ERROR:
				$__exc_strObjectType = &quot;E_ERROR&quot;;
				break;
			case E_WARNING:
				$__exc_strObjectType = &quot;E_WARNING&quot;;
				break;
			case E_PARSE:
				$__exc_strObjectType = &quot;E_PARSE&quot;;
				break;
			case E_NOTICE:
				$__exc_strObjectType = &quot;E_NOTICE&quot;;
				break;
			case E_STRICT:
				$__exc_strObjectType = &quot;E_STRICT&quot;;
				break;
			case E_CORE_ERROR:
				$__exc_strObjectType = &quot;E_CORE_ERROR&quot;;
				break;
			case E_CORE_WARNING:
				$__exc_strObjectType = &quot;E_CORE_WARNING&quot;;
				break;
			case E_COMPILE_ERROR:
				$__exc_strObjectType = &quot;E_COMPILE_ERROR&quot;;
				break;
			case E_COMPILE_WARNING:
				$__exc_strObjectType = &quot;E_COMPILE_WARNING&quot;;
				break;
			case E_USER_ERROR:
				$__exc_strObjectType = &quot;E_USER_ERROR&quot;;
				break;
			case E_USER_WARNING:
				$__exc_strObjectType = &quot;E_USER_WARNING&quot;;
				break;
			case E_USER_NOTICE:
				$__exc_strObjectType = &quot;E_USER_NOTICE&quot;;
				break;
			default:
				$__exc_strObjectType = &quot;Unknown&quot;;
				break;
		}

		$__exc_strFilename = $__exc_errfile;
		$__exc_intLineNumber = $__exc_errline;
		$__exc_strStackTrace = &quot;&quot;;
		$__exc_objBacktrace = debug_backtrace();
		for ($__exc_intIndex = 0; $__exc_intIndex &lt; count($__exc_objBacktrace); $__exc_intIndex++) {
			$__exc_objItem = $__exc_objBacktrace[$__exc_intIndex];

			$__exc_strKeyFile = (array_key_exists(&quot;file&quot;, $__exc_objItem)) ? $__exc_objItem[&quot;file&quot;] : &quot;&quot;;
			$__exc_strKeyLine = (array_key_exists(&quot;line&quot;, $__exc_objItem)) ? $__exc_objItem[&quot;line&quot;] : &quot;&quot;;
			$__exc_strKeyClass = (array_key_exists(&quot;class&quot;, $__exc_objItem)) ? $__exc_objItem[&quot;class&quot;] : &quot;&quot;;
			$__exc_strKeyType = (array_key_exists(&quot;type&quot;, $__exc_objItem)) ? $__exc_objItem[&quot;type&quot;] : &quot;&quot;;
			$__exc_strKeyFunction = (array_key_exists(&quot;function&quot;, $__exc_objItem)) ? $__exc_objItem[&quot;function&quot;] : &quot;&quot;;

			$__exc_strStackTrace .= sprintf(&quot;#%s %s(%s): %s%s%s()\n&quot;,
				$__exc_intIndex,
				$__exc_strKeyFile,
				$__exc_strKeyLine,
				$__exc_strKeyClass,
				$__exc_strKeyType,
				$__exc_strKeyFunction);
		}

		if (ob_get_length()) {
			$__exc_strRenderedPage = ob_get_contents();
			ob_clean();
		}

		// Call to display the Error Page (as defined in configuration.inc.php)
		require(__DOCROOT__ . ERROR_PAGE_PATH);
		if($blnExit)
			exit;
	}

-	function PrepDataForScript($strData) {
-		$strData = str_replace(&quot;\\&quot;, &quot;\\\\&quot;, $strData);
-		$strData = str_replace(&quot;\n&quot;, &quot;\\n&quot;, $strData);
-		$strData = str_replace(&quot;\r&quot;, &quot;\\r&quot;, $strData);
-		$strData = str_replace(&quot;\&quot;&quot;, &quot;&amp;quot;&quot;, $strData);
-		$strData = str_replace(&quot;&lt;/script&gt;&quot;, &quot;&amp;lt/script&amp;gt&quot;, $strData);
-		$strData = str_replace(&quot;&lt;/Script&gt;&quot;, &quot;&amp;lt/script&amp;gt&quot;, $strData);
-		$strData = str_replace(&quot;&lt;/SCRIPT&gt;&quot;, &quot;&amp;lt/script&amp;gt&quot;, $strData);
-		return $strData;
-	}
?&gt;
[/sourcecode]
