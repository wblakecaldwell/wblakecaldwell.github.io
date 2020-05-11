---
layout: post
post_number: 14
title: "Java: Exercising Caution With Hibernate Entities"
categories: Java
---

I heavily rely on the [Hibernate framework](http://www.hibernate.org) on all of my database-driven Java 
projects. It's a full-featured cross-database [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping)
framework capable of managing your [database schema](http://en.wikipedia.org/wiki/Database_schema),
persisting your [entities](http://docs.oracle.com/javaee/5/tutorial/doc/bnbqa.html) to the database,
populating entities from the database, 
[caching queries, caching entities](http://www.javalobby.org/java/forums/t48846.html), registering entities 
for [lifecycle events](http://docs.jboss.org/hibernate/entitymanager/3.6/reference/en/html/listeners.html),
indexing them in a [search index](http://www.hibernate.org/subprojects/search.html),
and just about everything else you could ask of an ORM.

When you fetch an entity from Hibernate, it remains attached to your <a href="http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html">EntityManager</a> for the duration of that transaction. The EntityManager is responsible for all of the magic - it watches the entity for changes, persisting it and calling lifecycle methods when appropriate, makes sure to return the same instance of an entity for multiple queries to the same record, builds and executes additional queries if you access properties that point to associations in the database, and of course much more.

_Take a look at my [hibernate-perf-test](https://github.com/wblakecaldwell/hibernate-perf-test)
sample project to see the relative performance of different Hibernate query strategies._

## Entities are Expensive!

Entities are convenient, but if you don't understand what they're doing, you can get yourself into trouble. 
It all boils down to the fact that an entity's getter method may run several more queries, so long as the 
entity is still attached to the EntityManager. A developer can be excused at first, because we're used to 
getters and setters containing little to no logic besides accessing a private member variable. Hibernate is 
an amazing framework, but its greatest strength: being easy to use, becomes a liability: it's too easy to 
use it to thrash your database.

The most common and expensive errors that I see are due to:

## Accidental "eager" wiring of associated entities

When you load a Customer that has a list of Orders that are wired "eagerly", Hibernate will build and execute a query to fetch all of those Orders, even if the code never accesses that list. The Orders can have their own eager associations, which compounds this problem. This becomes performance-crippling when your User table is eagerly self-referencing, and just about every query loads up your entire database. This is the sort of issue you might not notice during development, with 10 records in your database, but believe me, it shows itself in production.

<h3>Too many queries by accessing an association while looping over entities</h3>

When you're displaying a table of 20 User records, and each one needs the name of their Organization, the 
easiest path for that developer is to loop over the list of entities and access the "organization" property on 
each User. This is intuitive, and makes sense with how we think about objects - you have a User, the User 
has an Organization, and the Organization has a name. However, with Hibernate, you need to understand that 
Organization isn't loaded until you access it. By fetching each User's Organization this way, you're 
producing at least 20 more queries. Now, imagine if the user chooses a table size of 50 rows, or if 
Organization had some eagerly-loading associations to surprise you with (like another User record!). 
You've just killed your action.

This comes up a lot when a developer tries to do the right thing and adds a lot of logging. Even if we don't 
need the Organization for the data table we're building, the developer might think that someone that reads the 
logs might want to see each User's organization. Those logs just became very expensive, and nobody will notice 
until your users are complaining about page load times in production.

## Yes, there are better ways to use entities

If you're familiar with the framework, you're probably shaking your head, mumbling something about 
"[fetch joins](http://docs.jboss.org/hibernate/orm/3.3/reference/en-US/html/queryhql.html#queryhql-joins),
[closing the transaction before building your view](https://developer.atlassian.com/display/CONFDEV/Hibernate+Sessions+and+Transaction+Management+Guidelines),
and other strategies for more efficiently fetching entities. My point is that as much as I love entities, 
they're performance time bombs. It takes one careless moment, or one developer that doesn't know Hibernate as 
well as you do, to touch the wrong property and cause a big issue to ripple through your system.

## Querying Carefully

Rather than maintain constant vigilance and make sure that every member of your team has read all of the 
documentation, I prefer to tread lightly with Hibernate. I fully embrace it for helping me 
[generate my schema](http://blog.iprofs.nl/2013/01/29/hibernate4-schema-generation-ddl-from-annotated-entities/), for updating the database in an infrequent-write system, and for the 
[query language](https://docs.jboss.org/hibernate/orm/3.3/reference/en-US/html/queryhql.html)
that [bridges different databases](https://community.jboss.org/wiki/SupportedDatabases2).

There are several ways you can use Hibernate to query your database, from full-on magic entities to getting 
your hands dirty for better efficiency. In a system where reads are common and writes rare, my approach is 
typically to use as little "magic" as I can for my read-only queries. I don't fetch attached entities from 
the database, but rather, select the specific fields that I need, and use the results to build detached data 
transfer objects ([DTOs](http://en.wikipedia.org/wiki/Data_transfer_object)).

Aside from avoiding the big issues above, this also forces a developer to think more in terms of SQL, and 
where their data is coming from - it's more explicit, and thus, more understandable. If you have a "dumb" 
User DTO, and want the name of that User's Organization, you're going to have to either run a query on each 
User, query them in bulk and then join the two sets of data, or add the Organization name to the original 
User query. It's immediately obvious that the first option is ridiculous, and that the second one is annoying 
to write. The last one takes the least effort, and makes sense when you're thinking in terms of SQL and 
database access.

Here's another point to consider. Since it's typically considered poor form to return your entities to the 
client or view, then why not avoid the entity-to-DTO conversion and generate the DTOs in the first place?

## Coming Up: Performance Testing Different Query Approaches

There are several ways to use Hibernate to fetch data from your database, and 
[no](http://stackoverflow.com/questions/5155718/hibernate-performance)
[shortage](http://stackoverflow.com/questions/2764054/usual-hibernate-performance-pitfall)
[of](http://stackoverflow.com/questions/10319205/hibernate-performance-best-practice)
[conversations](http://stackoverflow.com/questions/485331/hibernate-performance-tweaks) 
[online](http://stackoverflow.com/questions/9602726/hibernate-performance-tuning) about the performance of
these different approaches, as well as 
[official documentation on the subject](http://docs.jboss.org/hibernate/orm/3.3/reference/en-US/html/performance.html).

In my [next post]({% post_url 2013-11-10-Performance-Testing-Hibernate-Query-Approaches %}), I'll walk you 
through a [sample project](https://github.com/wblakecaldwell/hibernate-perf-test) that I wrote to test 
out different query strategies. If you're interested, take a look for yourself. 
