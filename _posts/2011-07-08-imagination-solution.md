---
layout: post
title: Imagination - XSS and XSRF
tags:
- Cross Site Request Forgery (XSRF)
- Cross-Site Scripting (XSS)
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
## Details
__Affected Software:__ Zeus C&C

__Fixed in Version:__  ?

__Issue Type:__ XSS and XSRF

Original Code: <a href="http://spotthevuln.com/2011/07/imagination/">Found Here</a>
## Details
This week's bugs affected Zeus C&C 1.1.0.0. The file we're looking at is mod.bcmds.php. The first thing that popped out at me was the named constant "QUERY_STRING" that's being used in various places in code. Although we don't get to see exactly where QUERY_STRING is being defined in the code snippet as a general rule of thumb, values from the query string cannot be trusted. In this case, QUERY_STRING is defined in a different file (in.php) in the following line:
<code lang="PHP">
define('QUERY_STRING', QUERY_STRING_BLANK.$module);
</code>
QUERY_STRING_BLANK is defined in the following way (also in in.php):
<code lang="PHP">
define('QUERY_STRING_BLANK', $_SERVER['PHP_SELF'].'?m=');
</code>
Veteran Spot the Vuln readers will immediately realize that $_SERVER['PHP_SELF'] cannot be trusted and can contain attacker supplied data. An old, but good write-up on PHP_SELF XSS can be found <a href="http://seancoates.com/blogs/xss-woes" target="_blank">here</a>.
<br>
Knowing this, we're free to XSS the Zeus C&C and hijack the bots... as long as we can get the Zeus botmaster to visit a page we own (a reasonable request) AND we can figure out the domain name the botmaster is using for their C&C (fairly difficult). Botmasters can take advantage of browser same origin policy defenses and use a host file to create a unique domain for their C&Cs... minimizing the impact of reflected XSS exploits against their C&Cs. I'm wondering if this is the first public security advice for the botmaster community...
<br>
I've highlighted the lines that insecurely use the QUERYSTRING constant to build HTML markup, resulting in XSS. I couldn't find a mod.bcmds.php file after Zeus 1.1.0.0, so I'm considering this specific XSS issue fixed.
<br>
There is a second, more subtle issue in this code... one that still affects the latest Zeus C&C builds. The C&C developer seemingly went through great lengths to defend against SQL injection. A quick perusal through the code shows a smattering of addslashes() and is_numeric() in attempts to validate input before passing it to backend databases. What's missing however... are nonce/token checks (XSRF defenses). The following code snippet is a perfect example:
<code lang="PHP">
else if(isset($_GET['del'])&&is_numeric($_GET['del'])&&$pedt)
{
  mysql_query('DELETE FROM  '.TABLE_BCMDS.' WHERE id='.$_GET['del'].' LIMIT 1');
  header('Location: '.QUERY_STRING);
  die();
}
</code>
In the snippet above, we see that the C&C code grabs a value directly from the querystring, validates that it is_numeric(), and then passes the value to a DELETE statement. No where does the code attempt to validate that the request wasn't generated via XSRF. If an attacker can discover the location of the C&C and lure the botmaster to an attacker controlled page, they can setup an XSRF attack to delete the entire TABLE_BCMDS. Looking through the latest, most current Zeus C&C code, XSRF defenses still have not been put into place... come on guys, even WordPress has XSRF defenses!  <a href="http://codex.wordpress.org/Function_Reference/wp_verify_nonce" target="_blank">http://codex.wordpress.org/Function_Reference/wp_verify_nonce</a>

