---
layout: post
title: Radical - Cross-Site Scripting
tags:
- Cross-Site Scripting (XSS)
- Encode
- PHP
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
  _sexybookmarks_shortUrl: http://bit.ly/ffI5yX
  _sexybookmarks_permaHash: f6116e60b82ac890760a3c7ab00095c4
---
<h3>Details</h3>
__Affected Software:__ BezahlCode-Generator

__Fixed in Version:__  1.1

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Radical" href="http://spotthevuln.com/2011/02/radical/" target="_blank">Found    Here</a>
<h3>Description</h3>
A couple straightforward XSS bugs. $_REQUEST will create an associative array which contains the contents of $_GET, $_POST, and $_COOKIE which are all user/attacker controllable. These variables are then used to create HTML markup. Security bugs are caused by many different reasons. When auditing code for security issues, if you come across issues like the ones shown below its highly likely that the developer simply doesn't understand the security risk they created. It might be a good idea to review other change lists associated with this developer as they will likely contain similar code symptoms. This type of issue is also indicative of lack of security awareness. The developer here could use some security education about various security issues along with some tips on preventing these types of security issues in the future.
<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="16,17,19,20,25,26,28,29"]
&lt;?php
    if ($data!='')
    {
?&gt;
&lt;img src=&quot;/generator/?generate=&lt;?php echo urlencode($data)?&gt;&quot;/&gt;
&lt;?php
    }
?&gt;
&lt;/div&gt;&lt;br/&gt;
&lt;form action=&quot;/generator/&quot; name=&quot;wizard&quot; method=&quot;post&quot; class=&quot;BezahlCodeForm&quot;&gt;

&lt;label for=&quot;singlepayment&quot;&gt;&lt;input type=&quot;radio&quot; id=&quot;singlepayment&quot; name=&quot;gen_type&quot; value=&quot;singlepayment&quot; &lt;?php if($_REQUEST['gen_type']==&quot;singlepayment&quot; || empty($_REQUEST['gen_type'])) echo 'checked=&quot;checked&quot;'?&gt; /&gt; &amp;Uuml;berweisung&lt;/label&gt;&lt;br /&gt;
&lt;label for=&quot;singlepaymentspende&quot;&gt;&lt;input type=&quot;radio&quot; id=&quot;singlepaymentspende&quot; name=&quot;gen_type&quot; value=&quot;singlepaymentspende&quot; &lt;?php if($_REQUEST['gen_type']==&quot;singlepaymentspende&quot;) echo 'checked=&quot;checked&quot;'?&gt;/&gt; Spendenzahlung&lt;/label&gt;&lt;br /&gt;
&lt;label for=&quot;singledirectdebit&quot;&gt;&lt;input type=&quot;radio&quot; id=&quot;singledirectdebit&quot; name=&quot;gen_type&quot; value=&quot;singledirectdebit&quot; &lt;?php if($_REQUEST['gen_type']==&quot;singledirectdebit&quot;) echo 'checked=&quot;checked&quot;'?&gt;/&gt; Lastschrift&lt;/label&gt;&lt;br /&gt;

