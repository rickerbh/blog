{:layout :post, :title "\"Sandboxing\"", :date "2011-12-22 15:20"}
Since the last post and this one, we've moved 10500 miles (that's nearly 17,000 kilometres!) and only yesterday got all our utilities sorted out, meaning that I've basically been offline (bar mobile phone for the past 3 weeks). Melbourne is fantastic. I really like it here.

Anyway, I've been doing a little bit of work on my application, mostly completion of the requirements, data model, and competitor analysis, as well as reading up a bit on UX and Mac way of doing things (I'll probably cover this in another entry), and most recently have been prototyping some code that I thought would be particularly tricky.

What this code does is synchronise data across other applications on your mac, with my (intended) application. The reason behind this was to make the act of populating my application with data, and keeping the data fresh over time, simpler for users - or as John Maeda states in his Second Law of Simplicity:
>The positive emotional response derived from a simplicity experience has less to do with utility, and more to do with saving time.

I'm into saving time for my (future) happy users, and reusing data that is already on their Mac helps with this - they don't need to double key or manually join-the-dots between the applications.

The prototyping I was creating was using ScriptingBridge to access data that's available in other applications that a user may use. The prototype was going well - I had managed to get some queries and filtering down from 5 minutes to 90 seconds to 6 seconds which was a nice level optimisation, and although there are some issues with ScriptingBridge, it was going to let me do what I wanted to do.

One of the key things with the distribution of this application for me is to have it in the Mac App Store. New mac users are a growth market (look [at](http://mashable.com/2011/09/12/apple-set-to-break-record-for-mac-sales-this-fall-report/) [the](http://www.appleinsider.com/articles/11/11/14/apple_on_pace_to_sell_record_5_3m_macs_in_holiday_quarter.html) [stats](http://www.appleinsider.com/articles/11/12/12/quarterly_us_mac_sales_up_13_expected_to_grow_in_december.html)), and Apple is pushing them down the Mac App Store route to discover and purchase applications. Not having the app in the Mac App Store would be foolish from my perspective.

Today, I was seeing how I could get the application working under sandboxing - as this is now a requirement for new Mac Apps. It's all about making applications safer for users in terms of the resources they can use in terms of network, disk access etc (_although some [prominent developers](http://blog.wilshipley.com/2011/11/real-security-in-mac-os-x-requires.html) aren't convinced_). 

I was sad to discover that ScriptingBridge is precisely one of the technologies that is hit hard by the sandboxing. Basically, applications can receive AppleEvents, send AppleEvents to themselves, and respond to AppleEvents they receive ([source](http://developer.apple.com/library/mac/#documentation/Miscellaneous/Reference/EntitlementKeyReference/AppSandboxTemporaryExceptionEntitlements/AppSandboxTemporaryExceptionEntitlements.html#//apple_ref/doc/uid/TP40011195-CH5-SW1)). The upshot of this is that AppleEvents are pretty useless in a environment of sandboxed applications, as you can only send them to yourself (and no-one else can send them to you). 

There may be a way around this - Temporary Exceptions to the AppleEvent sending issue. You can obtain a temporary exception to send AppleEvents to other applications through the `com.apple.security.temporary-exception.apple-events` key-value pair in your Entitlements file. The main issues I see with this are:

1. They are likely to attract more attention in the review cycle - this in general is a good thing as it's important that Apple check what your application is doing to ensure that it's being a good citizen. However, this is likely to slow down the review process and I suspect guidance on what's acceptable is not consistently applied.
2. They are _temporary_ - Apple could disallow/reject them in the app updates, or just remove the function. The clue is in the name, people.
3. The intent behind the exception - I suspect that the intention behind the temporary exception is to allow pre-existing applications to migrate to a 100% compliant sandboxing world over time, not for new applications. Approval is at risk here, and I don't want to sink significant effort into getting rock solid sync only to be told it's not allowed.
4. The overall future of AppleEvents - this (in my mind) is a clear intention that AppleEvents will not have a strategic future at Apple, and that existing applications may drop support for them as no sandboxing compliant applications can actually use the functions. It seems to me risky to use a technology that seems to have a limited future.

This leaves me with two options. I can either remove the synchronisation functionality from the application, or I can distribute it outside the Mac App Store. My wife actually suggested a third - do both. I think that's what I'll do. Mac App Store is first priority, and depending on the markets response, I'll look at options for data synchronisation.

Still, it's a massive PITA.
