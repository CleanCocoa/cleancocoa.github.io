---
title: Core Data Violates DDD Principles by Default
created_at: 2014-11-13 17:21:37 +0100
kind: worklog
tags: [ ddd, core-data, software-architecture ]
vgwort: http://vg08.met.vgwort.de/na/c926cf6f279c41dbbb1a07261b80e0c8
comments: on
---


I'm working on a way to make the Word Counter watch files and folders on a user's disk. This will enable to measure project progress.

Until now, I always used `.plist` files to store records with the end in mind that I'm going to switch to a better alternative in the future. Since I'm adding all-new data tracking, I thought I might as well try different solutions now. 

I decided to use Core Data to persist the list of paths belonging to any given project and to store the usual 24 records per day. It's more convenient than managing a SQLite database on my own.

## Core Data Belongs in Infrastructure but Ripples Through the Whole App

Since Core Data is about data persistence, it belongs into the **infrastructure layer** of the application. I'm not exactly using a layered architecture, but it's easy to grasp: in infrastructure, the application communicates with the operating system, with the file system, or with remote servers.

All the word counting takes place in the application's **domain**. There, the notion of a `Recorder` and an `ApplicationRecord` exists. It's the heart of the application. They define ports to which the infrastructure can couple in order to save records to disk.

Starting with the domain, I'd like to create a `Project` to hold on to a list of `ProjectPath`s in order to keep track of the project's structure.  `PathRecord`s should contain the actual counts per hour per day. A project itself can't be counted; its progress meter is made up of the individual files's and folders's word counts, so `PathRecord` will suffice.

Core Data can take care of managing these objects and their relations just fine. The thing is that every Core Data managed object knows the bucket it belongs to (`NSManagedObjectContext`). Instead of keeping persistence logic to the infrastructure layer, you leak knowledge of changing data on disk throughout the app when you use Apple's convenience methods to tie Core Data objects to user interface components. 

The domain should contain the algorithms and mechanisms to manipulate data behind the scenes. The domain increases a counter when you type a word. The user interface can obtain the counter and display it, but it can't modify it on its own. This keeps things simple. Likewise, you decouple domain objects and the infrastructure mechanism.

For `.plist`s, I use a domain object's capability to create a `NSDictionary` from its data. This way, the whole aggregate will be saved (for example, the whole project tree).

What would you do with Core Data? Change the Core Data managed objects and save the changes. 

According to principles of _Domain Driven Design_, you focus on the domain and move infrastructure and user interface to the periphery. You can defer decisions about how you store data until later. As a litmus test, you should be able to swap out `.plist` based storage with anything else without touching the domain. That's a sign of clean decoupling of domain and infrastructure. To keep maintainability high, that's what I aim for.

With Core Data's suggested architecture, though, you dive in early and won't get out easily if you make up your mind later, for Core Data is entangled through the whole app.

You lose most of its benefits when you restrict it to the infrastructure, though.

Here's two approaches I came up with:

## 1\. Degrade Core Data Objects to Mere Data Access Objects

A Core Data object provides accessors for all of the entity's attributes. You change the `project.title` of a managed object, tell its context (not the object!) to save, and the changes are pulled-in and persisted.

Remember the [impedance problem][imp], stating that your database schema won't be able to be a 1:1 representation of your object model? Core Data does a lot to lower the wall separating the two, but in the end you can't store file paths as `NSURL` but have to resort to storing bookmarks as `NSData` and provide custom accessors for the path anyway. 

Instead of pimping the managed object model until it does both quack like a duck from the domain _and_ adhere to Core Data's requirements, make two objects of it.

To map domain objects to Core Data objects, you use a [Data Mapper][datamapper]. To provide easy access to the mapped managed objects in terms of the domain, you can employ a [Repository][] as a façade to the Data Mappers.

For my domain, this would have the following implications:

* `ProjectRepository` provides high-level access to `Project` objects. It works like a collection but does database queries in the background.
* `Project` describes a project composed of paths for the domain. It is identified by a `ProjectId`. It has a list of `ProjectPath`s.
* `ManagedProject` is the root Core Data managed object.
* `ManagedProjectPath` is the managed object for paths associated with a project.
* `ProjectMapper` transforms `Project`s into `ManagedProject`s. Check if there's a project with `ProjectId` in the database, fetch it, then update its values. (Maybe this step can become simpler.) Update its associated paths, too. To do so, the mapper has to find out which path is new and add it to the Core Data store.[^1]

Sounds complicated? Compared to doing nothing at all (Core Data), it sure is.

Does it pay off?

Well, it keeps the architecture clean and truly separates the layers. If Core Data turns out to be the ultimate solution and you'll never ever switch away, well, maybe the initial cost of mapping isn't worth the theoretical benefit of a [clean architecture][clean].

I like to keep the architecture clean, but I also want to move fast to provide the next update rather sooner than later.

[imp]: /posts/2013/09/map-class-hierarchies-to-database-tables/
[datamapper]: http://martinfowler.com/eaaCatalog/dataMapper.html
[repository]: http://martinfowler.com/eaaCatalog/repository.html
[clean]: http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html

