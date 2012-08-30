---
layout: post
title: Charming - XSS (uhhh wait, actually - SQL Injection)
tags:
- addslashes
- cross site scripting
- Cross-Site Scripting (XSS)
- injection bug
- PHP
- Solution
- SQL Injection
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
__Affected Software:__ StatPressCN 

__Fixed in Version:__  1.9.1

__Issue Type:__ SQL Injection

Original Code: <a href="http://spotthevuln.com/2011/04/charming/">Found Here</a>
## Details
This patch was full of interesting tidbits.  First, the change log for this patch is as follows:
<blockquote>
**1.9.1** 
+ fix a flaw allowing a remote cross-site scripting attack
</blockquote>
Keep the change list description in mind as we go over the patch submitted by the developers.  The submitted patch is pretty simple.  There is an additional qualifier set for an if statement that checks to see if $_GET["where$i"] is contained within array $f.  It's difficult to determine whether this is true... but it doesn't really matter.  The second change is an addslashes  to $_GET["what$i"] before using the tainted query string parameter to build a dynamic SQL statement.  This is to prevent an obvious SQL injection bug in the LIKE operator of the SQL statement.

What's surprising is the developer missed the $_GET["where$i"] query string parameter used to build the SQL statement on the same line.  This bug is equally devastating and results in SQL injection against the application.  So despite the change log description, this patch is to address a SQL injection bug, NOT an XSS.

Looking through the rest of the code, we see XSS (lines 7-9 and 17) and SQL injection bugs (lines 57,65, 77) littered throughout the code base.  These bugs still exist in the latest version, are not patched, and put users at risk.  If you have this plug-in installed, your server and users are at significant risk!


