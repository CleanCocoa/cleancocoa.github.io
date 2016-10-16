---
title: Solve database mapping problems and the impedance problem of ActiveRecord
created_at: 2013-09-22 12:01:51 +0200
kind: worklog
tags: [ rubyonrails, database, software-architecture ]
image: "cti.gif"
vgwort: http://vg02.met.vgwort.de/na/7df0a32231e340599cd25c72d1deae4d
---


Some time ago, I read about database design and mapping object hierarchies to database tables.  Ruby on Rails' default approach is to use a technique called [Single-Table Inheritance][sti].  This design pattern has some drawbacks.  

In short, the domain model design should come first, proper database design second, and only last comes the decision how you persist your objects.  Ruby on Rails, while not prohibiting this approach, doesn't really encourage you to make informed decisions about these three steps.  I aim to tell you something about stage two and three so you're design decisions might improve.

I've made heavy use of my _[Zettelkasten](/posts/tags/zettelkasten)_ to compose this post.  The list of _Zettel_ I used you'll find in the footnotes.[^zettel]

[sti]: http://www.martinfowler.com/eaaCatalog/singleTableInheritance.html

## Database Normalization and the Impedance Problem

{% include figure.md src="/assets/blog/2013/sti.png" alt="STI" url="http://www.martinfowler.com/eaaCatalog/singleTableInheritance.html" caption="Single-Table Inheritance, image (c) Martin Fowler" %}

Single-Table Inheritance (STI) fills unused columns with `NULL` depending on the object which is persisted.  Look at the image above.  If you store a `Footballer` object into the database table `Players`, only the columns `name` and `club` will be used.  The two others will be `NULL`ed-out.

This ruins your **database normalization**.  The table `Players` would be in third normal form ...

