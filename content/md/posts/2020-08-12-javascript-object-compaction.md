{:layout :post, :title "Javascript Object Compaction", :date "2020-08-12" :draft? false}

I'm working on a feature in [BeeCastle](https://beecastle.com) at the moment to ingest call data from Microsoft Teams to augment our interaction records. It requires a flexible approach to updating existing records, as with different types of interactions and calls, we get different data.

During this process we get a javascript object that might have `null` or `undefined` values and these should not be persisted in the database, so we need a way to strip these out. Idiomatic ES6 usage would mean we need to do something like below.

```
const myObject = {
  a: 'thing',
  b: 1,
  c: 0,
  d: true,
  e: false,
  f: null,
  g: undefined,
}

Object.keys(myObject).forEach((key) => (myObject[key] == null) && delete myObject[key]);

// myObject => {"a": "thing", "b": 1, "c": 0, "d": true, "e": false}
```

I've got a couple of issues with this.
- It mutates `myObject`, and `myObject` is `const` - I know in Javascript this is about reassignment, not modification of the internal structure, but I generally avoid mutation of data structures
- It's a bit obtuse - it requires knowledge of the chaining behaviour of `&&` and the very atypical syntax of `delete`. I find this implementation non-obvious at first glance
- The inner function for `forEach` reaches outside its scope to get to `myObject` and modify it. You can't abstract that implementation out without rework to pass in `myObject` and the key it's operating on, and this sounds like more of a disaster because then the scoping and repassing of `myObject` needs to be handled - it would make more sense at this point to rewrite it as `reduce`.

There must be a better way. Within the js parts of our codebase we use [Ramda](https://ramdajs.com) heavily. Ramda doesn't have a single function to enable this, but it does have two we can compose to provide the same functionality.

```
const myObject = {
  a: 'thing',
  b: 1,
  c: 0,
  d: true,
  e: false,
  f: null,
  g: undefined,
}

const compacted = reject(isNil)(a)

// compacted = {"a": "thing", "b": 1, "c": 0, "d": true, "e": false}
```

It hides lots of the implementation, but the code is so much clearer, due to the declarative nature of the operations. It is also reusable, as functions in ramda are curried and the inner functions don't need to grab data outside their scope. `const compactObject = reject(isNil)` and applied with `compactObject(myObject)`.

There can be significant benefits in terms of clarity and reduction in LOC (implying fewer bugs in general: less code = fewer opportunities for bugs) from a utilisation of more pure functional principles. Give it a go :-)
