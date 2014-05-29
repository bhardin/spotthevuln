---
layout: post
title: Spinning - Defense in Depth
tags:
- ActionScript
- All
- All Software
- application languages
- Code Snippet
- Defense In Depth
- DoJo
- framework level
- information disclosure
- internal functionality
- security flaws
- source code
- stefano di paola
- swf file
- swf files
- validation code
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _sexybookmarks_shortUrl: http://bit.ly/dpi4WG
  _edit_last: '2'
  _aktt_hash_meta: ''
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 5ebf6184c9cb5c8c32104394278af526
---
## Details
__Affected Software:__ Dojo Toolkit

__Fixed in Version:__  1.4.1

__Issue Type:__ Defense in Depth

Original Code: <a title="Spinning" href="http://spotthevuln.com/2010/08/spinning/" target="_blank">Found    Here</a>
## Description
This was a vulnerability affecting the Dojo toolkit. Apparently, the dojo toolkit shipped with a SWF file that had a few vulnerabilities. This particular vulnerability affected one of those SWF files. First, SWF files are compiled files, however they can be decompiled. Unlike traditional server side web application languages (PHP, ASP, JSP...etc), SWF files are downloaded and rendered on the clientside. Decompiling the SWF file gives the attacker full access to the ActionScript source code for the SWF application.

In this particular SWF file, we see that the developers explicitly set the Security.allowDomain to "*". This makes it so SWF flies from other, external domains can include the Dojo toolkit SWF file and script/access its internal functionality.

The Dojo toolkit devs fixed this particular issue by removing the allowDomain call and adding an Externalinterface call checking to see if a particular wrapper was available in HTML. If you're interested in Flash security, an excellent presentation on Flash security given by Stefano Di Paola can be found here:

<a href="http://www.slideshare.net/guestb0af15/owasp-wasc-app-sec2007-san-jose-finding-vulnsin-flash-apps">http://www.slideshare.net/guestb0af15/owasp-wasc-app-sec2007-san-jose-finding-vulnsin-flash-apps</a>
## Developers Solution
[cce lang="diff"]

public class FLVideo extends Sprite {
private var videoUrl:String;
private var video:Video;
private var connection:NetConnection;
private var autoPlay:Boolean;
private var videoStream:NetStream;
private var videoWidth:Number;
private var videoHeight:Number;
private var _currentVideo:VideoContainer;
private var preview:VideoContainer;
private var currentVolume:Number = 1;
private var isFullscreen:Boolean = false;
private var playlist:VideoPlaylist;
private var hasPlaylist:Boolean = false;
private var mode:String = "preview";

public function FLVideo() {
-    Security.allowDomain("*");
+    var secure:* = ExternalInterface.call("swfIsInHTML");
+    if(secure !== true){
+        return;
+    }
+    //Security.allowDomain("*");

stage.scaleMode = StageScaleMode.NO_SCALE;
stage.align = StageAlign.TOP_LEFT;
stage.addEventListener(Event.RESIZE, onStageResize);
stage.addEventListener(FullScreenEvent.FULL_SCREEN , onFullscreenChange);
stage.addEventListener(MouseEvent.CLICK, onClick);

var obj:Object = LoaderInfo(this.root.loaderInfo).parameters;
trace(obj)

if(!obj.videoUrl){
obj = {
autoPlay: true,
isDebug: true,
videoUrl:"demo_video.flv"
};
}


// ugh - booleans not coming through
if(obj.autoPlay===true || obj.autoPlay=="true"){
autoPlay = true;
}

if(obj.volume) {
currentVolume = obj.volume;
}

if(obj.isDebug===true || obj.isDebug=="true"){
console.isDebug(true);
Tracer.init({both:true})
Tracer.log("FLVideo initialized...")
}

+    Tracer.log("secure?::", secure)
MovieIdentity.identity = obj.id || "default";
this.playlist = new VideoPlaylist(autoPlay, currentVolume);

if(obj.videoUrl) {
videoUrl = obj.videoUrl;
}

preview = new VideoContainer(videoUrl, autoPlay, currentVolume);
addChild(preview);
provideCallbacks();
}


public function get currentVideo():VideoContainer{

if(mode=="playlist" &amp;&amp; hasPlaylist){
return this.playlist.current;
}else{
return preview;
}
}
[/cce]
