---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2005-12-17 04:13:00'
layout: post
slug: money-ruby-on-rails-and-aud
status: publish
title: Money, Ruby on Rails and AUD
wordpress_id: '198'
? ''
: - ruby
---

I had a bit of a play with the Money and Paypal GEMs available at <a href="http://dist.leetsoft.com">http://dist.leetsoft.com</a> .  The Money API only supports Euro, US and Canadian dollars.  For my paypal integration, I need to support Australian dollar (as dirty old paypal don't support NZD).  So, I've extended the Money class to create an AUD implementation.  There's nothing special about it, but if you're too lazy to write the couple of lines of code yourself, <a href="http://homepages.ihug.co.nz/~amorph/custom78_money.rb">here it is</a>.

Also, I've been having a bit of trouble getting the Money and Paypal GEMs to work.  The issue is that from within a rails controller it doesn't recognise the Money or Paypal classes.  It can't find them.  This is the only GEM that I'm having troubles with.  My solution was to import the .rb files into the /vendor directory.  If anyone has this problem, or has a solution for this problem - or even tips, please let me know.  It's annoying me now.

If I can get this working, I won't bother with my own IPN implementation.  Why reinvent the wheel?