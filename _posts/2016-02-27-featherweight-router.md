---
title: Featherweight Router
created_at: 2016-02-27 22:41:19 +0100
kind: worklog
tags: [ routing ]
url: https://karlbowden.com/featherweight-router-tldr/
preview: fulltext
comments: on
---

I read Karl Bowden's [Featherweight Router TL;DR](https://karlbowden.com/featherweight-router-tldr/) and kind of like how routes are handled. The setup looks complicated, though:

    #!swift
    let someRouter = Router(aDelegate).route("animal", children: [
        Router(aDelegate).route("animal/mammal", children: [
            Router(aDelegate).route("animal/mammal/cow"),
            Router(aDelegate).route("animal/mammal/pig"),
        ]),
        Router(aDelegate).route("animal/fish", children: [
            Router(aDelegate).route("animal/fish/salmon"),
            Router(aDelegate).route("animal/fish/trout"),
        ]),
    ])

    // Using regex's for matching
    let someRouter = Router(aDelegate).route("animal", children: [
        Router(aDelegate).route("animal/\\w+", children: [
            Router(aDelegate).route("animal/\\w+/\\w+")
        ])
    ])

Child routers carry the path component of the parent with them. Also, this is very verbose. I imagine a more terse version, like:

    #!swift
    // Assuming `aDelegate` is used for all of them and passed along,
    // let's omit it during setup of sub-routes:
    let router = Router(aDelegate) { router
        router.route("animal") { router in
            router.route("mammal") { router in
                router.route("cow")
                router.route("pig")
            }
        }
    }

Or let's say some part is variable, like viewing a user profile:

    #!swift
    let userRouter = Router(aDelegate) { router 
        router.route("users") { router in
            router.route(.Parameter("userId", "[0-9]+")) { router in
                router.route("friends")
                router.route("photos")
            }
        }
    }

Then the route `users/12345/friends` will parse and the "12345" part will be available in a `params` dictionary under the key `userId`.

I haven't given routers much though in the past but I think I'll explore the possibilities of routing and existing libraries some more.
