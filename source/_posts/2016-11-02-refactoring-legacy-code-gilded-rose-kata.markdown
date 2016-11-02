---
layout: post
title: "Refactoring Legacy Code - Gilded Rose Kata"
date: 2016-11-02 07:52:37 +1100
comments: true
categories: 
---
Often getting legacy code into a state where you're comfortable making changes to it can be challenging. Legacy code can often have varying levels of quality throughout the codebase, as well as different levels of testing applied to it. I've taken over my fair share of old projects that have no tests, or haven't followed SOLID principles, and understanding how the code works and what it does can be a challenge.

The [Gilded Rose Kata](https://github.com/emilybache/GildedRose-Refactoring-Kata) is an exercise in taking a system that calculates "quality" and expiry dates for items in a shop over time. When you start the Kata, you have a single method that calculates the changes in quality and expiry for the items. The business logic implemented is hard to understand, and features nested `if` statements with conditionals that feel inverted. You also need to add functionality to support a new item type to the shop to complete the Kata.

This post will describe how I refactored the code. All code and commits are available at [https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift)

## My Approach

### Acceptance Tests

When refactoring, you're typically looking to improve the internal structure of the code without changing the externally observable behaviour of the system. To ensure the existing behaviour didn't change whilst I was refactoring, I followed the advice in the [readme](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/master/README.md) that recommends "Text-Based Approval Testing". This is essentially an acceptance test in which the behaviour of the system is captured before you start refactoring, and then while refactoring you can compare the changed systems' output with the original output.

To implement this, I captured the text output from running the application in a file and stored it in the repo. I then implemented a test that would replicate running the application, but storing the results in memory. Then the test compared the results of my application with the output from the original application. _As an aside here, when I first implemented this I did a [comparison of the full body](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/25add659ca61871fd65d5204b1ef307c3adf8fc0/GildedRoseTests/GildedRoseTests.swift). I quickly found this to be a terrible solution, and then [implemented a line-by-line comparison](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/commit/ea2f4273b22b1dc61e2bfbb943c8f9874b2bc523) so I could easily tell what part of the calculations were going wrong. Also, I found an issue with the original project setup, so submitted an (accepted) PR to the original repo that corrected the store definintion._

### Understanding the Existing Logic

I've recently read [Working Effectively With Legacy Code](http://www.bookdepository.com/Working-Effectively-with-Legacy-Code-Michael-Feathers/9780131177055?a_aid=rickerbh) by Michael Feathers and there was one approach in this book that seemed to jump out at me to help understand the Gilded Roses logic - Sprout Method. This is where you take a small block of existing functionality (e.g., an `if` statement), extract it out to it's own function, and give it a sensible name. Then, rather than seeing a block of code such as 

```swift
if (items[i].name != "Sulfuras, Hand of Ragnaros") {
  items[i].sellIn = items[i].sellIn - 1
}
```

you see something like

```swift
decrementSellDate(item[i])
```

Example of this [here](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/commit/685e521451764d339d48849d5a9d11b02df01da5#diff-9f2a98a59f7438329af132a5cb5651e0).

The behaviour of the system doesn't change, but suddenly it becomes more readable.  Each "sprouted" method is also accompanied by a test. This allows you to also validate that your extracted code will then continue to work as expected, even with other changes taking place.

For this refactor, I continued to write tests and extract methods to get the giant block of if statements to a [point](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/685e521451764d339d48849d5a9d11b02df01da5/GildedRose/GildedRose.swift) where I could understand what they were doing.

After this, I considered the `if` statement itself. It used a bunch of negation statements to control the flow of execution through the application e.g., `if (items[i].name != "Aged Brie" && items[i].name != "Backstage passes to a TAFKAL80ETC concert")`. Personally, I find this style of `if` statement creates a higher congnitive load that "positive" statement, so I [restructured it](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/bbd3b0c71efec492cda6cc1326954098844cab9a/GildedRose/GildedRose.swift) to use statements more like `if (items[i].name == "Aged Brie" || items[i].name == "Backstage passes to a TAFKAL80ETC concert")`. Maybe that's a personal thing, but I find it simpler to understand - being explicit about what the predicate matches, rather than "everything but these items". During this activity, having the acceptance tests run was crucial to ensure the system behaved the same, while the statements were changing.

After this, I extracted one more method for managing expiry dates. I felt that I had a good understanding of how the existing system worked and the logic was clear in the codebase. Time for some bigger changes.

### Restrictions

One of the key restrictions in the Gilded Rose Kata is that you're not allowed to alter the `Items` class or the `Items` property. Not having this restriction would have made the refactor relatively trivial as you could just create different subclasses that represent the behavour of each of the types of items. To do this however, would have involved altering the `Items` propetry to instantiate the different item types, or turning the `Items` class into a factory that would return items of different types. So, I needed a way to move the rules for specific types of items away from the main calculation flow, so that new item types could be introduced without impacting the existing processes.

### Behaviours

I decided to add an `ItemBehaviour` [class](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/533be7a6b599cbe97b2537cbed4a7e8820e63ebc/GildedRose/ItemBehaviour.swift) that would act as a superclass for each of the items to encapsualte the rules/logic for that specific item. The goal was to separates out the process of incrementing a day in the store, and how each item changes as the days increment. Then I created subclasses ([Brie](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/142973824c68c64a637c3826f59949ac90572348/GildedRose/AgedBrieBehaviour.swift), [Sulfuras](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/a36201b42314a8c2a80c155f3e9aab72610b4e5b/GildedRose/SulfurasBehaviour.swift), [BackStagePass](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/41ce082a2db297176da6dcb4ab68d663e9a3e4e4/GildedRose/BackstagePassBehaviour.swift)) to override the default behaviour for each item that has unique logic. To surface this behaviour, a `BehaviourFactory` [class](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/23c09686703a0248858fef55de8fbb25bc10f96f/GildedRose/BehaviourFactory.swift) was implemented that would return an `ItemBehaviour` based on the items name (not the approach I would have liked to implement, but not being allowed to change the `Item` class is quite a restriction).

The behaviour was then [integrated](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/3809fe56f32c55e2b0c839a0060a16b61ebd55c4/GildedRose/GildedRose.swift) into the main application flow by replacing the if-statement/logic that takes place before the date change with a single call to `updateQualityPreDateChange`.

Next, the functionality to [decrement the sales date](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/commit/ae3a6b704a124f697778e1409d11d03d525a49de) was extracted from the sprouted method, and placed into the specific item behaviour classes. And finally, the changes to [process expired items](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/commit/f12c0174a27e43484cc3bd23b077cc087509b94b) was extracted out into the item behaviour classes.

### Done

The main body of the store updating function has been dramatically simplified. It now `map`s through all the items, retrieves the behaviour for an item, and calls the functions to advance the item through a day. Simples.

```swift
let _ = items.map { item in
  let itemBehaviour = BehaviourFactory.getBehaviour(item)
  itemBehaviour.updateQualityPreDateChange(item)
  itemBehaviour.decrementSellDate(item)
  itemBehaviour.processExpiredItem(item)
}
```

I believe that's significantly easier to understand than the [original code](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/master/swift/Sources/GildedRose.swift).

## New Item

Now that the item behaviour is decoupled from the main day-end processing, adding a new item type is trivial. The new type gets [added](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/188b53c4122ec2fb25e8888633acd9a72d4cee29/GildedRose/ConjuredBehaviour.swift) and then integrated into the [factory](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/188b53c4122ec2fb25e8888633acd9a72d4cee29/GildedRose/BehaviourFactory.swift). The only other thing that needs to happen is the old acceptance test needs to be updated, as this item didn't exist in the original set. Ideally at this point the acceptance test would be updated (or removed) as the system behaviour has moved on, but for the purposes of this exercise I [excluded](https://github.com/rickerbh/GildedRose-Refactoring-Kata-Swift/blob/188b53c4122ec2fb25e8888633acd9a72d4cee29/GildedRoseTests/GildedRoseTests.swift) the new item from the test.

## Wrap Up

Legacy System maintenance is hard, but not impossible. You can always get a piece of business logic to be able to be tested, and restructure messy code so it's simple and understandable. Techniques such as sprout method make gaining initial understanding and making code a bit more clear relatively easy to perform. [Working Effectively With Legacy Code](http://www.bookdepository.com/Working-Effectively-with-Legacy-Code-Michael-Feathers/9780131177055?a_aid=rickerbh) was a super-valuable resource, and covers many, many techniques to get an unwieldy codebase under control, both in terms of gaining understanding as well as getting code testable.

Personally, I found this kata a valuable experience in getting an existing system under control, whilst maintaining existing behaviour. I recommend if you have a spare hour or so give it a go yourself.
