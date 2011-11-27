---
layout: post
title: "CASE and Requirements Management"
date: 2011-11-07 09:09
comments: true
categories: 
---
I've started the feature & function planning for my application - or really, I've started to identify the requirements for it. I'm starting by evaluating existing applications in the marketplace, considering the functions they support, the data they collect, and how the user navigates through the application. So far, I have about 300 screenshots from various applications,  strewn all over the floor of my office, and 4 coloured pens for me to mark up the screenshots. The colours refer to the following areas of requirements or commentary.

1. Brick red - bad UI
2. Blue - data to collect
3. Green - good UI
4. Pink - application function, feature, or derived data

The functions are further classified into "things that should be there for a first release" and "things that can wait until later".

What I'm looking for now is a sensible way to capture and manage all this information. Carrying around 300 pages of screenshots is just silly. I could have electronically marked up the Keynote presentation rather than do this on paper, but I don't see myself reusing the information if it's stored in that format.

As part of a contract my company has right now, I'm using a [CASE](http://en.wikipedia.org/wiki/Computer-aided_software_engineering) tool for a project with a client. The business analysts on this project are tracking all requirements, use cases and UI modules within a tool called Enterprise Architect by Sparx Systems. I'm currently leading the technical workstream and we are also using it to produce our technical application design. It seems like a very comprehensive tool, but it's Windows only. I'd love for a tool like that to be native on the Mac.

Other CASE and UML tools I've used in the past are:

- Popkin System Architect (became Telelogic, then IBM and consumed into the Rational suite)
- Rational Rose
- ArgoUML
- Troux Architect
- Various Eclipse plugins

The issue I have with all of these is that it seems they will be disconnected from the development workflow if you use a mac, or are working on a non-Java project. These tools are supposed to help with the development process, but I can't see how that will work if they aren't connected to it. Yes, I understand the value in the whole planning and design side, but the disconnect to development seems large. If I didn't care about having the process connected, I'd just use OmniOutliner as I have in the past for other personal projects - it can track requirements, feature/release planning, data model elements etc. But because of the potential size of this application, I feel I should be using something more structured.

Maybe the project should change to creating a decent CASE tool for the Mac :-)

At the moment the plan is to use Enterprise Architect, and we'll see how that goes. One good thing is that I can export data out of it if the process doesn't work out, so there will be little wasted effort.

If anyone has used CASE tools on the mac for Objective-C based projects, I'd love to hear what was used, and how the experience was - good or bad.