---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2009-09-20 16:58:33'
layout: post
slug: iphone-camera-overlays-iphonearkit
status: publish
title: iPhone Camera Overlays + iphonearkit
wordpress_id: '424'
? ''
: - apple
  - augmented reality
  - iphone
  - mobile development
---

Today I have been hacking away on the <a href="http://github.com/zac/iphonearkit" target="_self">iphonearkit source</a> available at github and have incorporated the ARGeoViewController as an overlay over a ImagePickerController with the camera as the source (which was introduced with iPhone OS 3.1).  Results below.

<p><img class="size-medium wp-image-425" title="ARGeoViewController overlay" src="http://hamishrickerby.com/wp-content/uploads/2009/09/IMG_0337-300x200.PNG" alt="ARGeoViewController as the overlay on a ImagePickerController on iPhone" width="300" height="200" /></p>

I want to tidy up some of the code before I check it back into <a href="http://github.com/rickerbh/iphonearkit" target="_blank">my clone of the source</a>, but this is a really good basis for some smart location and direction aware augmented reality apps on iPhone.  Wonder what the iphonearkit license is - it's unclear...

--- EDIT ---

Bugger.  It appears that <a href="http://github.com/zac/" target="_blank">zac</a> has implemented similar functionality to me already :-(  Bloody github and it's slow (never!) updates to fork queues and network graphs.  Oh well, maybe I won't bother tidying my code.