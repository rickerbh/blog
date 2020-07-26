{:layout :post, :title "\"Back to Basics - Editors\"", :date "2016-06-27 20:54:48 +1000"}
My boss bought me a copy of the seminal classic [The Pragmatic Programmer](http://www.bookdepository.com/Pragmatic-Programmer-Andrew-Hunt/9780201616224?a_aid=rickerbh). It's full of sensible advice for both beginners and experienced software professionals. 

I have lots of thoughts about the contents of this book but what I'd like to cover is usage of text editors, and specifically becoming proficient with a editor that's typically universally available. I can think of 3 of these for UNIX systems. `vi`, `emacs`, and `nano`. 

When I was at university I used emacs for all CS course text editing. In my first job, the UNIX servers we had didn't have emacs installed. I wanted to install emacs, but the application vendor didn't allow us to, as they had a support contract and hadn't tested their application with emacs on the box, so no-dice. vi was it for a few years. Then, I just a stopped using these types of editors for about 11 years. The advent of IDEs (Eclipse, Xcode) meant I didn't have to use these "basic" editors about had a lot of point-and-click, plugins, refactoring etc functionality available. 

Until now. 

The Pragmatic Programmer has convinced me to relearn basic (and powerful) UNIX tools, and I'm starting with emacs. In my job (at the moment) I typically do about 50-80% dev, the rest other stuff (sales, management, architecture, analysis). So I spend a fair amount of time in development environments. I'm not a big shortcut key user, and this needs to change - it's just wasting time moving that mouse around to get to menu items and select text. 

So far, I've started with base emacs installations, Haskell packages, git packages, Clojure packages, some convenience packages, and I'm about to start with some JavaScript packages. Also getting used to the keystrokes again is a bit tricky, but I'm getting there. To help out, I've moved other IDEs I use (like WebStorm) to use emacs bindings as well. 

I'm enjoying the ability to use the same environment for text + 3 languages + code control. Super powerful.

If you're interested in my setup, I have an emacs file in my [dotfiles git repo](https://github.com/rickerbh/dotfiles). Check it out (ho ho ho). 

If you have any tips for emacs please let me know in the comments or on Twitter - [@rickerbh](http://twitter.com/rickerbh)
