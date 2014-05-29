---
layout: post
title: Noble - Cross Site Scripting
tags:
- All
- attacker
- bugs
- cross site scripting
- Cross-Site Scripting (XSS)
- Encode
- PHP
- php echo
- Solution
- vulnerabilities
- Wordpress
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_tweeted: '1'
  _sexybookmarks_shortUrl: http://bit.ly/ah7y9S
  _sexybookmarks_permaHash: 5cd00fb0a086c660713c3692e8861999
---
## Details
__Affected Software:__ WordPress (core)

__Fixed in Version:__  1.2

__Issue Type:__ XSS

Original Code: <a title="Noble" href="http://spotthevuln.com/2010/09/noble/" target="_blank">Found    Here</a>
## Description
Looking at this code made me smile, its about 6 years old. There's a lot going on here and quite a few issues. The first thing that jumped out at me was the use of $HTTP_POST_FILES. $HTTP_POST_FILES means were working with user controlled files. There are tons of things that can go wrong when dealing with user/attacker controlled files (a list too long to go into here). Lets hope the WordPress devs are on their A-game here. Looking at the patch submitted by the WordPress developers, we see that they changed references to $HTTP_POST_FILES to the superglobal $_FILES. Within the $_FILES array there are a couple indexes that are commonly used. These indexes are:
<blockquote>[name]
[type]
[tmp_name]
[error]
[size]</blockquote>
Name, type, and size are all controlled by the user/attacker, so the WordPress developers should be wary when dealing with these values. Surprisingly (or unsurprisingly, depending on your point of view), this patch doesn't contain seem to contain any robust validation of data associated with the uploaded file data. Instead, the defenses put in place here seem to be centered around replacing a poor validation/sanitization routine with a more robust encoding routine which prevents a XSS vulnerability. The replaced sanitization routine and the XSS bug are presented in the following lines:
<blockquote><span style="color: #ff0000;">-$imgdesc = str_replace('"', '&amp;amp;quot;', $_POST['imgdesc']);</span><span style="color: #00ff00;">
+$imgdesc = htmlentities2($imgdesc);</span>
class="uploadform" value="&lt;?php echo $imgdesc;?&gt;" /&gt;</blockquote>
What's surprising is although the WordPress developers prevented this single XSS vulnerability, there is a large number of XSS vulnerabilities in this file. The most obvious symptom is "&lt;?php echo $_REQUEST[] ?&gt;. Additionally, the loose validation of the user uploaded file is concerning, especially with the number of problems that can be encountered when dealing with user controlled files. Maybe the varsity team was on vacation when this patch went in.
## Developers Solution
```diff
&lt;?php //Makes sure they choose a file

//print_r($HTTP_POST_FILES);
//die();


- $imgalt = (isset($_POST['imgalt'])) ? $_POST['imgalt'] : $imgalt;
-
- $img1_name = (strlen($imgalt)) ? $_POST['imgalt'] : $HTTP_POST_FILES['img1']['name'];
- $img1_type = (strlen($imgalt)) ? $_POST['img1_type'] : $HTTP_POST_FILES['img1']['type'];
- $imgdesc = str_replace('"', '&amp;amp;quot;', $_POST['imgdesc']);
+$imgalt = basename( (isset($_POST['imgalt'])) ? $_POST['imgalt'] : '' );
+
+$img1_name = (strlen($imgalt)) ? $imgalt : basename( $_FILES['img1']['name'] );
+$img1_type = (strlen($imgalt)) ? $_POST['img1_type'] : $_FILES['img1']['type'];
+$imgdesc = htmlentities2($imgdesc);

$imgtype = explode(".",$img1_name);
$imgtype = strtolower($imgtype[count($imgtype)-1]);

if (in_array($imgtype, $allowed_types) == false) {
die(sprintf(__('File %1$s of type %2$s is not allowed.') , $img1_name, $imgtype));
}

if (strlen($imgalt)) {
$pathtofile = get_settings('fileupload_realpath')."/".$imgalt;
-$img1 = $_POST['img1'];
+$img1 = $_POST['img1']['tmp_name'];
} else {
$pathtofile = get_settings('fileupload_realpath')."/".$img1_name;
-$img1 = $HTTP_POST_FILES['img1']['tmp_name'];
+$img1 = $_FILES['img1']['tmp_name'];
}

// makes sure not to upload duplicates, rename duplicates
$i = 1;
$pathtofile2 = $pathtofile;
$tmppathtofile = $pathtofile2;
$img2_name = $img1_name;

while (file_exists($pathtofile2)) {
$pos = strpos($tmppathtofile, '.'.trim($imgtype));
$pathtofile_start = substr($tmppathtofile, 0, $pos);
$pathtofile2 = $pathtofile_start.'_'.zeroise($i++, 2).'.'.trim($imgtype);
$img2_name = explode('/', $pathtofile2);
$img2_name = $img2_name[count($img2_name)-1];
}

if (file_exists($pathtofile) &amp;&amp; !strlen($imgalt)) {
$i = explode(' ', get_settings('fileupload_allowedtypes'));
$i = implode(', ',array_slice($i, 1, count($i)-2));
$moved = move_uploaded_file($img1, $pathtofile2);
// if move_uploaded_file() fails, try copy()
if (!$moved) {
$moved = copy($img1, $pathtofile2);
}
if (!$moved) {
die(sprintf(__("Couldn't upload your file to %s."), $pathtofile2));
} else {
chmod($pathtofile2, 0666);
@unlink($img1);
}

//

// duplicate-renaming function contributed by Gary Lawrence Murphy
?&gt;
&lt;p&gt;&lt;strong&gt;&lt;?php __('Duplicate File?') ?&gt;&lt;/strong&gt;&lt;/p&gt;
&lt;p&gt;&lt;b&gt;&lt;em&gt;&lt;?php printf(__("The filename '%s' already exists!"), $img1_name); ?&gt;&lt;/em&gt;&lt;/b&gt;&lt;/p&gt;
&lt;p&gt; &lt;?php printf(__("Filename '%1\$s' moved to '%2\$s'"), $img1, "$pathtofile2 - $img2_name") ?&gt;&lt;/p&gt;
&lt;p&gt;&lt;?php _e('Confirm or rename:') ?&gt;&lt;/p&gt;
&lt;form action="upload.php" method="post" enctype="multipart/form-data"&gt;
&lt;input type="hidden" name="MAX_FILE_SIZE" value="&lt;?php echo  get_settings('fileupload_maxk') *1024 ?&gt;" /&gt;
&lt;input type="hidden" name="img1_type" value="&lt;?php echo $img1_type;?&gt;" /&gt;
&lt;input type="hidden" name="img1_name" value="&lt;?php echo $img2_name;?&gt;" /&gt;
&lt;input type="hidden" name="img1_size" value="&lt;?php echo $img1_size;?&gt;" /&gt;
&lt;input type="hidden" name="img1" value="&lt;?php echo $pathtofile2;?&gt;" /&gt;
&lt;input type="hidden" name="thumbsize" value="&lt;?php echo $_REQUEST['thumbsize'];?&gt;" /&gt;
&lt;input type="hidden" name="imgthumbsizecustom" value="&lt;?php echo $_REQUEST['imgthumbsizecustom'];?&gt;" /&gt;
&lt;?php _e('Alternate name:') ?&gt;&lt;br /&gt;&lt;input type="text" name="imgalt" size="30" value="&lt;?php echo $img2_name;?&gt;" /&gt;&lt;br /&gt;
&lt;br /&gt;
&lt;?php _e('Description:') ?&gt;&lt;br /&gt;&lt;input type="text" name="imgdesc" size="30" value="&lt;?php echo $imgdesc;?&gt;" /&gt;
&lt;br /&gt;
&lt;input type="submit" name="submit" value="&lt;?php _e('Rename') ?&gt;" /&gt;
&lt;/form&gt;
&lt;/div&gt;
```
