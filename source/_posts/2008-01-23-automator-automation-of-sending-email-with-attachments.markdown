---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2008-01-23 17:39:45'
layout: post
slug: automator-automation-of-sending-email-with-attachments
status: publish
title: Automator - Automation of sending email with attachments
wordpress_id: '226'
? ''
: - apple
  - productivity
---

Today I had my first experience creating a VERY simple workflow with Automator.

<img id="image179" src="http://hamishrickerby.com/wp-content/uploads/2008/01/icalbirthdaysleopard_20071126175350-thumb.jpg" alt="Automator" />

I was editing word documents, excels and powerpoints on my mac and then sending them back to my work PC so that I could embed visio diagrams in them, as well as image files I have on that machine.

I was emailing lots and lots of files, and doing it over and over again, so I thought there must be a better way to do this.  Even Windows has a "Send To" option in Explorer, so the mighty OSX 10.4 should be able to do that too.

So, to cut a long story short I clicked on Finder in the applications pane, then selected "Get Selected Finder Items" as the action.  That was dragged over to the workflow area.

<img id="image181" src="http://hamishrickerby.com/wp-content/uploads/2008/01/step1.png" alt="Step 1" />

Then, I checked out the actions in Mail.  This was a bit confusing, because initially I tried to run "New Mail Message", and then "Add Attachments to Front Message".  This second step is unnecessary.  First of all "New Mail Message" returns "Mail Messages" and can't be used as input for "Add Attachments to Front Message", and secondly files/folders are input to "New Mail Message" and they are attached automagically.  I then selected my details by clicking on the Address Book icon, and set a subject "Files from home".

<img id="image182" src="http://hamishrickerby.com/wp-content/uploads/2008/01/step2.png" alt="step 2" />

Third step was to send the message.  The action is called "Send Outgoing Messages".  This was dragged over, and then I saved the action as "Send Files to Work".

<img id="image183" src="http://hamishrickerby.com/wp-content/uploads/2008/01/step3.png" alt="step 3" />

Now I have a shiny new menu item under automator.  Basically any file(s) or folders I want to email to work, I now just right click, go to automator, and click on Send Files to Work.  And off they go!

I will have to play with Automator a little more I think.  I've had my mac over 2 years now, and this is the first I've used it.