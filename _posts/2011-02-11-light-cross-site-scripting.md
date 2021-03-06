---
layout: post
title: Light - Cross-Site Scripting
tags:
- All
- Cross-Site Scripting (XSS)
- FreeNAS
- html markup
- htmlspecialchars
- PHP
- Solution
- svg xml
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/hhETkG
  _sexybookmarks_permaHash: 565cfdb26bfe66c37240c12d0e0237e3
---
<h3>Details</h3>
__Affected Software:__ FreeNAS

__Fixed in Version:__  0.69.3

__Issue Type:__ Cross-Site Scripting

Original Code: <a title="Light" href="http://spotthevuln.com/2011/02/light/" target="_blank">Found    Here</a>
<h3>Description</h3>
The code sample for this week contained a couple XSS vulnerabilities. Although not essential for exploitation, its also interesting to note that this response is within an SVG image. You can see this by examining the header() api specifying the content-type: header("Content-type: image/svg+xml");

The first issue is pretty easy to follow, so we'll begin there. On line 11, $ifname is assigned a tainted value from $_GET["ifname"]. After the variable assignment, the authors use the tainted variable to build HTML markup on line 66.

The second issue requires a little bit of tracing. First, the $ifnum variable is assigned a tainted value from $_GET["ifnum"] on line 10. $ifnum is then used to build the $fetch_link variable on line 18. If $fetch_link is ever used to build HTML markup, it will result in XSS.

The third issue also requires a bit of tracing as well. Once again, we start with the assignment of a tainted variable to $ifnum on line 10. $ifnum is then used to build an error message on line 37 ($error_text). $error_text is then used to build HTML markup on line 72 resulting in XSS.

The developers addressed this issue by using htmlspecialchars() during the inital variable assignments. This takes care of all three of the XSS issues described above.
<h3>Developer's Solution</h3>
[sourcecode language="diff" highlight="8-11,18,37,66,72"]
&lt;?php
... snip ...
require(&quot;guiconfig.inc&quot;);

header(&quot;Content-type: image/svg+xml&quot;);

/********** HTTP GET Based Conf ***********/
+$ifnum=@htmlspecialchars($_GET[&quot;ifnum&quot;]);  // BSD / SNMP interface name / number
+$ifname=@htmlspecialchars($_GET[&quot;ifname&quot;]) ? htmlspecialchars($_GET[&quot;ifname&quot;]) : &quot;Interface $ifnum&quot;;  //Interface name that will be showed on top right of graph
-$ifnum=@$_GET[&quot;ifnum&quot;];  // BSD / SNMP interface name / number
-$ifname=@$_GET[&quot;ifname&quot;]?$_GET[&quot;ifname&quot;]:&quot;Interface $ifnum&quot;;  //Interface name that will be showed on top right of graph

/********* Other conf *******/
$scale_type=&quot;follow&quot;; //Autoscale default setup : &quot;up&quot; = only increase scale; &quot;follow&quot; = increase and decrease scale according to current graphed datas
$nb_plot=120;         //NB plot in graph
$time_interval=1;		  //Refresh time Interval
$unit=&quot;bits&quot;;         //Initial unit type: &quot;bits&quot; or &quot;bytes&quot;
$fetch_link = &quot;stats.php?if=$ifnum&quot;;

//SVG attributes
$attribs['bg']='fill=&quot;#EEEEEE&quot; stroke=&quot;none&quot; stroke-width=&quot;0&quot; opacity=&quot;1&quot;';
$attribs['axis']='fill=&quot;black&quot; stroke=&quot;black&quot;';
$attribs['in']='fill=&quot;#00CC00&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;7&quot;';
$attribs['out']='fill=&quot;#FF0000&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;7&quot;';
$attribs['graph_in']='fill=&quot;none&quot; stroke=&quot;#00CC00&quot; stroke-opacity=&quot;0.8&quot;';
$attribs['graph_out']='fill=&quot;none&quot; stroke=&quot;#FF0000&quot; stroke-opacity=&quot;0.8&quot;';
$attribs['legend']='fill=&quot;black&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;4&quot;';
$attribs['graphname']='fill=&quot;#435370&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;8&quot;';
$attribs['grid_txt']='fill=&quot;gray&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;6&quot;';
$attribs['grid']='stroke=&quot;gray&quot; stroke-opacity=&quot;0.5&quot;';
$attribs['switch_unit']='fill=&quot;#435370&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;4&quot; text-decoration=&quot;underline&quot;';
$attribs['switch_scale']='fill=&quot;#435370&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;4&quot; text-decoration=&quot;underline&quot;';
$attribs['error']='fill=&quot;red&quot; font-family=&quot;Arial&quot; font-size=&quot;4&quot;';
$attribs['collect_initial']='fill=&quot;gray&quot; font-family=&quot;Tahoma, Verdana, Arial, Helvetica, sans-serif&quot; font-size=&quot;4&quot;';

