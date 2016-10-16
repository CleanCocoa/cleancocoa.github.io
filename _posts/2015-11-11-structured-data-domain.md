---
title: "Modeling: From Structured Data Representation to Problem Domain"
created_at: 2015-11-11 09:00:44 +0100
kind: worklog
tags: [ domain, database, modeling, oop ]
shared_image: "blog/201511111020_clay-t.jpg"
comments: on
---

Recently, I was discussing adding a feature to an application which is about event creation and booking. The project manager has a strong database background and works on the schema himself. The schema is also his go-to tool to express changes in the features. But thinking about the real Domain in terms of objects revealed much, much more.

Imagine a schema which kind of looks like the following diagram:

{% include figure.md src="/assets/blog/2015/events.png" alt="Event schema" caption="A schema showing the details of events in the database" %}

Events have a limited number of seats. Users can book seats at events. Once the user pays, his order is added to the database and the remaining number of seats should decrease. -- This was the new feature, expressed through the `seatsLeft` row in the `Event` schema.

We don't have performance problems, so I didn't see why we should mutate an existing event in the database when a booking takes place to decrease the number of `seatsLeft`. Also, I figured that concurrent access could cause problems -- problems which the DBMS will probably take care of in transactions.

{% include figure.md src="/assets/blog/201511111020_clay.jpg" alt="clay" url="https://www.flickr.com/photos/suckamc/161380717/in/photostream/" caption="Photo credit: <a href=\"https://www.flickr.com/photos/suckamc/161380717/in/photostream/\">Blob o' clay</a> by Martin Cathrae. License: <a href=\"https://creativecommons.org/licenses/by-sa/2.0/\">CC BY-SA 2.0</a>" %}

I'm a developer, and I'm an [object thinker](/posts/2015/10/domain-driven-design-meaning/), it seems. Capturing the problem domain by the means of a database schema (solution space/implementation concern) is a bad idea because lots of information get lost.

In the discussion, I proposed a different kind of representation. In essence, it boiled down to this:

> PM: _"When ordering, you can choose how many seats you want"_
>
> Me: "Didn't know that. Could cause trouble faster. The checkout process may take some time. If `seatsLeft` is checked by the client before ordering and by the server before accepting an order, this number may have changed in the meantime."
>
> _"Yes. We definitely need to check twice before accepting."_  
>
> "We could also async check for `seatsLeft` during order process and see if the current booking settings still work to make UX better."  
>
> _"Good point."_  
>
> "Seems unfair if looking for a password for too long will ruin the order though. What about a **Reservation** for an **Event** which expires after a given **Timeout**? The server prunes reservations automatically."
>
> _"So the actual number of seats left = total seats - bookings - reservations. Sounds good!"_

I created this initial model sketch from that discussion, shown here:

{% include figure.md src="/assets/blog/2015/domain-model.png" alt="Domain model" caption="Sketch of objects with real behavior in the problem space" %}

Up next came concerns about representing this in the schema and server logic:

> _"So when a user completes an order with all details, do we need to create a new row in `Reservation` or `Booking`?"_

Well, this depends on the _context:_ the server will not have the exact same problem and solution space as the client apps. In fact, a client will not even know about a `Reservation` because it's just the server's way to capture an active session.

The actual relation of objects in the server's code could be different. Instead of a `Booking` replacing a `Reservation` once the order is complete, it could be better to keep a reservation around next to a booking and simply remove the timeout. So there are temporary reservations and final reservations. A final reservation is one which is booked: the customer has paid and completed the process.

{% include figure.md src="/assets/blog/2015/domain-model2.png" alt="2nd domain model diagram" caption="Another model where Reservations are not temporary. Captures how we use the term better: a reservation at a restaurant doesn't go away when we arrive." %}

Of course, `TemporaryReservation` is just an object waiting to see the light. So is the `BookingRequest` on the client side. These things are details of both contexts.

In the database, a `Booking` table could suffice to capture both concepts. Reducing JOIN operations without violating normalization too much can be an objective of the backend guys:

{% include figure.md src="/assets/blog/2015/booking.png" alt="schema variants" caption="Three alternative schema representations" %}

Variant "C" makes a lot less of the business rules explicit. A temporary reservation is saved as a `Booking` with a timeout; a final reservation will need the timeout to be removed, or let the server not remove rows with full details, or whatever.

This discussion was very fun and revealed a lot of the application of object thinking to me. This stuff works, makes talking about problems easier -- and thanks to the "object as person" metaphor reveals a lot of user stories in the meantime.