## Developers Solution
[sourcecode language="diff" highlight="7-9,57,65,77"]
        &lt;/table&gt;
        &lt;br&gt;
        &lt;table&gt;
            &lt;tr&gt;
                &lt;td&gt;
                    &lt;table&gt;
                        &lt;tr&gt;&lt;td&gt;&lt;input type=checkbox name=oderbycount value=checked &lt;?php print $_GET['oderbycount'] ?&gt;&gt; &lt;?php _e('sort by count if grouped','statpresscn'); ?&gt;&lt;/td&gt;&lt;/tr&gt;
                        &lt;tr&gt;&lt;td&gt;&lt;input type=checkbox name=spider value=checked &lt;?php print $_GET['spider'] ?&gt;&gt; &lt;?php _e('include spiders/crawlers/bot','statpresscn'); ?&gt;&lt;/td&gt;&lt;/tr&gt;
                        &lt;tr&gt;&lt;td&gt;&lt;input type=checkbox name=feed value=checked &lt;?php print $_GET['feed'] ?&gt;&gt; &lt;?php _e('include feed','statpresscn'); ?&gt;&lt;/td&gt;&lt;/tr&gt;
                    &lt;/table&gt;
                &lt;/td&gt;
                &lt;td width=15&gt; &lt;/td&gt;
                &lt;td&gt;
                    &lt;table&gt;
                        &lt;tr&gt;
                            &lt;td&gt;&lt;?php _e('Limit results to','statpresscn'); ?&gt;
                                &lt;select name=limitquery&gt;&lt;?php if($_GET['limitquery'] &gt;0) { print &quot;&lt;option&gt;&quot;.$_GET['limitquery'].&quot;&lt;/option&gt;&quot;;} ?&gt;&lt;option&gt;200&lt;/option&gt;&lt;option&gt;150&lt;/option&gt;&lt;option&gt;50&lt;/option&gt;&lt;/select&gt;
                            &lt;/td&gt;
                        &lt;/tr&gt;
                        &lt;tr&gt;&lt;td&gt;&amp;nbsp;&lt;/td&gt;&lt;/tr&gt;
                        &lt;tr&gt;
                            &lt;td align=right&gt;&lt;input type=submit value=&lt;?php _e('Search','statpresscn'); ?&gt; name=searchsubmit&gt;&lt;/td&gt;
                        &lt;/tr&gt;
                    &lt;/table&gt;
                &lt;/td&gt;
            &lt;/tr&gt;
        &lt;/table&gt;&lt;!-- It's strange that the page value should be spc-search, and not others. --&gt;
        &lt;input type=hidden name=page value='spc-search'&gt;&lt;input type=hidden name=statpress_action value=search&gt;
    &lt;/form&gt;&lt;br&gt;
           &lt;?php
        if(isset($_GET['searchsubmit'])) {
        # query builder
            $qry=&quot;&quot;;
            # FIELDS
            $fields=&quot;&quot;;
            for($i=1;$i&lt;=5;$i++) {
-               if($_GET[&quot;where$i&quot;] != '') {
+				if($_GET[&quot;where$i&quot;] != '' &amp;&amp; array_key_exists($_GET[&quot;where$i&quot;], $f)) {//??where??????
                    $fields.=$_GET[&quot;where$i&quot;].&quot;,&quot;;
                }
            }
            $fields=rtrim($fields,&quot;,&quot;);
            # WHERE
            $where=&quot;WHERE 1=1&quot;;
            if($_GET['spider'] != 'checked') { $where.=&quot; AND spider=''&quot;; }
            if($_GET['feed'] != 'checked') { $where.=&quot; AND feed=''&quot;; }
            for($i=1;$i&lt;=5;$i++) {
                if(($_GET[&quot;what$i&quot;] != '') &amp;&amp; ($_GET[&quot;where$i&quot;] != '')) {
-                   $where.=&quot; AND &quot;.$_GET[&quot;where$i&quot;].&quot; LIKE '%&quot;.$_GET[&quot;what$i&quot;].&quot;%'&quot;;
+					$where.=&quot; AND &quot;.$_GET[&quot;where$i&quot;].&quot; LIKE '%&quot;.addslashes($_GET[&quot;what$i&quot;]).&quot;%'&quot;;//addslashes??????
                }
            }
            # ORDER BY
            $orderby=&quot;&quot;;
            for($i=1;$i&lt;=5;$i++) {
                if(($_GET[&quot;sortby$i&quot;] == 'checked') &amp;&amp; ($_GET[&quot;where$i&quot;] != '')) {
                    $orderby.=$_GET[&quot;where$i&quot;].',';
                }
            }

            # GROUP BY
            $groupby=&quot;&quot;;
            for($i=1;$i&lt;=5;$i++) {
                if(($_GET[&quot;groupby$i&quot;] == 'checked') &amp;&amp; ($_GET[&quot;where$i&quot;] != '')) {
                    $groupby.=$_GET[&quot;where$i&quot;].',';
                }
            }
            if($groupby != '') {
                $grouparray = explode(&quot;,&quot;,rtrim($groupby,','));
                $groupby=&quot;GROUP BY &quot;.rtrim($groupby,',');
                $fields.=&quot;,count(*) as totale&quot;;
                if($_GET['oderbycount'] == 'checked') { $orderby=&quot;totale DESC,&quot;.$orderby; }
            }

            if($orderby != '') { $orderby=&quot;ORDER BY &quot;.rtrim($orderby,','); }


            $limit=&quot;LIMIT &quot;.$_GET['limitquery'];

            # Results
            print &quot;&lt;h2&gt;&quot;.__('Results','statpresscn').&quot;&lt;/h2&gt;&quot;;
            $sql=&quot;SELECT $fields FROM $table_name $where $groupby $orderby $limit;&quot;;
            //	print &quot;$sql&lt;br&gt;&quot;;
            print &quot;&lt;table class='widefat'&gt;&lt;thead&gt;&lt;tr&gt;&quot;;
            for($i=1;$i&lt;=5;$i++) {
                if($_GET[&quot;where$i&quot;] != '') { 
                    print &quot;&lt;th scope='col'&gt;&quot;;
                    if((count($grouparray)&gt;0)&amp;&amp;in_array($_GET[&quot;where$i&quot;],$grouparray)){
                        print &quot;&lt;font color=red&gt;&quot;;
                    }
                    print ucfirst($f[$_GET[&quot;where$i&quot;]]);
                    if((count($grouparray)&gt;0)&amp;&amp;in_array($_GET[&quot;where$i&quot;],$grouparray)){
                        print &quot;&lt;/font&gt;&quot;;
                    }
                    print &quot;&lt;/th&gt;&quot;;
                }
            }
            if($groupby != '') { print &quot;&lt;th scope='col'&gt;&lt;font color=red&gt;&quot;.__('Count','statpresscn').&quot;&lt;/font&gt;&lt;/th&gt;&quot;; }
            print &quot;&lt;/tr&gt;&lt;/thead&gt;&lt;tbody id='the-list'&gt;&quot;;
            $qry=$wpdb-&gt;get_results($sql,ARRAY_N);
            $cloumnscount = count($wpdb-&gt;get_col_info(&quot;name&quot;));
            foreach ($qry as $rk) {
                print &quot;&lt;tr&gt;&quot;;
                for($i=1;$i&lt;=$cloumnscount;$i++) {
                    print &quot;&lt;td&gt;&quot;;
                    if($_GET[&quot;where$i&quot;] == 'urlrequested') { 
                        print &quot;&lt;a href=&quot;.heart5_config_url($rk[$i-1]).&quot; target=_heart5&gt;&quot;;
                        print iri_StatPress_Decode($rk[$i-1]);
                        print &quot;&lt;/a&gt;&quot;;
                    } else {
                        print $rk[$i-1];
                    }
//                    print $rk[$i-1];
                    print &quot;&lt;/td&gt;&quot;;
                }
                print &quot;&lt;/tr&gt;&quot;;
            }
            print &quot;&lt;/table&gt;&quot;;
            print &quot;&lt;br /&gt;&lt;br /&gt;&lt;font size=1 color=gray&gt;sql: $sql&lt;/font&gt;&quot;;
        }?&gt;
&lt;/div&gt;
[/sourcecode] 
