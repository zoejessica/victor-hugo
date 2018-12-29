+++
date = "2018-04-08"
draft = false
title = "CloudKit & CoreData: Testing 101"
categories = ["CloudKit"]

+++

#### N.B. This post will be part of a series I'm putting together on CloudKit and CoreData syncing. As I'm still in the middle of actually writing the code 😬 there will be definitely be updates to this post as my approach evolves. I'm leaving it live because I'm also testing how the site gets generated. Multitasking ftw. 

## CloudKit test setup
There’s probably a lot of code setting up a CoreData and CloudKit stack in your application - in mine it all gets kicked off after launch in the `AppDelegate` , starting a long process of setting up the model stack, finding containers, creating record zones and subscriptions, before beginning to sync any changes from the UI and the cloud.

To keep things clean, I created a new `TestApp` application target against which all the CoreData and CloudKit tests can be run.  This keeps any schema changes made in testing away from the main app - CloudKit’s malleable development schema is super useful, but it would be pretty annoying to see a `TestObject` record type appear alongside actual development data. It also means you can nuke the whole test container with abandon when needed 😈💥 as well as starting each test with the cloud database in a known state.

1. Create a `TestApp` target application - I just used the standard single view template.
2. Set its CloudKit entitlements - its default container will be different from the main app, which is just what we want:
![Screenshot of test app CloudKit entitlements using default container](/images/Screenshot 2018-04-02 16.37.19.png)
3. My code interacting with CloudKit is in a separate `CloudFramework`, with its own test target, `CloudFrameworkTests` that was automatically created. To actually use the  the `TestApp` instead of the main app, edit the `CloudFrameworkTests` target settings; in the General tab set `TestApp` as the host application:
![Screenshot of test framework scheme, showing test app as host application](/images/Screenshot 2018-04-02 16.39.26.png)
4. Now you can delete the main app from the `CloudFrameworkTests` scheme so it won’t get run automatically at the same time:![Screenshot of test framework scheme build options and targets](/images/Screenshot 2018-04-03 16.10.19.png)

## CoreData test setup
The same applies to the local model cache in CoreData - it’s really handy to have a nice clean base for testing. I’m using the “in memory” option to spin up a brand new database whenever needed:
```swift
func mockPersistentContainer(trackHistory: Bool = false) -> NSPersistentContainer {
	let mom = managedObjectModel()
	let container = NSPersistentContainer(name: "Test container", managedObjectModel: mom)
	let description = NSPersistentStoreDescription()
	description.type = NSInMemoryStoreType
	description.shouldAddStoreAsynchronously = false
	container.persistentStoreDescriptions = [description]
	container.loadPersistentStores(completionHandler: { (description, error) in
		precondition(description.type == NSInMemoryStoreType)
		if let error = error {
			fatalError("Failed to stand up in memory coordinator: \(error.localizedDescription)")
		}
	})
	return container
}

func managedObjectModel() -> NSManagedObjectModel {
    let bundle = Bundle(identifier: "BUNDLE_IDENTIFIER_FOR_FRAMEWORK_WITH_DATA_MODEL")
    guard let modelURL = bundle?.url(forResource: "DATA_MODEL_NAME", withExtension: "momd"),
        let model = NSManagedObjectModel(contentsOf: modelURL) else {
            fatalError("Couldn't load managed object model")
    }
    return model
}
```

## Tips & gotchas
* Have a `CloudKitIdentifiers` struct which contains all the strings needed for your stack setup - mine includes the container and subscription IDs. Make one instance for the main app and one for testing, and pass it as a dependency into any setup code.
* Deleting a whole `CKRecordZone` also deletes the records it contains - really useful for the `teardown` function to leave the database clean.
* To check that what you think should be on the server is indeed there, you can use a `CKQuery` with a true predicate for a given record type:

```swift
CKQuery(recordType: "Plan", predicate: NSPredicate(value: true))
```
* But this will only work if the `recordName` field for that type is indexable - and this is only settable via the CloudKit web dashboard. If you are regularly resetting the test database via the CloudKit dashboard, that index will go away. (You'll see a `BAD_REQUEST` error for a `record query` operation in the dashboard logs.)
* Persistent history tracking is not available on an in-memory store type (more on this in an upcoming post about syncing).
* `waitForExpectations(timeout:handler:)` and `OperationQueue.waitUntilAllOperationsAreFinished()` are invaluable for testing all those asynchronous CloudKit calls.
* [Pass in file and line numbers](https://www.bignerdranch.com/blog/creating-a-custom-xctest-assertion/) into any extracted methods, to see test failures in the right place.
