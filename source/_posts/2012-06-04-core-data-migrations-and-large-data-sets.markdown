---
layout: post
title: "Core Data Migrations and Large Data Sets"
date: 2012-06-04 12:14
comments: true
categories:
---
I recently updated [Moving Van](http://click.linksynergy.com/fs-bin/stat?id=*W1h7qYtoaI&offerid=146261&type=3&subid=0&tmpid=1826&RD_PARM1=http%253A%252F%252Fitunes.apple.com%252Fus%252Fapp%252Fmoving-van%252Fid357418069%253Fmt%253D8%2526uo%253D4%2526partnerId%253D30) (_you should buy it now!_) and published the new version in the app store late last week. It was a substantial update to the application - it had a completely new UI with custom interface controls, as well as a whole stack of features that customers had been asking for - things like room autocompletion, saving images to camera roll, more export options, moving items between boxes etc.

As part of this update, I also remodelled the Core Data entities that power the application. The initial model that was used was, let's say, a little naïve in terms of the way that the stored data would impact performance of the application. It stored an image on an item as binary data within the Item entity itself, which in retrospect was a terrible idea because of table view performance. The new version split out the image to a separate entity, which means that when the Item entity loads, the image doesn't get loaded unless explicitly needed because of the faulting behaviour of Core Data and entity relationships.

So, to get out of this historic design decision, a data migration was required. The migration itself was relatively simple, with pretty much everything working from a standard mapping model (add two entities, copy existing entities, create relationships with new entities). I had to use a custom migration policy for one aspect of the migration - two image entities are created for each item (for tableview performance reasons). There is the original image, and a thumbnail version of that image. The custom policy needed to take the original image from the source Item, scale the image down, and set it in the new Thumbnail entity, but that itself was relatively simple.

The migration was tested with all possible permutations of the data that a user could create, including a large data set with over 100 boxes and hundreds of items. The migration would take a few seconds to run, and everything was working well. I submitted, and released the new version.

DISASTER.

It appears that my data sets for testing were inadequate. Quite a few users of the application store images for every one of their items. 300, 400 of them. Some users don't even use the text descriptions for items, they just use images. The larger data sets used for testing were text only - none of the testing involved hundreds of items with images. A database with around 500 images is about 300Mb - I think that's quite a large CD store for the iPhone.

What was happening is that Core Data, while doing the migration, was choking trying to load all the Item entities (with images embedded) into memory. The lightweight migration mechanism seems to try to be fast, over being resource efficient. On the iPhone this is a _bad thing_ if you have a large volume of data - your application will be terminated with little to no warning.

Apple have [specific recommendations](http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/CoreDataVersioning/Articles/vmCustomizing.html%23//apple_ref/doc/uid/TP40004399-CH8-SW9) for what do with large core data sets - mainly around splitting a lightweight migration into separate mapping models. This approach is fine if you have a large number of entities, but it a _useless_ strategy if you have a large number (or more precisely, a large data volume) of an individual entity. Their "chunks" of data refer to a per entity chunk - the approach still attempts to load all instances of an entity into memory. What I needed was a way to have multiple "chunks" of a specific entity, so the whole set was not loaded into memory at once.

The approach I took to solve this problem is very "manual". It consists of the following steps:

1. Determine if a migration is required - if so, pop a migration controller that informs the user a migration is taking place, and start the migration.
2. Create a Core Data stack with the "old" model, and old version of the store as a source.
3. Create another Core Data stack with the "new" model, and a new store as the destination.
4. Request a set of entities from the old data store, with a small batch size to avoid loading all entities at once.
5. Traverse the object graph of those old entities, creating each instance of an entity in the new data store.
6. Save the new store every 10 or so entities - this is to ensure that the NSManagedContext doesn't consume too much memory with unsaved objects hanging around.
7. After this is all finished, backup the original data store, and move the new one to take its place.
8. Finally, post a notification for the AppDelegate to receive, that signals the migration is complete and the rest of the startup sequence can continue.

The approach works - the application no longer runs out of memory on migration. However, the mapping model is now useless as it's never used, and there are a couple of interesting points. First one is that the migration takes up extra storage space as we are creating an extra store with pretty much the same volume of data in it - just laid out differently. I'm not sure if this happens when CD performs a migration - I suspect it is, but what worries me is that if a user is low on space, the migration could cause the disk to fill up. The other thing that I noticed was that the migration is considerably slower that a CD managed lightweight migration. However, it actually works on large data sets, unlike the CD managed lightweight migration, so the positives here outweigh the negatives.

There is probably a way to solve this that utilises more of the Migration classes that Apple provide - specifically subclassing NSMigrationManager - but, I didn't really have enough time available to figure that out - I needed a fix *now*.

And now some code.

Determining if your Core Data store needs to be migrated
--------------------------------------------------------
    // See if a database exists to be migrated
    NSString *sourceStorePath = <Your source store path in the file system>
    if (![[NSFileManager defaultManager] fileExistsAtPath:sourceStorePath]) {
      // Database doesn't yet exist. No need to test data compatibility"
      return NO;
    }

    // Create a persistence controller that uses the model you've defined as the "current" model
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"<Your models directory name>" withExtension:@"momd"];
    NSManagedObjectModel *model = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    NSPersistentStoreCoordinator *psc = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];

    NSError *error = nil;
    NSURL *sourceStoreURL = [NSURL fileURLWithPath:sourceStorePath];
    NSDictionary *sourceStoreMetadata = [NSPersistentStoreCoordinator metadataForPersistentStoreOfType:NSSQLiteStoreType
                                                                                                   URL:sourceStoreURL
                                                                                                 error:&error];
    // Do error checking... Removed from the code sample.
    NSManagedObjectModel *destinationModel = [psc managedObjectModel];
    BOOL pscCompatible = [destinationModel isConfiguration:nil
                               compatibleWithStoreMetadata:sourceStoreMetadata];
    // if pscCompatible == YES, then you don't need to do a migration.