## Vulnerable Code
<code lang="PHP" highlight="50,55,87,101">
<?php if(!defined('__INDEX__'))die();
$pedt=PRIV&PRIV_BOTS_CMDS_EDIT;
if((isset($_GET['new'])&&$pedt)||(isset($_GET['edit'])&&is_numeric($_GET['edit'])))
{
  if(!@include_once('fmt.php'))die('fmt.php not founded!');
  $name=isset($_POST['name'])?$_POST['name']:time();
  $stat=isset($_POST['stat'])?($_POST['stat']?1:0):0;
  $limit=(isset($_POST['limit'])&&is_numeric($_POST['limit']))?$_POST['limit']:0;
  $cnts=isset($_POST['cnts'])?$_POST['cnts']:'';
  $cids=isset($_POST['cids'])?$_POST['cids']:'';
  $bns=isset($_POST['bns'])?$_POST['bns']:'';
  $cmds=isset($_POST['cmds'])?$_POST['cmds']:'';

  if($_SERVER['REQUEST_METHOD']=='POST'&&strlen($name)>0&&$pedt)
  {
    $cmdsb=EncodeBuffer(str_replace("\r\n","\n",trim($cmds)));
    $data='name=\''.addslashes($name).'\',stat='.$stat.',lim='.$limit.',c=\''.addslashes(SepFmt($cnts)).'\',comps=\''.addslashes(SepFmt($cids)).'\',bns=\''.addslashes(SepFmt($bns)).'\',cmds=\''.addslashes($cmdsb).'\'';
    if(isset($_GET['new']))mysql_query('INSERT INTO '.TABLE_BCMDS.' SET '.$data.',id2='.time());
    else mysql_query('UPDATE '.TABLE_BCMDS.' SET '.$data.' WHERE id=\''.$_GET['edit'].'\' LIMIT 1');
    header('Location: '.QUERY_STRING);
  }
  else
  {
    if(!$pedt&&isset($_GET['new']))unset($_GET['new']);
    HTMLBegin(isset($_GET['new'])?LNG_MBCMDS_NEWCMD:($pedt?LNG_MBCMDS_EDITCMD:LNG_MBCMDS_VIEWCMD));
    if(isset($_GET['new']))print CmdForm('new',LNG_MBCMDS_NEWCMD,LNG_MBCMDS_ADD,$name,$stat,$limit,$cnts,$cids,$bns,$cmds);
    else
    {
      $r=mysql_query('SELECT * FROM '.TABLE_BCMDS.' WHERE id=\''.$_GET['edit'].'\' LIMIT 1');
      if($r&&mysql_affected_rows()==1&&($m=mysql_fetch_assoc($r)))print CmdForm('edit='.$_GET['edit'],$pedt?LNG_MBCMDS_EDITCMD:LNG_MBCMDS_VIEWCMD,$pedt?LNG_MBCMDS_EDIT:'',$m['name'],$m['stat'],$m['lim'],SepFmtB($m['c']),SepFmtB($m['comps']),SepFmtB($m['bns']),DecodeBuffer($m['cmds']));
      else print '<font class="error">'.LNG_MBCMDS_ERROR_1.'</font>';
    }
    HTMLEnd();
  }
  die();
}
else if(isset($_GET['del'])&&is_numeric($_GET['del'])&&$pedt)
{
  mysql_query('DELETE FROM  '.TABLE_BCMDS.' WHERE id='.$_GET['del'].' LIMIT 1');
  header('Location: '.QUERY_STRING);
  die();
}
else if(isset($_GET['res'])&&is_numeric($_GET['res'])&&$pedt)
{
  mysql_query('UPDATE '.TABLE_BCMDS.' SET exc=\'0\',rcomps=\'\',exct=\'0\' WHERE id='.$_GET['res'].' LIMIT 1');
  header('Location: '.QUERY_STRING);
  die();
}

HTMLBegin(LNG_MBCMDS,$pedt?'function DelCmd(uid,q){if(confirm(q))window.location=\''.QUERY_STRING.'&del=\'+uid;};function ResCmd(uid,q){if(confirm(q))window.location=\''.QUERY_STRING.'&res=\'+uid;}':'');

$r=mysql_query('SELECT * FROM '.TABLE_BCMDS);
$total=mysql_affected_rows();
print '<table class="tbl1"><tr><td class="td1" colspan="'.($pedt?9:10).'">'.LNG_MBCMDS_R_CMDS.'&nbsp;('.$total.')</td>';
if($pedt)print '<td class="td1" align="center"><input type="submit" value="'.LNG_MBCMDS_NEWCMD.'" class="ism" style="width:100%" onClick="window.location=\''.QUERY_STRING.'&new\';"></td>';
print '</tr><tr><td class="td1">'.LNG_MBCMDS_R_ID.'</td><td class="td1">'.LNG_MBCMDS_R_NAME.'</td><td class="td1">'.LNG_MBCMDS_R_STAT.'</td><td class="td1">'.LNG_MBCMDS_R_LIMIT.'</td><td class="td1">'.LNG_MBCMDS_R_REQ.'</td><td class="td1">'.LNG_MBCMDS_R_EXEC.'</td><td class="td1">'.LNG_MBCMDS_R_CNTS.'</td><td class="td1">'.LNG_MBCMDS_R_CIDS.'</td><td class="td1">'.LNG_MBCMDS_R_BNS.'</td><td class="td1">&nbsp;</td></tr>';
if($total>0)
{
  $j=0;
  while(($m=mysql_fetch_assoc($r)))
  {
    $a=(($j++)%2==0?1:2);
    print '<tr valign="top"><td align="right" class="tdx'.$a.'">'.$m['id2'].'</td>'.
          '<td class="tdx'.$a.'">'.htmlentities($m['name']).'</td>'.
          '<td class="tdx'.$a.'">'.($m['stat']?LNG_MBCMDS_STAT_ON:LNG_MBCMDS_STAT_OFF).'</td>'.
          '<td align="right" class="tdx'.$a.'">'.$m['lim'].'</td>'.
          '<td align="right" class="tdx'.$a.'">'.$m['exc'].'</td>'.
          '<td align="right" class="tdx'.$a.'">'.$m['exct'].'</td>'.
          '<td class="tdx'.$a.'">'.($m['c']==''?'-':str_replace(',','<br>',htmlentities(SepFmtB($m['c'])))).'</td>'.
          '<td class="tdx'.$a.'">'.($m['comps']==''?'-':str_replace(',','<br>',htmlentities(SepFmtB($m['comps'])))).'</td>'.
          '<td class="tdx'.$a.'">'.($m['bns']==''?'-':str_replace(',','<br>',htmlentities(SepFmtB($m['bns'])))).'</td>'.
          '<td class="tdx'.$a.'" align="center"><input class="ism" style="width:90%" type="submit" value="'.($pedt?LNG_MBCMDS_R_EDIT:LNG_MBCMDS_R_VIEW).'" onClick="window.location=\''.QUERY_STRING.'&edit='.$m['id'].'\';return false;">';
    if($pedt)print '<br><input class="ism" style="width:90%" type="submit" value="'.LNG_MBCMDS_R_RES_OK.'" onClick="javascript:ResCmd(\''.$m['id'].'\',\''.addslashes(sprintf(LNG_MBCMDS_R_RES,$m['name'])).'\');return false;"><br><input class="ism" style="width:90%" type="submit" value="'.LNG_MBCMDS_R_DEL_OK.'" onClick="javascript:DelCmd(\''.$m['id'].'\',\''.addslashes(sprintf(LNG_MBCMDS_R_DEL,$m['name'])).'\');return false;">';
    print '</td></tr>';
  }
}
else print '<tr><td align="center" colspan="10" class="tdx1"><i>'.LNG_MBCMDS_R_NONE.'</i></td></tr>';
print '</table>';
HTMLEnd();

