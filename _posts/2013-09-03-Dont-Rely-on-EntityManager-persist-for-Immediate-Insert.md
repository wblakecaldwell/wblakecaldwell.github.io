---
layout: post
post_number: 8
title: "Java: Don't Rely on EntityManager.persist() for Immediate Insert"
categories: Java
---

I had always counted on 
[EntityManager's persist()](http://docs.oracle.com/javaee/6/api/javax/persistence/EntityManager.html#persist(java.lang.Object)) 
method to immediately insert entities. I would rely on this when writing database integration tests - I'd persist some records, then test my DAO methods to find them.

On my current project, I decided to add a configuration option to allow me to run my datbase integration tests on my development Oracle database rather than my embedded [HSQLDB](http://hsqldb.org) test database - just for an extra sanity check. The tests that tried to persist() and then retrieve  those new entities failed. Adding an entityManager.flush() method after the persist() invocations solved the issue.

...But why?

From [en.wikibooks.org](http://en.wikibooks.org/wiki/Java_Persistence/Persisting#Persist):

> The EntityManager.persist() operation is used to insert a new object into the database. persist does not directly insert the object into the database, it just registers it as new in the persistence context (transaction). When the transaction is committed, or if the persistence context is flushed, then the object will be inserted into the database.
> If the object uses a generated Id, the Id will normally be assigned to the object when persist is called, so persist can also be used to have an object's Id assigned. The one exception is if IDENTITY sequencing is used, in this case the Id is only assigned on commit or flush because the database will only assign the Id on INSERT. If the object does not use a generated Id, you should normally assign its Id before calling persist.

Here's how I wire up my entities' primary key:

{% highlight java %}
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
{% endhighlight %}

For my embedded HSQLDB database, the generation strategy is _[GenerationType.IDENTIY](http://docs.oracle.com/javaee/5/api/javax/persistence/GenerationType.html#IDENTITY)_, which relies on the database to generate an autoincrementing primary key for that row. This requires an insert, so the persist() immediately inserts in HSQLDB.

Oracle, on the other hand, uses a cross-table _[GenerationType.SEQUENCE](http://docs.oracle.com/javaee/5/api/javax/persistence/GenerationType.html#SEQUENCE)_ @Id generator, which doesn't require an insert, but the following SELECT:

{% highlight sql %}
select
  hibernate_sequence.nextval
from
  dual
{% endhighlight %}

This select is called immediately on persist() so that the EntityManager has an ID to assign the entity. That entity will only be inserted after a flush(), which is called automatically on transaction commit.

Long story short: If you're relying on your entity existing in the database after your call to persist(), but before the transaction commits, then call flush() first. Leave a comment justifying it, as manually calling flush is largely considered an anti-pattern akin to invoking the garbage collector. Delayed flush() calls give Hibernate the chance to perform more performant bulk updates.
