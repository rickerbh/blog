---
layout: post
title: "A small functional refactoring of Javascript"
date: 2016-07-05 16:26:31 +1000
comments: true
categories: 
---
I'm working on a web based productivity application at the moment, and have been modifying some old code. This code deals with synchronisation of data with external services, and storage of metadata about the synchronisation of that data. The application had quite a bit of duplication in dealing with this metadata; specifically in extracting data from the stored structures. In the interests of having very DRY Javascript, it was time to refactor.

The configuration/metadata structures typically look like

```
{
  ...
  config: [{ key: 'attribute-name', value: { value: 'attribute-value' }}],
}
```

_Sidenote: value is embedded in value because the top-level value item actually receives an object to store, so other fields can be added in the future_

The metadata object was then optionally attached to other user-entered data entities, and queried when updates that require synchronisation to be triggered.

```
const containerObject = {...
  config: [
      { key: 'itemId', value: { value: 12345 }}, 
      { key: 'itemState', value: { value: 'active' }}
    ],
};
```

The application was already using [RamdaJS](http://ramdajs.com/) to extract data from these types of structures.  To query, there was a whole bunch of duplicated code that traversed the containing objects, and extracted data. Example below.

```
// Extract config object, maybe.
const syncData = R.propOr({}, 'config')(containerObject);
// Find a structure with a specific key value
const itemSyncData = R.find(R.propEq('key', 'itemId'), syncData) || {};
// Extract the value from the entity with that key
const itemId = R.path(['value', 'value'], itemSyncData);
```

To DRY all this up, I looked at the possibility of using [Currying](https://en.wikipedia.org/wiki/Currying) and [Partial Application](https://en.wikipedia.org/wiki/Partial_application) to help me define generic extraction functions, and reuse them.

First of all, I turned the above into a composed function, with the results of one step flowing as the inputs to the next.

```
// compose reads from the bottom up, like you're feeding in the object from
// the end and it's consuming it, right to left.
const composedFunction = R.compose(
  R.path(['value', 'value'],
  R.find(R.propEq('key', 'itemId')),
  R.propOr({}, 'config')
); 

// How to use?
const itemId = composedFunction(containerObject);
```

However, this only supports `config` objects in the container, and `itemId`'s inside that. We can make this more generic.

```
const composedGenericFunction = (data, key) => R.compose(
  R.path(['value', 'value'],
  R.find(R.propEq('key', key)),
  R.propOr({}, data)
); 

// How to use?
const itemId = composedGenericFunction('config', 'itemId')(containerObject);
const itemState = composedGenericFunction('config', 'itemState')(containerObject);
```

Better because we get more reuse, but we still are repeating ourselves with the definition of the attribute that houses the config. So, lets make our function even more reusable with currying and partial application.

```
const composedGenericCurriedFunction = R.curry((data, key) => R.compose(
  R.path(['value', 'value'],
  R.find(R.propEq('key', key)),
  R.propOr({}, data)
)); 

// How to use?
const configGetter = composedGenericCurriedFunction('config'); // Partial Application
const itemId = configGetter('itemId')(containerObject);
const itemState = configGetter('itemState')(containerObject);
```

We can take this a step further with something like below, and generate a getter that will retrieve the itemId from different containerObjects.

```
const idGetter = configGetter('itemId');
const idOne = idGetter(containerObject);
const idTwo = idGetter(containerObjectTwo);
```

We can also extract fields from other objects that conform to the same structure, but aren't under a `config` key.

```
const syncDataGetter = composedGenericCurriedFunction('syncData');
```

So, with the use of currying and partial application with Ramda, we can create generic, reusable functions that are then used to generate other functions that we can use in our application. Super easy and effective way to DRY out your javascript.

