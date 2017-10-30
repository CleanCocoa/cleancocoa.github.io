---
title: "Dependency Injection via the Recent \"Cake Pattern in Swift\" Pattern is Useless in Practice"
created_at: 2017-10-30 16:11:41 +0100
kind: worklog
tags: [ swift, oop ]
comments: on
---

Dependency Injection means you do not create objects where you use them but "inject" objects a function/method/object depends on from outside. This is a useful trick to make code testable, for example. Once you adopt the mindset, it becomes second nature.

From the Java world stems the imperative to program against interfaces, not implementations. In Swift, this means to define injected dependencies not as concrete types but as protocols.

In an ongoing object collaboration, you will want to store the dependency as a property. The storage of the dependency is up to the component itself. Now the, let's say, "injectability" can be lifted into a protocol, too. Then you can provide a protocol extension with a default implementation. That's what Peter-John did in ["Dependency Injection with the Cake Pattern"](https://medium.com/swift-programming/dependency-injection-with-the-cake-pattern-3cf87f9e97af) value.

The problem with this approach is that it has 0 real world value and is totally misleading because with that code you will not be able to actually inject anything; you'll be hiding object creation in a protocol extension only.

Dependency injection is a useful pattern because it allows you to change the concrete object passed in. As I mentioned, apart from the final object in your production code, tests will benefit from injections a lot. When you do not write tests and never pass in another object in production code, you do not need dependency injection. "But it makes my code cleaner!", people arguing based on principles will say. Yeah sure, it reads like cleaner code, but it's useless code written like useful code in a "clean" manner. Getting rid of that code is more important than sticking the dependency injection label on it.

Look at the sample code from Peter-John. Here is the dependency as a protocol and some default implementation:

```swift
protocol ProductsRepository {
    func fetchProducts() -> [Product]
}

struct ProductsRepositoryImplementation: ProductsRepository {
    func fetchProducts() -> [Product] { return [] }
}
```

And here is the "having a dependency"-protocol, which is the result of what I called "lifting injectability into a protocol":

```swift
protocol ProductsRepositoryInjectable {
    var products : ProductsRepository {get}
}

extension ProductsRepositoryInjectable {
    var products : ProductsRepository {
        return ProductsRepositoryImplementation()
    }
}
```

This protocol with its extension will help you get rid of writing client code like this:

```swift
class ClientLong {
    let products: ProductsRepository

    // Inject dependency in initializer:
    init(products: ProductsRepository) {
        self.products = products
    }
    
    func printPrices() {
        self.products.fetchProducts().forEach {
            print("This \($0.name) costs R\($0.price)")
        }
    }
}
```

Instead you can start with a shorter implementation of `ProductsRepositoryInjectable`:

```swift
class ClientShort: ProductsRepositoryInjectable {
    func printPrices() {
        self.products.fetchProducts().forEach {
            print("This \($0.name) costs R\($0.price)")
        }
    }
}

ClientShort().printPrices()

/*
 This Adidas Sneakers costs R2030.0
 This Nike Sneakers costs R1000.0
 */
```

There you have it: less code, using cleverly conceived protocols. 

There are two very obvious problems: 

- **Computed properties** don't work well for non-trivial object creation, for example if setting up the dependency in turn depends on some other dependency; you will want to store the objects. So a `*Injectable` protocol with a default extension might not always work.
- **No injection**, but lots of hand-waving: it doesn't help a bit when you actually want to reap any benefits of dependency injection, like, _injecting a dependency_. **The code as cited does not allow injection of dependencies. It only hides object creation.** To change the dependency in tests, you will need to define a property and override it somehow, via initializer injection or by making the property mutable internally. In any case, you will end up with code similar to `ClientLong`.

That's when I concluded that this approach is not going to help you improve your code at all. There has to be more to make it useful in practice. All it does now is shorten the code of a contrived example to impress the reader.

A comment on the original post asked how to use this to inject mocks in tests, and [Peter-Johns answer](https://medium.com/@pjwelcome/hi-lehlohonolo-c763f7a3fa7b?source=responses---------0----------------) was along the lines of: use a dependency container singleton to create a testing seam. The protocol extension would then return `DependencyContainer.instance.productRepository` instead of creating a new instance. Actually providing a testing seam is an improvement. It has nothing to do with the pattern, though.

What you will want is code similar to `ClientLong` to keep the mechanism of injecting dependencies _local_. Then you use the initializer directly to inject your mock repository: `ClientLong(products: mockRepository)`. You can do this a million times and it will Just Work. But forget to reset a dependency container singleton _once_ and a lot of other tests that use the container will suffer and may even break. It's also a lot less straightforward to read and understand than the initializer, thus increasing maintenance cost. If you think that accidentally breaking unit tests is a sign of bad tests, you're right, and writing atomic unit tests for very cohesive objects is an obvious antidote -- and now we've done a full circle, looking at unit tests for `ClientShort` which suffer from the very same problem.

A verdict from the author himself in the comment section on Medium is: this is only part 1 to show the basic technique. But, as I argued, it does not show actual dependency injection. It hides object creation in a protocol extension only.

If you take away anything from my post here, it should be that nice-looking samples do not always convey meaningful information, and that slapping "Pattern" onto something does not make it awesome. Or even useful.