> if, and only if, for all times, each row consists of a unique object 
> identifier together with a number of mutually independent attribute 
> values
> [273][#jacobson1990oose]

How could this be broken in STI?

Think about a column `sum_total` which stores the sum of columns `price` and `shipping`.  Database tables won't ensure that `sum_total` is updated once you changed `price`, hence you're prone to data inconsistency.  `sum_total` is **redundant** since it repeats information you already have.[cf 273][#jacobson1990oose]

Polluting the table with `NULL` values is adding dependency between the columns, too.  It doesn't make sense to fill in both `club` and `bowling average` for a single object or table row, although that combination is totally possible in your table.  Your code probably won't assign both attributes at once, one belonging to a `Footballer`, the other to a `Bowler` entity, but your database cannot be trained to adhere to the same principles.  STI introduces bad database design for the sake of coder's convenience.

The Ruby on Rails mentality to "let the database be dumb and the app be smart" isn't to be depended on:  [Nathan Long][foll] pointed out he switched frameworks and languages over the years but had to keep the database.  That's a good reason to worry about database design and sustainable data integrity first and consider shortcuts your current programming framework provides second.

Object-oriented programming encourages objects with complex attributes.  Objects have both primitive values and connections to other objects, the latter called "complex".  Relational Database Management Systems (RDBMS) like MySQL are only capable of storing primitive data types like integers and text, though.  Object's mutual attribute exclusivity can't be expressed in RDBMS, either.

This incompatibility is called **the impedance problem**.[269--283, notably 271][#jacobson1990oose]  You'd need to map complex attributes like object references to other tables.  A way out is to use Object Database Management Systems (ODBMS) to store objects and their relationships directly.  Another way is to improve your database design drastically, which I suggest you do.

[foll]: http://nathanmlong.com/2013/09/followup-to-better-single-table-inheritance/


## Solving the Impedance Problem with Good Design

{% include figure.md src="/assets/blog/2013/cti.gif" alt="CTI" url="http://www.martinfowler.com/eaaCatalog/classTableInheritance.html" caption="Class-Table Inheritance, image (c) Martin Fowler" %}

Your database design will need to differ from your domain model because class hierarchies cannot be expressed in databases easily and you wouldn't want to get the easy pie by throwing database integrity away, buying into redundancy.

Consider the image above.  The same class hierarchy as before is shown, but the database design is fundamentally different:  specifications of `Player` like `Bowler` get their own _details table_.  When you fetch a `Bowler` object from the database, you'd have to query both the `Players` and `Bowlers` table to get all the data.

This technique or design pattern is called [Class-Table Inheritance][cti] (CTI)[#fowler2009peaa][].  It solves the impedance problem in a sane way,[275f][#jacobson1990oose] utilizing [two-dimensional mapping][foll].

I'd call creating an [Entity--Attribute--Value Model][eav] (EAV) an _insane_ solution.  EAV maps all attributes to relationships.  Your object's table would consist of these columns:

<ol class="short">
<li><em>Entity</em>,</li>
<li><em>Attribute</em>, a &#8220;Foreign Key&#8221; reference to the attribute&#8217;s definition, and</li>
<li>the <em>Value</em> of the Attribute.</li>
</ol>

To me, this is a few miles too far from the domain model.  I wouldn't know how a database schema could be any more distanced from the objects it persists.

If you want to see some CTI applications, the Ruby gem [Sequel][] has an adapter for that.  There's [a post by Pablo Astigarraga][poteland] with good example code using the Sequel gem.  [Nathan Long][ctilong] on the other hand posted a minimal solution on top of Rails' `ActiveRecord`.


[cti]: http://www.martinfowler.com/eaaCatalog/classTableInheritance.html
[eav]: http://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model
[sequel]: http://sequel.rubyforge.org/rdoc-plugins/classes/Sequel/Plugins/ClassTableInheritance.html
[poteland]: http://blog.poteland.com/blog/2013/05/09/single-table-inheritance-vs-class-table-inheritance
[ctilong]: http://nathanmlong.com/2013/05/better-single-table-inheritance


### Mix all the Things into Repositories

It occured to me that you wouldn't need to persist every object in the same way.  If your design doesn't suck, your domain model is independent from persistency mechanisms like `ActiveRecord`.  Making use of a Repository object which hides the database from the domain and translates search criteria to database queries helps.[#fowler2009peaa][]

Some entities can be accessed and stored behind the curtains via Rails' `ActiveRecord`.  Others can be modeled according to the Class-Table Inheritance pattern.

The entities' repositories will be able to chose which persistency mechanism they utilize.  You can implement this nicely as a [Strategy][] which can be plugged into the Repositories.  Bonus: you can stub the repository strategy during tests.  I took the example from Eric Evans' _Domain-Driven Design_[#evans2006ddd][] and wrote [a Gist in Ruby][gh].


[strategy]: http://en.wikipedia.org/wiki/Strategy_pattern
[gh]: https://gist.github.com/DivineDominion/6658694

## Discussion

Ruby on Rails is all about convenience.  I was all in for that for a very long time.  Every book on software architecture I read tought me to do things different, tough.  I now consider Rails' `ActiveRecord` class an anti-pattern:  it mixes the responsibilities of persistency, data validation and more into one single fat model object.  It can work without problem in lots of cases.  But it can also cause trouble in others.

For example, `ActiveRecord` rewards you with a single point of data validation.  These objects not only check for proper data types (fail when user tries to assign a String to an Integer attribute) and _object consistency_.  They also check for database _table coherence_:  is the user name unique or has someone already taken it?

The Open Web Application Security Project (OWASP) suggests you [validate objects on three layers][val] in your web applications:

1.  Persistency/Database: validate against SQL injections, for example
2.  Presentation/Views: validate boundaries like character counts
3.  Domain Model: validate attribute consistency, ensure business rules are met

That's a lot more work on your side if you don't mix everything into one object!

It raises the likelihood of making future changes painless, though.  This seemingly complex three-layered design (compared to inheriting from `ActiveRecord::Base`!) adheres to the **Open/Closed Principle**, which says "a module should be open for extension but closed for modification."[#martin2000dpdp][]  You can exchange parts and, for as long as the parts implement a common interface, expect the system to continue running smoothly.

Your code benefits from the Repository-Strategy combo pattern I already introduced for the same reason.  We need to design software not only to be open for extension but for change, too, because along the way some things will eventually change.  Good object-oriented design prepares your application for change.[#metz2013oodrub][]

That's why they call the "[lock-in effect][lockin]" the lock-in effect:  once you're in, the cost of switching increases significantly.  Adhering to Rails' conventional way of implementing stuff can make you vulnerable against unexpected changes in the long run.

([Discuss on Hacker News](https://news.ycombinator.com/item?id=6426462))

[lockin]: http://en.wikipedia.org/wiki/Vendor_lock-in

## Conclusion

There are many different ways to approach software architecture.  Every angle seems to have its own pros and cons, leaving you to wonder what's right in your case.  You'll become a better designer if you learn about the alternatives. Becoming a better designer will improve your decisions when writing software because you will be able to make _informed decisions_.

Ruby on Rails lowers the bar of getting a walking skeleton up and running by magnitude.  It's great for prototyping and sketching web applications.  Rails is kind of open for extension, too:  you can ditch the training wheels of `ActiveRecord` and roll your own persistency layer, for example, without inhibiting a lot of pain.  I hope you learned why this might be a good decision:

- Database design comes after domain modelling but before programming persistency mechanisms.  Keep in mind that your database will eventually survive the code you're about to write.
- `ActiveRecord` mixes responsibilites, increasing domain model pollution and locking you into it's own way of doing things.  Don't decide on the easy path only because it's convenient.  That's not a well-informed but a clueless way to decide.
- Mapping object and class hierarchies to database tables can be hard.  You can solve the impedance problem with the Class-Table Inheritance pattern.  Again, Rails points to the easier solution of STI but wreaks havoc with your database design.

[val]: https://www.owasp.org/index.php/Data_Validation


[#jacobson1990oose]: Ivar Jacobson, Magnus Christerson, Patrik Jonsson, and Gunnar Övergaard (1990):  _Object-Oriented Software Engineering. A Use Case Driven Approach_, Wokingham: Addison-Wesley.

[#fowler2009peaa]: Martin Fowler (2009):  _Patterns of Enterprise Application Architecture_, Boston: Addison-Wesley.

[#evans2006ddd]: Eric Evans (2006):  _Domain-Driven Design. Tackling complexity in the heart of software_, Upper Saddle River, NJ: Addison-Wesley.

[#martin2000dpdp]: Robert C. Martin (2000):  _Design Principles and Design Patterns_, 2000. [Download PDF](http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)

[#metz2013oodrub]: Sandi Metz (2013):  _Practical object-oriented design in Ruby: an agile primer_, Upper Saddle River, NJ: Addison-Wesley.


[^zettel]: I pulled the following _Zettel_ to assemble this post:<ul>
<li><code>§201307291910 Persistency mechanisms can vary per entity</code></li>
<li><code>§201305161728 Validate multiple layers</code></li>
<li><code>§201305101927 STI denormalizes database tables</code></li>
<li><code>§201305101924 Database normalization</code></li>
<li><code>§201305101917 Impedance Problem</code></li>
<li><code>§201305091507 Class-Table Inheritance instead of STI</code></li>
<li><code>§201305031836 Open&#8211;Closed Principle</code></li>
<li><code>§201305012044 Entity&#8211;Attribute&#8211;Value Model</code></li>
</ul>
