---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2009-07-31 11:51:04'
layout: post
slug: ussd-codes-on-iphone
status: publish
title: USSD Codes on iPhone
wordpress_id: '371'
? ''
: - apple
  - iphone
  - mobile development
  - telecommunications
---

makeuseof.com have recently posted about <a href="http://www.makeuseof.com/tag/cool-iphone-keypad-codes/">11 cool iPhone keypad codes</a> - these are special codes (known as <a href="http://en.wikipedia.org/wiki/Unstructured_Supplementary_Service_Data">USSD</a> codes) that send messages via the signalling channel direct to the core of a mobile operators network.  These codes are nothing new, they have been around for years and years.  They are also not generally universal (there are some standard, but they provide relatively boring functionality).  Different networks can enable different functionality on different codes.

The codes can do boring things like retrieve your divert status from the network, return your <a href="http://en.wikipedia.org/wiki/IMEI">IMEI</a> or perhaps <a href="http://en.wikipedia.org/wiki/ICCID">ICCID</a>, but they can also interact with specialised applications driven via <a href="http://en.wikipedia.org/wiki/USSD_Gateway">USSD Gateways</a> to return useful information and execute transactions.  These are applications that are sent specific codes by an operators core network, perform some processing on the data received, and return a response.  Things that are non-standard that are enabled by USSD Gateways are services such as USSD-based prepay balance retrieval, USSD-topup, or interactions with <a href="http://en.wikipedia.org/wiki/NGIN">NGIN</a> features to alter a network based service.

I looked into USSD codes on the iPhone a while back, not to be used by users typing them in, but more to be used by applications querying information from the network via them.  The reason why I wanted to programatically access them?  To look at what's possible for network operators or enterprises to release as iPhone based network service management applications.  

Sadly, Apple have disable the use of USSD codes from within the (legitimate) iPhone sandbox available to developers (via the open URL methods, passing in a tel://xxxxxx URL).  This means that there won't be any applications from your operators that will make it easy to retrieve and change network settings that can be released thru the app store - at least not until Apple change their mind about interactions with USSD codes.  Which is a pity - there are lots of useful services that would be useful to expose simple interfaces for usage for to the operators end users.  USSD is an efficient, fast way to configure the network, using capabilities that most operators already have. 