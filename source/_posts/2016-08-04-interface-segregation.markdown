---
layout: post
title: "Interface Segregation"
date: 2016-08-04 09:28:17 +1000
comments: true
categories: 
---
The Interface Segregation principle (part of [S.O.L.I.D.](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design\))) is about, very simply, not making subclasses or clients implement interfaces they're not concerned with. [Robert Martin](https://drive.google.com/a/simplemachines.com.au/file/d/0BwhCYaYDn8EgOTViYjJhYzMtMzYxMC00MzFjLWJjMzYtOGJiMDc5N2JkYmJi/view) calls these 'fat' interfaces. They contain functions or methods that are unrelated to each other, and could be split out into more cohesive interfaces.

Forcing clients to implement interfaces they're not concerned with causes unnecessary tight coupling of the client to the interface. If the interface changes, the client needs to reimplement/update itself even if it doesn't use that specific interface function. The result is wasted development effort in maintaining unnecessary code for testing and implementation. This should be avoided.

Issues with interface definition can arise in languages that support [inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming\)), [subtype](https://en.wikipedia.org/wiki/Subtyping) conformance, or concepts like [Interface](https://en.wikipedia.org/wiki/Interface_%28Java%29) or [Protocol](https://en.wikipedia.org/wiki/Protocol_(object-oriented_programming\)). Specifically, issues are more likely to occur when an object or type can only inherit/implement one super-class or protocol/interface, such as with inheritance with (most) object-oriented languages. C++ is a notable exception here with support for multiple-inheritance, and implementation of protocols/interfaces via abstract base classes with pure virtual functions. The majority of languages I've seen that support the concept of interfaces, also support multiple interface inheritance. This is supported in Swift, Java, and Objective-C. Ruby can support this via the include statement, although the duck typing removals the formal need for this definition - same with Python. Haskell supports this via type class conformance.

## Show me some code

Here's a contrived example of an interface that tries to do too much. It's in swift, but it should be understandable. Lets say we're modelling animals.

```swift
protocol Animal {
  var species: String { get }
  var legs: Int { get }
  func speak() -> String
  func birth() -> Animal
}
```

And we define a couple of animals.

```swift
class Dog: Animal {
  var species = "Canis lupus familiaris"
  
  var legs = 4
  
  func speak() -> String {
    return "Woof"
  }
  
  func birth() -> Animal {
    return Dog()
  }
}

class Cat: Animal {
  var species = "Felis catus"
  
  var legs = 4
  
  func speak() -> String {
    return "Meow"
  }
  
  func birth() -> Animal {
    return Cat()
  }
}
```

This all seems fine, but becomes unstuck when we attempt to model something [Oviparous](https://en.wikipedia.org/wiki/Oviparity), or a sterile hybrid.

```swift
class Chicken: Animal {
  var species = "Gallus gallus domesticus"

  var legs = 2
  
  func speak() -> String {
    return "Cluck"
  }
  
  func birth() -> Animal {
    // Chickens have eggs, not chickens. 
    // And they don't "birth" them. They lay them.
  }
}

class Mule: Animal {
  var species = "Equus asinus x Equus caballus"

  var legs = 4
  
  func speak() -> String {
    return "Bray"
  }
  
  func birth() -> Animal {
    // Uh oh, Mules are typically sterile. They can't reproduce.
  }
}
```

The interface for Animal forces all animals to be able to birth things, and not all animals do. To solve this, I see a couple of options.

- We make the `birth()` function optional
- We extract the `birth()` function out to a separate `Protocol` and compose multiple protocols together

Personally I prefer the extraction of the function to a separate protocol. If we made the `birth()` function optional, any object that attempts to use it on any animal will need to ensure it's available before it can use it, and potentially force consumers of a function that uses that function to also deal with optional returns.

```swift
func birthSays(parent: Animal) -> String? {
  if let child = parent.birth()? {
    return child.speak()
  } else {
    return nil
  }
}
```

If the `birth()` function is extracted out, then:

1. Chickens and Mules won't need to implement the `birth()` method, and
2. We can typecheck methods so we don't need optionals, in languages that support this construct.

```swift
protocol Animal {
  var species: String { get }
  var legs: Int { get }
  func speak() -> String
}

protocol Egg {
  func hatch() -> Animal
}

protocol Viviparous {
  func birth() -> Animal
}

protocol Oviparous {
  func lay() -> Egg
}

class ChickenEgg: Egg {
  func hatch() -> Animal {
    return Chicken()
  }
}

class Chicken: Animal {
  var species = "Gallus gallus domesticus"

  var legs = 2
  
  func speak() -> String {
    return "Cluck"
  }
}

extension Chicken: Oviparious {
  func lay() -> Egg {
    return ChickenEgg()
  }
}

func birthSays(parent: Viviparous) -> String {
  return parent.birth().speak()
}
```

You can see above that the Chicken is no longer required to implement birth. Through conformance to multiple, specific/detailed protocols it only needs to support functions and properties that make sense to that specific Class. This splitting of protocols alse ensures that we can typecheck inputs to functions, reducing the need for boilerplate code performing nil checks on optionals.

### Surprising Usage

To illustrate another benefit of small interfaces, we consider the relationship between parents and children. If the relationship between two entities is abstracted out and made generic, we can think of it as a Node in a Graph, with a parent (node), and multiple children (other nodes).

```swift
protocol Node {
  var parent: Node { get }
  var children: [Node] { get }
}
```

With this view of a `Node`, we can model families of Viviparous animals. A Dog can return it's children, and they can reference their parents. This `Node` however, can also be reused for any directed graph, such as dependencies between different software libraries. If a client implements a function to produce a family tree of Dogs via the `Node` interface, the exact same code can be reused to produce a tree of library dependencies, as it's based on the generic `Node`, not `Animal`.

_hat tip to [@triggerNZ](https://twitter.com/triggernz) for this example_

Interface Segregation is one of the S.O.L.I.D. principles (I). Through ensuring that your interfaces small, targetted, and cohesive, you simplify implementation for clients. Clients won't be required to implement interfaces that don't make sense in the context of their object. Your interfaces also have greater opportunities for reuse, due to being more composable. Clients will be forced to change less, as only changes that impact their operation will need to be managed, rather than interface changes that they don't care about. 