-Name:&lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: DTAUS Text&quot; id=&quot;gen_name&quot; onblur=&quot;checkInput(this, 'dtaus')&quot; name=&quot;gen_name&quot; maxlength=&quot;27&quot; value=&quot;&lt;?= isset($_REQUEST['gen_name'])?$_REQUEST['gen_name']:&quot;&quot;?&gt;&quot;&gt;
+Name:&lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: DTAUS Text&quot; id=&quot;gen_name&quot; onblur=&quot;checkInput(this, 'dtaus')&quot; name=&quot;gen_name&quot; maxlength=&quot;27&quot; value=&quot;&lt;?= isset($_REQUEST['gen_name'])?htmlspecialchars($_REQUEST['gen_name']):&quot;&quot;?&gt;&quot;&gt;
&lt;br /&gt;
-Kontonummer:&lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: Ganzzahl z.B. 1234&quot; id=&quot;gen_account&quot; onblur=&quot;checkInput(this, 'ganzzahl')&quot; name=&quot;gen_account&quot; value=&quot;&lt;?= isset($_REQUEST['gen_account'])?$_REQUEST['gen_account']:&quot;&quot;?&gt;&quot; &gt;
+Kontonummer:&lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: Ganzzahl z.B. 1234&quot; id=&quot;gen_account&quot; onblur=&quot;checkInput(this, 'ganzzahl')&quot; name=&quot;gen_account&quot; value=&quot;&lt;?= isset($_REQUEST['gen_account'])?htmlspecialchars($_REQUEST['gen_account']):&quot;&quot;?&gt;&quot; &gt;
&lt;br /&gt;
-BLZ:&lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: Ganzzahl z.B. 1234&quot; id=&quot;gen_BNC&quot; onblur=&quot;checkInput(this, 'ganzzahl')&quot; name=&quot;gen_BNC&quot; value=&quot;&lt;?= isset($_REQUEST['gen_BNC'])?$_REQUEST['gen_BNC']:&quot;&quot;?&gt;&quot; &gt;
+BLZ:&lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: Ganzzahl z.B. 1234&quot; id=&quot;gen_BNC&quot; onblur=&quot;checkInput(this, 'ganzzahl')&quot; name=&quot;gen_BNC&quot; value=&quot;&lt;?= isset($_REQUEST['gen_BNC'])?htmlspecialchars($_REQUEST['gen_BNC']):&quot;&quot;?&gt;&quot; &gt;
&lt;br /&gt;
-Betrag in Euro (z.B. 1234,50) &lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: Dezimalzahl z.B. 1234,50&quot; onblur=&quot;checkInput(this, 'dezimalzahl')&quot; id=&quot;gen_amount&quot; name=&quot;gen_amount&quot; value=&quot;&lt;?= isset($_REQUEST['gen_amount'])?$_REQUEST['gen_amount']:&quot;&quot;?&gt;&quot; &gt;
+Betrag in Euro (z.B. 1234,50) &lt;br /&gt;&lt;input type=&quot;text&quot; tooltipText=&quot;Format: Dezimalzahl z.B. 1234,50&quot; onblur=&quot;checkInput(this, 'dezimalzahl')&quot; id=&quot;gen_amount&quot; name=&quot;gen_amount&quot; value=&quot;&lt;?= isset($_REQUEST['gen_amount'])?htmlspecialchars($_REQUEST['gen_amount']):&quot;&quot;?&gt;&quot; &gt;
&lt;br /&gt;
-Verwendungszweck:&lt;br /&gt;&lt;input type=&quot;text&quot; id=&quot;gen_reason&quot; tooltipText=&quot;Format: DTAUS Text&quot; onblur=&quot;checkInput(this, 'dtaus')&quot; name=&quot;gen_reason&quot; maxlength=&quot;54&quot; value=&quot;&lt;?= isset($_REQUEST['gen_reason'])?$_REQUEST['gen_reason']:&quot;&quot;?&gt;&quot; &gt;
+Verwendungszweck:&lt;br /&gt;&lt;input type=&quot;text&quot; id=&quot;gen_reason&quot; tooltipText=&quot;Format: DTAUS Text&quot; onblur=&quot;checkInput(this, 'dtaus')&quot; name=&quot;gen_reason&quot; maxlength=&quot;54&quot; value=&quot;&lt;?= isset($_REQUEST['gen_reason'])?htmlspecialchars($_REQUEST['gen_reason']):&quot;&quot;?&gt;&quot; &gt;
&lt;br/&gt;
&lt;input type=&quot;button&quot; value=&quot;Erstellen&quot; onclick='javascript:generateImage();'&gt;
&lt;/form&gt;
&lt;?php if(!(get_option(&quot;bezahlcode_showlink&quot;) == &quot;hidden&quot;)) {	?&gt;
&lt;br /&gt;
&lt;span class=&quot;bezahlCodeLink&quot;&gt;Weitere Informationen: &lt;a href=&quot;http://www.bezahlcode.de&quot; title=&quot;BezahlCode - Schnell, einfach und sicher bezahlen&quot; target=&quot;_blank&quot;&gt;www.bezahlcode.de&lt;/a&gt;&lt;/span&gt;
&lt;?php } ?&gt;
&lt;/div&gt;

&lt;script type=&quot;text/javascript&quot;&gt;
var tooltipObj = new DHTMLgoodies_formTooltip();
tooltipObj.initFormFieldTooltip();
&lt;/script&gt;
[/sourcecode]
