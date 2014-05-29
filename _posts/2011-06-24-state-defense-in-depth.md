---
layout: post
title: State - Defense in Depth
tags:
- Defense In Depth
- PHP
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ! '#secure #code #dev'
  _edit_last: '2'
  aktt_tweeted: '1'
---
## Details
__Affected Software:__ Adrenalin C&C

__Fixed in Version:__  Not Patched

__Issue Type:__ Defense in Depth

Original Code: <a href="http://spotthevuln.com/2011/06/stat/">Found Here</a>
## Details
First, I'll talk about a couple of interesting things about this bug that cannot be seen from just the code sample. When I received this sample, it was encoded with Zend Guard. While the Zend Guard encoding was easily defeated, it is interesting to see that these malware authors are interested in protecting their intellectual property. Once again, the malware industry doesn't get a magical free pass on all the things traditional development shops face. Monetizing, feature requests, protecting IP, and even security problems are issues all dev shops face.
<br/>
After the code was decoded, it was quickly apparent that this file contained several routines for dealing with uploding files to the web C&C. I pulled out a routine that I thought was particularly interesting for this week's code sample. The sample takes several variables from user/attacker controlled parameters (lines 6, 8, and 9). One of these variables ($logfolder) is passed directly to fopen(). Fopen is an interesting API. In this code sample, fopen() is intended to open a file from the local filesystem. There is no directory traversal check for $logfolder, so the attacker is free to pass a simple ../../../ in the $logfolder variable and control where the txt file gets written to. In addition to directory traversal bugs, fopen() can actually open more than just local files. fopen()supports a number of schemes such as: http://, ftp://, php://, ssh2://, and several others. A full list of protocols supported by fopen() can be found here:  http://www.php.net/manual/en/wrappers.php. Because the $logfolder variable is the first variable passed to fopen(), the attacker can supply any of these protocols to fopen(). Using these protocols, the attacker can cause the C&C to make arbitrary requests to external servers. Full compromise of theC&C web server would be difficult, but information leakage can definitely be accomplished.
<br/>
## Vulnerable Code
<code lang="PHP" highlight="14">
<?php
...snip...
function srch( )
{
    set_time_limit( 0 );
    $word = $_REQUEST['word'];
    $word2 = $word;
    $logfolder = $_REQUEST['infile'];
    $arch = $_REQUEST['xxx'];
    if ( $word != "" )
    {
        $word = explode( "\r\n", $word );
        $wordc = count( $word );
        $hl9 = fopen( $logfolder."/.out.txt", "w" );
        fclose( $hl9 );
        $dir = opendir( $logfolder );
        $finded = "";
        while ( $file = readdir( $dir ) )
        {
            if ( !( $file != "." ) || !( $file != ".." ) || !( $file != ".out.txt" ) || !( substr( $file, -4 ) == ".txt" ) )
            {
                $hl = fopen( $logfolder."/".$file, "rb" );
                $readsz = filesize( $logfolder."/".$file );
                if ( $readsz < 1041076 )
                {
                    $readszR = $readsz;
                }
                else
                {
                    $readszR = 1041076;
                    $readsz -= 1041076;
                }
                while ( $data = fread( $hl, $readszR ) )
                {
                    $pos = 0;
                    $posC = 0;
                    $posS = 0;
                    while ( $pos = strpos( $data, "[IP:", $pos ) )
                    {
                        $pos = strpos( $data, "]", $pos ) + 1;
                        if ( $pos < $posC )
                        {
                            break;
                        }
                        else
                        {
                            $posC = $pos;
                            $lent = $pos - $posS;
                            unset( $cutblock );
                            $cutblock = substr( $data, $posS, $lent );
                            $rd = 0;
                            for ( ; $rd < $wordc; ++$rd )
                            {
                            }
                            if ( !( $word[$rd] != "" ) || !( $ftmp = strpos( $cutblock, $word[$rd], 0 ) ) )
                            {
                                $hl9 = fopen( $logfolder."/.out.txt", "ab+" );
                                fwrite( $hl9, $cutblock );
                                fclose( $hl9 );
                            }
                        }
                        unset( $rd );
                        unset( $lent );
                        unset( $ftmp );
                        unset( $cutblock );
                        $posS = $pos;
                    }
                    unset( $data );
                    if ( $readsz < 1041076 )
                    {
                        $readszR = $readsz;
                    }
                    else
                    {
                        $readszR = 1041076;
                        $readsz -= 1041076;
                    }
                }
                unset( $data );
                fclose( $hl );
            }
        }
        if ( 0 < filesize( $logfolder."/.out.txt" ) )
        {
            $hl9 = fopen( $logfolder."/.out.txt", "r" );
            $finded = fread( $hl9, filesize( $logfolder."/.out.txt" ) );
            fclose( $hl9 );
            if ( $arch == 1 )
            {
                header( "Content-type: application/octet-stream" );
                $cl_Zip = new cl_zip( );
                $cl_Zip->onaddfile( $finded, "log".time( ).".txt" );
                header( "Content-Length: ".strlen( $cl_Zip->ondumpfileout( ) ) );
                header( "Content-disposition: attachment; filename=log".time( ).".zip" );
                echo $cl_Zip->ondumpfileout( );
                exit( );
            }
            return $finded;
        }
    }
}
...snip...
?>
</code>
