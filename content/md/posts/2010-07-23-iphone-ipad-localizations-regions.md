{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2010-07-23 07:45:12", :layout :post, :draft? false, :title "'iPhone & iPad (iOS) Localizations and Regions '"}

Recently I have been doing some localizations of an iOS app from English (US) to English (UK). The iPhone development guides from Apple describe how to support multiple languages (such as English, German, Japanese), but fail to describe how to support multiple variants of a single language. By this I mean support support for US English, English, NZ English, AU English. The word I needed to regionalize was Behavior (or Behaviour, depending where you come from). 

In the Apple Developer Library, it explicitly states that for MacOS applications take both the Language and Regional preferences of the user into account, but only look at the preferred <em>language</em> on iOS - <a href="http://developer.apple.com/iphone/library/documentation/MacOSX/Conceptual/BPInternational/Articles/InternatSupport.html">Support for Internationalization</a>.  This means that a single variant <em>per language</em> is supported. 

However, these is a way around this. I'm not sure if this is a <em>good</em> thing to do, but it works for me and I haven't noticed any ill side effects yet.

To support both US English and British English in your iOS application, create 2x Localization.strings files just as you would for multiple language. Put the US English translation file Localization.strings in a directory in your iPhone app called English.lproj (Apple defaults) and the British English translation in a directory named en_GB.lproj (just in case they decide to support regions in the future).

Then, you'll need to create some code to manually set the preferred localization. In your main.m file (yup, main.m is being edited) alter it so it performs some logic similar to below.

``` objective-c
int main(int argc, char *argv[]) {
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];
	NSString *language = [[NSLocale preferredLanguages] objectAtIndex:0];
	NSString *locale = [[NSLocale currentLocale] objectForKey: NSLocaleCountryCode];
	if ([language isEqualToString:@"en"] && [locale isEqualToString:@"GB"]) {
		[[NSUserDefaults standardUserDefaults] setObject:[NSArray arrayWithObjects:@"en_GB", @"en", nil] forKey:@"AppleLanguages"];
	}
	int retVal = UIApplicationMain(argc, argv, nil, nil);
	[pool release];
	return retVal;
}
```

When the line 
``` objective-c
int retVal = UIApplicationMain(argc, argv, nil, nil);
```
gets executed, it seems to set up all the Localization bundles before calling the application:didFinishLaunchingWithOptions method on your app delegate, so putting Localization code in the app delegate is too late. So, what the code above does is retrieve the users current language and region, and compares those against pre-determined values - en for the language and gb for the region. If these match, then I force a new setting in the NSUserDefaults to overwrite the users preferred language. Then, when the UIApplicationManager function gets called, it appears to retrieve the users preferred language setting, and look up the Localization for that - which in my case I've forced to be en_GB.

One thing you need to be careful about is persistence of this NSUserDefault setting.  It is saved once it's set, and persists through multiple application executions. To get around this (lets say the user changes their region back to US), you need to remove the setting after the bundle initialization has taken place.  In you app delegates application:didFinishLaunchingWithOptions method, just execute the following code.

``` objective-c
[[NSUserDefaults standardUserDefaults] removeObjectForKey:@"AppleLanguages"];
```

This wipes out the NSUserDefault setting that the app made in the main.m file.

If anyone knows of issues with this approach (apart from being a dirty hack), or faults with my code please let me know in the comments below. I wish Apple supported different regions per language natively in iOS but they don't. This is the only way I've found to do this, and continue to use localization functions such as NSLocalizedString.

 