//Error text if we cannot fetch data : depends on which method is used
$error_text = gettext(&quot;Cannot get data about interface&quot;) . &quot; $ifnum&quot;;

$height=100;            //SVG internal height : do not modify
$width=200;             //SVG internal width : do not modify

$encoding = system_get_language_codeset();

/********* Graph DATA **************/
header(&quot;Last-Modified: &quot; . gmdate( &quot;D, j M Y H:i:s&quot; ) . &quot; GMT&quot;);
header(&quot;Expires: &quot; . gmdate( &quot;D, j M Y H:i:s&quot;, time() ) . &quot; GMT&quot;);
header(&quot;Cache-Control: no-store, no-cache, must-revalidate&quot;); // HTTP/1.1
header(&quot;Cache-Control: post-check=0, pre-check=0&quot;, FALSE);
header(&quot;Pragma: no-cache&quot;); // HTTP/1.0
header(&quot;Content-type: image/svg+xml&quot;);
echo &quot;&lt;?xml version=\&quot;1.0\&quot; encoding=\&quot;{$encoding}\&quot;?&gt;\n&quot;;
?&gt;
&lt;svg width=&quot;100%&quot; height=&quot;100%&quot; viewBox=&quot;0 0 &lt;?=$width?&gt; &lt;?=$height?&gt;&quot; preserveAspectRatio=&quot;none&quot; xml:space=&quot;preserve&quot; xmlns=&quot;http://www.w3.org/2000/svg&quot; xmlns:xlink=&quot;http://www.w3.org/1999/xlink&quot; onload=&quot;init(evt)&quot;&gt;
  &lt;g id=&quot;graph&quot;&gt;
    &lt;rect id=&quot;bg&quot; x1=&quot;0&quot; y1=&quot;0&quot; width=&quot;100%&quot; height=&quot;100%&quot; &lt;?=$attribs['bg']?&gt;/&gt;
    &lt;line id=&quot;axis_x&quot; x1=&quot;0&quot; y1=&quot;0&quot; x2=&quot;0&quot; y2=&quot;100%&quot; &lt;?=$attribs['axis']?&gt;/&gt;
    &lt;line id=&quot;axis_y&quot; x1=&quot;0&quot; y1=&quot;100%&quot; x2=&quot;100%&quot; y2=&quot;100%&quot; &lt;?=$attribs['axis']?&gt;/&gt;
    &lt;path id=&quot;graph_out&quot; d=&quot;M0 &lt;?=$height?&gt; L 0 &lt;?=$height?&gt;&quot; &lt;?=$attribs['graph_out']?&gt;/&gt;
    &lt;path id=&quot;graph_in&quot;  d=&quot;M0 &lt;?=$height?&gt; L 0 &lt;?=$height?&gt;&quot; &lt;?=$attribs['graph_in']?&gt;/&gt;
    &lt;path id=&quot;grid&quot;  d=&quot;M0 &lt;?=$height/4*1?&gt; L &lt;?=$width?&gt; &lt;?=$height/4*1?&gt; M0 &lt;?=$height/4*2?&gt; L &lt;?=$width?&gt; &lt;?=$height/4*2?&gt; M0 &lt;?=$height/4*3?&gt; L &lt;?=$width?&gt; &lt;?=$height/4*3?&gt;&quot; &lt;?=$attribs['grid']?&gt;/&gt;
    &lt;text id=&quot;grid_txt1&quot; x=&quot;&lt;?=$width?&gt;&quot; y=&quot;&lt;?=$height/4*1?&gt;&quot; &lt;?=$attribs['grid_txt']?&gt; text-anchor=&quot;end&quot;&gt;75%&lt;/text&gt;
    &lt;text id=&quot;grid_txt2&quot; x=&quot;&lt;?=$width?&gt;&quot; y=&quot;&lt;?=$height/4*2?&gt;&quot; &lt;?=$attribs['grid_txt']?&gt; text-anchor=&quot;end&quot;&gt;50%&lt;/text&gt;
    &lt;text id=&quot;grid_txt3&quot; x=&quot;&lt;?=$width?&gt;&quot; y=&quot;&lt;?=$height/4*3?&gt;&quot; &lt;?=$attribs['grid_txt']?&gt; text-anchor=&quot;end&quot;&gt;25%&lt;/text&gt;
    &lt;text id=&quot;graph_in_lbl&quot; x=&quot;5&quot; y=&quot;8&quot; &lt;?=$attribs['in']?&gt;&gt;&lt;?=gettext(&quot;In&quot;);?&gt; &lt;tspan id=&quot;graph_in_txt&quot; &lt;?=$attribs['in']?&gt;&gt; &lt;/tspan&gt;&lt;/text&gt;
    &lt;text id=&quot;graph_out_lbl&quot; x=&quot;5&quot; y=&quot;16&quot; &lt;?=$attribs['out']?&gt;&gt;&lt;?=gettext(&quot;Out&quot;);?&gt; &lt;tspan id=&quot;graph_out_txt&quot; &lt;?=$attribs['out']?&gt;&gt; &lt;/tspan&gt;&lt;/text&gt;
    &lt;text id=&quot;ifname&quot; x=&quot;&lt;?=$width?&gt;&quot; y=&quot;8&quot; &lt;?=$attribs['graphname']?&gt; text-anchor=&quot;end&quot;&gt;&lt;?=$ifname?&gt;&lt;/text&gt;
    &lt;text id=&quot;switch_unit&quot; x=&quot;&lt;?=$width*0.55?&gt;&quot; y=&quot;5&quot; &lt;?=$attribs['switch_unit']?&gt;&gt;&lt;?=sprintf(gettext(&quot;Switch to %s/s&quot;), (&quot;bits&quot; === $unit) ? &quot;bytes&quot; : &quot;bits&quot;);?&gt;&lt;/text&gt;
    &lt;text id=&quot;switch_scale&quot; x=&quot;&lt;?=$width*0.55?&gt;&quot; y=&quot;11&quot; &lt;?=$attribs['switch_scale']?&gt;&gt;&lt;?=gettext(&quot;AutoScale&quot;);?&gt; (&lt;?=(&quot;up&quot; === $scale_type) ? gettext(&quot;Up&quot;) : gettext(&quot;Follow&quot;);?&gt;)&lt;/text&gt;
    &lt;text id=&quot;datetime&quot; x=&quot;&lt;?=$width*0.55?&gt;&quot; y=&quot;17&quot; &lt;?=$attribs['legend']?&gt;&gt; &lt;/text&gt;
    &lt;text id=&quot;graphlast&quot; x=&quot;&lt;?=$width*0.55?&gt;&quot; y=&quot;23&quot; &lt;?=$attribs['legend']?&gt;&gt;&lt;?=gettext(&quot;Graph shows last&quot;);?&gt; &lt;?=$time_interval*$nb_plot?&gt; &lt;?=gettext(&quot;seconds&quot;);?&gt;&lt;/text&gt;
    &lt;polygon id=&quot;axis_arrow_x&quot; &lt;?=$attribs['axis']?&gt; points=&quot;&lt;?=($width) . &quot;,&quot; . ($height)?&gt; &lt;?=($width-2) . &quot;,&quot; . ($height-2)?&gt; &lt;?=($width-2) . &quot;,&quot; . $height?&gt;&quot;/&gt;
    &lt;text id=&quot;error&quot; x=&quot;&lt;?=$width*0.5?&gt;&quot; y=&quot;&lt;?=$height*0.4?&gt;&quot; visibility=&quot;hidden&quot; &lt;?=$attribs['error']?&gt; text-anchor=&quot;middle&quot;&gt;&lt;?=$error_text?&gt;&lt;/text&gt;
    &lt;text id=&quot;collect_initial&quot; x=&quot;&lt;?=$width*0.5?&gt;&quot; y=&quot;&lt;?=$height*0.4?&gt;&quot; visibility=&quot;hidden&quot; &lt;?=$attribs['collect_initial']?&gt; text-anchor=&quot;middle&quot;&gt;&lt;?=gettext(&quot;Collecting initial data, please wait...&quot;);?&gt;&lt;/text&gt;
  &lt;/g&gt;
  &lt;script type=&quot;text/ecmascript&quot;&gt;
    &lt;![CDATA[

/**
 * getURL is a proprietary Adobe function, but it's simplicity has made it very
 * popular. If getURL is undefined we spin our own by wrapping XMLHttpRequest.
 */
if (typeof getURL == 'undefined') {
  getURL = function(url, callback) {
    if (!url)
      throw 'No URL for getURL';

    try {
      if (typeof callback.operationComplete == 'function')
        callback = callback.operationComplete;
    } catch (e) {}
    if (typeof callback != 'function')
      throw 'No callback function for getURL';

    var http_request = null;
    if (typeof XMLHttpRequest != 'undefined') {
      http_request = new XMLHttpRequest();
    }
    else if (typeof ActiveXObject != 'undefined') {
      try {
        http_request = new ActiveXObject('Msxml2.XMLHTTP');
      } catch (e) {
        try {
          http_request = new ActiveXObject('Microsoft.XMLHTTP');
        } catch (e) {}
      }
    }
    if (!http_request)
      throw 'Both getURL and XMLHttpRequest are undefined';

    http_request.onreadystatechange = function() {
      if (http_request.readyState == 4) {
        callback( { success : true,
                    content : http_request.responseText,
                    contentType : http_request.getResponseHeader(&quot;Content-Type&quot;) } );
      }
    }
    http_request.open('GET', url, true);
    http_request.send(null);
  }
}
[/sourcecode]
