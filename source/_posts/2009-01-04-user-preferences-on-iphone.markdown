---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2009-01-04 11:23:42'
layout: post
slug: user-preferences-on-iphone
status: publish
title: User Preferences on iPhone
wordpress_id: '276'
? ''
: - apple
  - iphone
  - mobile development
  - software
---

Getting and setting user preferences on iPhone is pretty darn easy.  I was looking for a way to automagically store a users email address between different application sessions, and found the linked tutorial immensely helpful (<a href="http://iphonedevelopertips.com/cocoa/read-and-write-user-preferences.html" target="_blank">iphonedevelopertips.com</a>).

I needed to store a string (NSString object actually) however, and the tutorial didn't help me with that. The NSUserDefaults object has dedicated methods for storing and retrieving BOOL, float, and NSInteger values.  It also has a setObject:forKey: method - which is what I ended up using.  The setObject method handles data of types NSData, NSString, NSDate, NSArray or NSDictitionary - making it incredibly useful indeed.

My Preferences.h

``` objective-c
#import <Foundation/Foundation.h>
#import "Constants.h"
@interface Preferences : NSObject {
}
+ (NSString *)emailAddress;
+ (BOOL)setPreferences:(NSString *)emailAddress;
@end
```

My Preferences.m
``` objective-c
#import "Preferences.h"

@implementation Preferences

/*-------------------------------------------
* Return the users default email address
*-------------------------------------------*/
+ (NSString *)emailAddress {
  NSString *returnValue;
  // If preference exists
  if ([[NSUserDefaults standardUserDefaults] stringForKey:kPreferencesEmailAddress]) {
    returnValue = [[NSUserDefaults standardUserDefaults] stringForKey:kPreferencesEmailAddress];
  } else {
    returnValue = @"";
  }
  return returnValue;
}

/*-------------------------------------------
* Write preferences to system
*-------------------------------------------*/
+ (BOOL)setPreferences:(NSString *)emailAddress {
  // Set values
  [[NSUserDefaults standardUserDefaults] setObject:emailAddress forKey:kPreferencesEmailAddress];
  // Return the results of attempting to write the preferences to system
  return [[NSUserDefaults standardUserDefaults] synchronize];
}
@end
```

kPreferencesEmailAddress is a constant I've defined with the key value of the users email address.  It just makes sure I don't mistype anything.

Key (only) differences between my code and the code at <a href="http://iphonedevelopertips.com/cocoa/read-and-write-user-preferences.html" target="_blank">iphonedevelopertips.com</a> are the use of the objectForKey method to set the NSString value, and stringForKey to retrieve the object and cast it to a NSString object.

Hope that helps someone!