function CmdForm($cmd,$title,$action,$name,$stat,$limit,$cnts,$cids,$bns,$cmds)
{
  $en=$action==''?0:1;
  $stat=$stat?1:0;
  $ro=$en?'':'readonly ';

  $str=$en?'<form method="POST" action="'.QUERY_STRING.'&'.$cmd.'">':'';
  $str.='<table class="tbl1" width="350"><tr><td class="td1" colspan="2">'.$title.'</td></tr>'.
        '<tr><td>'.LNG_MBCMDS_NAME.'</td><td width="100%"><input '.$ro.'type="text" name="name" value="'.htmlentities($name).'" style="width:100%"></td></tr>'.
        '<tr><td colspan="2"><table class="tbl1"><tr><td>'.LNG_MBCMDS_STAT.'</td><td width="100%"><select '.($en?'':'disabled ').'name="stat" style="width:100%">'.
        '<option value="1"'.($stat==1?' selected':'').'>'.LNG_MBCMDS_STAT_ON.'</option>'.
        '<option value="0"'.($stat==0?' selected':'').'>'.LNG_MBCMDS_STAT_OFF.'</option>'.
        '</select></td></tr>'.
        '<tr><td>'.LNG_MBCMDS_LIMIT.'</td><td width="100%"><input '.$ro.'type="text" name="limit" value="'.$limit.'" style="width:100%"></td></tr>'.
        '<tr><td>'.LNG_MBCMDS_CNTS.'</td><td width="100%"><input '.$ro.'type="text" name="cnts" value="'.$cnts.'" style="width:100%"></td></tr>'.
        '<tr><td>'.LNG_MBCMDS_CIDS.'</td><td width="100%"><input '.$ro.'type="text" name="cids" value="'.$cids.'" style="width:100%"></td></tr>'.
        '<tr><td>'.LNG_MBCMDS_BNS.'</td><td width="100%"><input '.$ro.'type="text" name="bns" value="'.$bns.'" style="width:100%"></td></tr>'.
        '<tr><td valign="top">'.LNG_MBCMDS_CMDS.'</td><td><textarea wrap="off" '.$ro.'name="cmds" style="width:100%;height:100">'.htmlentities($cmds).'</textarea></td></tr>'.
        '</table></tr></td><tr><td colspan="2" align="right">';
  if($en)$str.='<input type="submit" class="ism" value="'.$action.'" style="width:100">&nbsp;';
  $str.='<input type="submit" class="ism" value="'.LNG_MBCMDS_BACK.'" style="width:100" onClick="window.location.href=\''.QUERY_STRING.'\';return false;"></td></tr>';
  if($en)$str.='</form>';
  return $str.'</table>';
}
function SepFmt($str){if(strlen($str)>1){$str=str_replace(',','|',trim($str));if($str[0]!='|')$str='|'.$str;if($str[strlen($str)-1]!='|')$str.='|';}return $str;}
function SepFmtB($str){if(strlen($str)>1){$str=str_replace('|',',',trim($str));if($str[0]==',')$str=substr($str,1);$l=strlen($str);if($str[$l-1]==',')$str=substr($str,0,$l-1);}return $str;}
?>
</code>