[^1]: Maybe the mapper won't have to do this, since I'm not advocating a dumb data store. I don't assemble a complex `Project` in the user interface and then simply push it to the database. Every single change is committed to the database. Create a project, save it. Add a path to a project, save it. The user interface will update first and send a message to the database asynchronously to store the new object. The interface doesn't wait for the database. This is some kind of **eventual consistency**, I suppose. Make two processes reflect the same changes without sync'ing with one another. This way, the mapper will only have to insert new paths, but not save a whole project tree.


# 2\. Pretend Core Data Doesn't Exist

The managed objects are only changed in infrastructure, but never anywhere else. To make changes outside infrastructure impossible, you can use read-only managed object contexts.

You can do two things:

1. Pretend your object isn't a Core Data managed object and just never ever change attribute values, or
2. introduce a protocol for the read-only properties and work with these instead of the classes defined for Core Data.

Pretending only gets you so far. I prefer to codify contracts through accessing policies. If an attribute is meant to be read-only, make it read-only. Since that's not possible if you only have a single interface definition, I'd favor the formal protocol-based solution to codify the read-only contract:

* `ProjectRepository` still provides high-level access, but its protocol is fulfilled by the rather dumb `CoreDataProjectRepository`, a wrapper around the standard ways to fetch Core Data objects. The resulting objects are returned, no mapping necessary.
* `Project` is a formal protocol which defines the attribute I'd like to read or write to. This means I have to write `id<Project> aProject` instead of `Project aProject` throughout my app. That's bearable. It defines, for example, `-(NSArray *)paths` so I can iterate over all the associated paths to display them in a list. Since `Project` is an aggregate, it'll sport methods like `-addPath:(id<ProjectPath>)path` to insert paths into its underlying collection.
* `ProjectPath`, too, is a formal protocol. The part of the domain which deal with project trees will be content with `ProjectPath` objects as leaves of the project tree. It doesn't have to know that the database connects `PathRecord`s to `ProjectPath`s.
* Talking about `PathRecord`: this object is tackled a totally different branch of the domain. When I talk about records, I talk about changing a word counter and saving its value. This happens in the background. It doesn't affect the project tree.
* `ManagedProject` is a managed object satisfying the `Project` protocol. That's what `CoreDataProjectRepository` fetches and returns. It has all the Core Data bells and whistles and knows about its child managed objects, `ManagedProjectPath`s. These child objects won't leak to the domain, though. They are fetched behind the scenes to assemble the array I need for the above mentioned `-(NSArray *)paths` method.
* `ManagedPathRecord` is trivial: it represents a day's record and consists of 24 individual counts. That's a database detail. I need not worry about this from the domain's protocol's point of view.

I wouldn't need to implement any actual mapping. That's a pro. The rest is kind of the same, only without real domain objects. They take some time to develop and test. In exchange for some hand-waving when it comes to identity questions ("What kind of object are you, really?"), I get time to spend on other tasks -- such as writing blog posts about it.


## Comparing the two Approaches

The difference between hiding infrastructure details in a formal protocol and mapping attributes to first-class objects is this: every change in the domain will ripple through to the infrastructure, because the Core Data managed objects have to satisfy the changed contract.

Say I'd like to provide a way to obtain the `-pathsCount`. That's a reasonable requirement for the `Project` aggregate. Since I can't add any domain-specific implementation logic in the protocol itself, I have to resort to changing the Core Data implementations.

Fully-fledged domain objects could simply delegate the child object count to the list of child objects. That's a feature infrastructure won't need.

I prefer approach (1): I don't save a lot of the effort but gain much more flexibility by decoupling domain from infrastructure instead.

I can make things a bit simpler:

**Ditch `NSManagedObject` subclasses where appropriate.** I think it's a good idea to create a `ManagedProject` subclass to provide convenience methods for project creation and inserting paths into the tree. 

**Force thinking in parallel processes.** When the user adds a path to a project, a new path node in the visible user interface project tree is displayed immediately and a `ManagedPath` object will be created. Persisting and displaying data work with two different representations of a path object. They are connected by a unique `PathId` which the domain also employs. All in all, there are three concerns: working with data (domain), persisting data (infrastructure), and displaying data (user interface).

Thinking "parallel" means I don't have to make a button create an entity in the database, tell the domain to retrieve a new object from this, and pull-in the result again. There's no need to use a view model which is derived from Core Data via the domain. I can create both the view model and the Core Data entity based on the same input. Synchronicity, then, is a matter of propagating changes in Core Data to the view. When all three layers know about the `PathId` of the object in question, propagating events is all there is to it.

It's possible I'd be better off using a SQLite wrapper like [FMDB][]. I wouldn't know. My approach enables me to swap the two in the end, though, should I come to the sudden realization that Core Data doesn't cut it.

I still have to wrap my head around parts of this, but I think it's a good way to get started coding something.

[fmdb]: https://github.com/ccgus/fmdb
