---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2008-12-13 08:50:32'
layout: post
slug: memory-management-in-objective-c
status: publish
title: Memory Management in Objective-C
wordpress_id: '271'
? ''
: - apple
  - mobile development
---

I was having some issues with Memory Management and dealloc'ing objects (and subsequently crashing software) and found the following resources very useful:
<ul>
	<li><a title="Memory Management Blog Entry on iphonedevelopertips.com" href="http://iPhoneDeveloperTips.com/objective-c/memory-management.html" target="_blank">Memory Management Blog Entry</a> (<a href="http://iphonedevelopertips.com" target="_blank">iphonedevelopertips.com</a>)</li>
	<li><a title="Memory Management Programming Guide for Cocoa PDF" href="http://developer.apple.com/documentation/Cocoa/Conceptual/MemoryMgmt/MemoryMgmt.pdf" target="_blank">Memory Management Programming Guide for Cocoa</a> (<a href="http://developer.apple.com" target="_blank">developer.apple.com</a>)</li>
</ul>
<div>To paraphrase</div>
<div>
<ol>
	<li>Never <code>dealloc</code>, use <code>release</code></li>
	<li>If you create an object it's your responsibility to release it</li>
	<li>If you want to own an object or use and object returned from another object you should <code>retain</code> it</li>
	<li>Don't release an object you didn't create or own</li>
</ol>
<div>Now I remember why I like programming languages that take memory management out of my hands. Last time I touched memory management was at university, and it was a pain then too.</div>
<div></div>
<div>Good thing is that Objective-C/Cocoa has those simple rules/conventions to follow.  Memory Management is not onerous on the mac/iPhone, so I can't complain too much.</div>
</div>