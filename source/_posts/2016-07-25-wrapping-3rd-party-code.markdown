---
layout: post
title: "Wrapping 3rd Party Code"
date: 2016-07-25 13:05:35 +1000
comments: true
categories: 
---
I've been integrating a system I've been developing with a third party service for data synchronisation. We're looking to synchronise tasks out with systems like Pivotal Tracker, Jira, Trello etc, but were unsure which of those systems we would actually use. I'm currently [reading](https://hamishrickerby.com/books/) [Clean Code](https://www.bookdepository.com/Clean-Code-Robert-C-Martin/9780132350884?a_aid=rickerbh), and there's a really interesting and relevant chapter on "Boundaries". It covers a similar scenario to what we were facing: _Using Third-Party Code_.

## Isolation

Generally, you should wrap any third party code that you're dependent on with your own interfaces for that code. That way you get to define _how_ your main application logic interfaces with the third party code, rather than having to have to bend your application to conform to a third party library, API, or applications structure. Isolating the complexities of dealing with third parties within an application to a particular class or module behind an interface that you control also allows you to:

1. deal with changes to that third party (e.g., API upgrade)
2. swap out that integration with another one

If we were building direct integration (with a project management tool like Jira, Pivotal Tracker, Trello) into our application, the models and interfaces of the third party system would leak into our core system.

_Pivotal Tracker models epics with attached labels, and to attach a story to an epic we actually attach it to the label of the epic. Pivotal Tracker also handles story creation with labels/epics differently from story updates with labels/epics._

```javascript
const saveStoryCallback = (response, error) => {
  ... // Error handling
  const story = response.data
  // Perform other post-save actions (eg, update aggregated or count fields) 
  // Sync with Pivotal
  if story.epic {
    // check epic exists at pivotal
    // if so, extract the label ID
    // if not, create it and gather the label ID
  }
  // check story exists at pivotal
  // if so, update fields with attached label ID if the story has an epic
  // if not, create story, then attach label as a separate call
  // save sync metadata
}
```

Our save story callback will need to deal with the intricacies of the external system/library. If we wanted to swap this out for another tool (e.g. Trello) we'd need to completely rewrite the logic in our save story callback, which is intermingled with core application logic. If we wanted to model the same synchronisation process with Trello, the logic would be different because the external model is different. We'd model epics as a Board, and stories as a Card.

To avoid changes to our core application logic, we need to isolate the synchronisation logic behind an interface that represents entities in the language of our application, not the third party application.

```javascript
const syncStory = (story) => { ... }
const deleteStory = (story) => { ... }
const syncProject = (project) => { ... }
const deleteProject = (project) => { ... }
const syncEpic = (epic) => { ... }
const deleteEpic = (epic) => { ... }
```

Then, we need to implement an [adapter](https://en.wikipedia.org/wiki/Adapter_pattern) to interface with one of the third parties. Lets say we're interacting with Pivotal again:

```javascript
import { syncEpic, syncStory } from "sync-adapter"

const saveStoryCallback = (response, error) => {
  ... // Error handling
  const story = response.data
  // Perform other post-save actions (eg, update aggregated or count fields)
  syncEpic(story.epic)
  syncStory(story)
}
```

Then in our sync-module:

```javascript
const syncEpic = (epic) => {
  // if no epic passed in, return
  // check epic exists at pivotal
  // if so, update the fields that have changed and save
  // if not, create the epic
  // save sync metadata
}
```

If we need to swap out Pivotal for Trello, we can simply replace the contents of the `sync-adapter` with the implementation for the different provider. The core application callback won't have to change if the interface is isolated in this way.

## Multiple Third Parties

Lets say in the future we have a requirement to support more than one external system. The existing isolation model between the callback and the sync-module still applies. We'd just need to inject another adapter in the middle of this flow.

1. Rename `sync-adapter` to represent the specific external system it relates to: `pivotal-adapter`.
2. Implement the appropriate adapter for the new external system: `trello-adapter`
3. Implement a new `sync-adapter` that will interface with both of these modules.

```javascript
import { syncEpic as syncPivotalEpic } from 'pivotal-adapter'
import { syncEpic as syncTrelloEpic } from 'trello-adapter'

const syncEpic = (epic) => {
  const syncData = epic.syncMetadata
  if syncData.service == 'pivotal' {
    syncPivotalEpic(epic)
  } else if storySyncData.service == 'trello' {
    syncTrelloEpic(epic)
  }
}
```

Addition of more third parties can take place in the future without further changes to the core application logic. All the changes are pushed out to the boundaries of the sytems.

## Upgrades and Changes

If a third party changes their interface, or even their domain model, all changes will be isolated to the integration module alone. Your core application flow should be unaffected by the change, as the interface it interacts with should remain stable.

_When Pivotal changed their API from v3 to v5, they introduced the concept of the Epic. Previously our application would have had epics internally, but the adapter would have converted epics to labels to support Pivotals model. With changes for their v5 API, Epics become a first class citizen as far as they are concerned and we'd update the adapter, but our application core application would not need to change._

## Wrap Up

Isolating third party code via adapters provides benefits in terms of abstracting logic and complexity out of your main application flow. Your application will communicate with the third parties in a consistent manner via the interface, and not be forced to change if a change appears. It ensures that any future conceptual, model, flow/logic and interface changes in the third party only impact the code that deals with that third party. It gives you the ability to swap out your external dependencies with little to no impact on your core applications, as well inject new functionality.
