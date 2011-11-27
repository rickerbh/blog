---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2009-02-01 08:59:05'
layout: post
slug: cocos2d-iphone-blocking-touch-events
status: publish
title: cocos2d-iphone blocking touch events
wordpress_id: '285'
? ''
: - apple
  - iphone
  - mobile development
---

The game development is going well so far.  From yesterday when I knew NOTHING about game development I've managed to figure out how to use sprites, labels, timed actions, sequenced actions, scenes and layers with cocos2d-iphone.

I did hit a strange error though.  On the simulator the touch events were working correctly - screens transitioning, sprite/label touches generating events and triggering animation and so on.  On the actual device the story was quite different.  My menu items were triggering correctly, but no subsequent touch events were doing anything on the phone.  When the application exited (home button) all the touch events were then passed through as I could see them all fly up the screen in the console.

A long search in google yielded the following page:  <a href="http://groups.google.com/group/cocos2d-iphone-discuss/browse_thread/thread/8aae440d81721ff4" target="_blank">http://groups.google.com/group/cocos2d-iphone-discuss/browse_thread/thread/8aae440d81721ff4</a>

I don't know what causes the actual issue, but the fix is to alter the Director.m's main method - adding the following code fixed the issue right up.

``` objective-c
while (CFRunLoopRunInMode(kCFRunLoopDefaultMode, 0, YES) ==  kCFRunLoopRunHandledSource) {}; 
```

The odd thing is (and let me know if I'm wrong here) that that code doesn't actually do anything.  It performs a test, but changes nothing.  I don't get why this fix works.

?