Loading old and new Core Data Stacks
------------------------------------
You'll need to do this twice - just swap out the model name for old/new models and keep the references to the MOCs that are created. Ensure you have a different store path for your new store!
For the new model, it's a good idea to also test if a file exists at the new model location - it could be indicative of a migration that's previously failed.
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"<source or destination model name>" withExtension:@"mom" subdirectory:@"<Your models directory name>.momd"];
    NSManagedObjectModel *model = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    NSPersistentStoreCoordinator *psc = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];

    // Get the store url
    NSString *sourceStorePath = <Your source/destination store path in the file system>
    NSURL *sourceStoreURL = [NSURL fileURLWithPath:sourceStorePath];

    // Use this for source store - ensures you don't accidentally write to the entities
    NSDictionary *options = [NSDictionary dictionaryWithObject:[NSNumber numberWithBool:1]
                                                        forKey:NSReadOnlyPersistentStoreOption];

    // Use this for destination store - makes it writeable
    NSDictionary *options = [NSDictionary dictionaryWithObject:[NSNumber numberWithBool:0]
                                                        forKey:NSReadOnlyPersistentStoreOption];
    NSError *error = nil;
    [psc addPersistentStoreWithType:NSSQLiteStoreType
                      configuration:nil
                                URL:sourceStoreURL
                            options:options
                              error:&error];
    // Do error checking... Removed from the code sample.
    NSManagedObjectContext *moc = [[NSManagedObjectContext alloc] init];
    [moc setPersistentStoreCoordinator:psc];
    [moc setUndoManager:nil];

Get your entities from your original store, and create them in the new store
----------------------------------------------------------------------------
You can't use your entity classes here, everything has to be done via KVC. This is because your entity classes will no longer map to the old model correctly.
    NSFetchRequest *oldFetchRequest = [[NSFetchRequest alloc] init];
    NSEntityDescription *oldEntity = [NSEntityDescription entityForName:@"EntityName"
                                                 inManagedObjectContext:oldContext];
    [oldFetchRequest setEntity:oldEntity];
    // Set the batch size so we don't attempt to retrieve all the data at once - this is the key to the whole thing!
    [oldFetchRequest setFetchBatchSize:10];

    NSError *error = nil;
    NSArray *entities = [oldContext executeFetchRequest:oldFetchRequest error:&error];
    int count = 0;
    for (NSManagedObject *oldEntity in entities) {
      // Creating new entity
      NSManagedObject *newEntity = [NSEntityDescription insertNewObjectForEntityForName:@"EntityName"
                                                                 inManagedObjectContext:newContext];
      [newEntity setValue:[oldEntity valueForKey:@"someAttribute"] forKey:@"someAttribute"];

      // If your entity has relationships...
      for (NSManagedObject *aRelatedEntity in [oldEntity mutableSetValueForKey:@"someRelationship"]) {
        NSManagedObject *newRelatedEntity = [NSEntityDescription insertNewObjectForEntityForName:@"RelatedEntityName"
                                                                          inManagedObjectContext:newContext];
        [newRelatedEntity setValue:[aRelatedEntity valueForKey:@"someOtherAttribute"] forKey:@"someOtherAttribute"];
      }
      // Save periodically
      count++;
      if (count % 10 == 0) {
        [newContext save:&error];
        // Do some error handling
      }
    }
    [newContext save:&error];
    // Do some error handling
    // Migration is complete, if you've traversed all your entities.

When I encountered this problem I couldn't find any example code for how to do this migration - hopefully this helps someone.

If anyone does know of an alternative (better) way to get around this issue, please let me know in the comments.





