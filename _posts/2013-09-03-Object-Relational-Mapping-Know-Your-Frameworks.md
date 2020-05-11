---
layout: post
post_number: 9
title: "Object/Relational Mapping: Know Your Frameworks"
categories: Java
---

I've been working with [Hibernate](http://www.hibernate.org) for several years now, yet I learn something new about it all the time. The more time I spend with the framework, the more concerned I am about how it will be used by developers new to it.

Mirko Novakovic Alois Reitbauer nails it in a post about [O/R Mapping Anti-Patterns](http://www.developerfusion.com/article/84945/flush-and-clear-or-mapping-antipatterns/):

> The simplicity of the entrance into the world of O/R mapping however gives a wrong impression of the complexity of these frameworks. Working with more complex applications you soon realize that you should know the details of framework implementation to be able to use them in the best possible way. In this article, we describe some common anti-patterns which may easily lead to performance problems.

This is an echo of [Joel Spolsky's](http://www.joelonsoftware.com) warnings of the [Law of Leaky Abstraction](http://www.joelonsoftware.com/articles/LeakyAbstractions.html):

> The law of leaky abstractions means that whenever somebody comes up with a wizzy new code-generation tool that is supposed to make us all ever-so-efficient, you hear a lot of people saying "learn how to do it manually first, then use the wizzy tool to save time." Code generation tools which pretend to abstract out something, like all abstractions, leak, and the only way to deal with the leaks competently is to learn about how the abstractions work and what they are abstracting. So the abstractions save us time working, but they don't save us time learning.

Don't stop learning about a framework once you figure out how to use it - that's only the beginning.
