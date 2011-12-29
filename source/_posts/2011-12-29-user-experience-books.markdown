---
layout: post
title: "User Experience Books"
date: 2011-12-29 16:44
comments: true
categories: 
---
As I alluded to in a [previous post](/2011/10/24/work-plans/), things like User Experience, Human Computer Interaction, and Graphic Design are not my fortes. All my professional software experience until I started iOS development was either in developing (~2 years) or designing/architecting (9+ years) large scale telecommunications software systems (specifically [BSS](http://en.wikipedia.org/wiki/Business_support_system) and [OSS](http://en.wikipedia.org/wiki/Operations_support_system) systems if you're interested). These sorts of systems do not have pretty user interfaces - there is no need to provide them. In fact, the majority of systems I have designed only have human interactions with back-office operators who are Unix system administrators - the main "user" of the system would typically be another system. 

The users of my new software will be proper "end-users". People who do not use computers for a job. People who are unsure, or even wary about computers for fear of breaking them. People who do not know commands like `kill -9` when something isn't working, or even how to drive a CLI. 

It would be delinquent of me to not take some advice from well respected professionals in this space. So, I read some books. (_I will take some advice from actual people later, but as a first step I think education is always a good step._)

Designing Interactions by Bill Moggridge
----------------------------------------

[Amazon UK](http://www.amazon.co.uk/gp/product/0262134748?ie=UTF8&tag=hamishrickerb-21&linkCode=shr&camp=3194&creative=21330&creativeASIN=0262134748&ref_=sr_1_1&qid=1325142051&sr=8-1), [Amazon US](http://www.amazon.com/gp/product/0262134748/ref=as_li_tf_tl?ie=UTF8&tag=hamricsblo-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0262134748)

I actually got this book expecting it to have practical advice on how to design software products, and was (pleasantly) surprised when it didn't. It's a collection of interviews (and then analysis and commentary on the interviews) with a set of influential product designers. There is a strong bias towards IDEO, but I guess that's to do with the authors familiarity with the organisation. The book gives insight into the interaction design processes used to create some very popular products, as well as covering some things that haven't worked. It gives you a good idea of how the designers _think_ about problems - which is great to understand as an outsider to this world.

The two parts of this book that stick with me are Terry Winograd's discussion on Mark Weiser's _ubiquitous computing_ - the idea that people don't want to interact with computers - they want to get something done. The computer is an instrument to be used to get something done - the purpose is not to interact with the computer, but to achieve a goal. The other part was John Maeda's Laws of Simplicity. These eight laws are about making complex systems simpler for users, either by relating functions, using knowledge that users already have, and recognising when materials (technology in my case) is inappropriate for a task, either by it being too hard to use, or breaking "laws".

TOG on Interface by Bruce "TOG" Tognazzini
------------------------------------------

[Amazon UK](http://www.amazon.co.uk/gp/product/0201608421?ie=UTF8&tag=hamishrickerb-21&linkCode=shr&camp=3194&creative=21330&creativeASIN=0201608421&ref_=sr_1_1&qid=1325143254&sr=8-1), [Amazon US](http://www.amazon.com/gp/product/0201608421/ref=as_li_tf_tl?ie=UTF8&tag=hamricsblo-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0201608421)

I got this book because of the Apple Macintosh legacy the author has. Bruce Tognazzini developed the first set of Human Interface Guidelines for Apple in the late 70's, and was involved with the Macintosh UI which still has a significant influence on the modern interface we use on OSX. The book is a collection of his writing for the Apple Direct publication as well as other text gathered or produced for this book. The book was written in 1992 (updated in 1996 - 15 years ago!) but remains relevant. The principles are as relevant today as they were way back in the 90's.

Three things made this book useful to me. The first are all Bruce's Principles and Guidelines (which are outstanding) and the collection of them in the appendix. This gives you an overview (and shortcut to relevant places in the book) for each item he discusses in the book. The second is (and this may sound odd given I've used a mac now for 5 years) he describes what the ellipsis means in a Mac menu or on some buttons. I remember seeing it, but I never thought about why it is there. It means that a window or pane will pop up asking for more information before the action you have requested will be executed. This is all about letting users know they can experiment with the system and be assured they can get out again without making any changes. This was a little UI thing I just hadn't consciously grasped the meaning of, and it's just nice to understand why it's there and what it means. The third thing was the in-depth discussion about the Conceptual Model a user generates when they interact with a system. This was like a lightbulb going off for me. This is such a fundamental part of any system and user interface design. It's obvious it happens (users have a view of what systems do, which may or may not be correct), and designers need to cater for this - exposing elements to users that influence that conceptual model, to make it easier for users to understand and interact with the system. I've got a lot of thoughts on this so may write another entry on it.

Anyway, I would definitely recommend both books to anyone involved with software product design - I'm not sure how much new information there will be for experienced UE practitioners, but I'm sure there are some useful nuggets in both for all.