{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2010-01-07 19:30:13", :layout :post, :draft? false, :title "'Calculate age in objective-c '"}

For an iPhone application I'm developing for a client I need to capture the birthdate of a user, and then show their age on a profile screen. I went looking for a function to help with this simple and tedious task, but couldn't find any example code that could be lifted to help me, so I rolled my own.  Here is what I made, steal as you see fit.

``` objective-c
- (NSInteger)age:(NSDate *)dateOfBirth {
  NSCalendar *calendar = [NSCalendar currentCalendar];
  unsigned unitFlags = NSYearCalendarUnit | NSMonthCalendarUnit | NSDayCalendarUnit;
  NSDateComponents *dateComponentsNow = [calendar components:unitFlags fromDate:[NSDate date]];
  NSDateComponents *dateComponentsBirth = [calendar components:unitFlags fromDate:dateOfBirth];
  
  if (([dateComponentsNow month] < [dateComponentsBirth month]) ||
      (([dateComponentsNow month] == [dateComponentsBirth month]) && ([dateComponentsNow day] < [dateComponentsBirth day]))) {
    return [dateComponentsNow year] - [dateComponentsBirth year] - 1;
  } else {
    return [dateComponentsNow year] - [dateComponentsBirth year];
  }
}
```
 
