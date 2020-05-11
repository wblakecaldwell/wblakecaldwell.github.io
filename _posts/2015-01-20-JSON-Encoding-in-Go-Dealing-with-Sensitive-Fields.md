---
layout: post
post_number: 17
title: "JSON Encoding in Go: Dealing with Sensitive Fields"
categories: Go
---

JSON marshalling in [Go](http://golang.org/) is pretty 
[straight-forward](http://golang.org/pkg/encoding/json/#example_Marshal), but sometimes your structs contain sensitive 
information that you'd like to keep out of your public JSON API. Fortunately, there are a few good solutions that don't 
require you to manually copy every field from one struct type to another.

Let's start with our basic `Person` struct:

{% highlight golang %}
// Person struct that includes all fields in JSON
type Person struct {
	FirstName string
	LastName  string
	AuthToken string
}
{% endhighlight %}

The simplest way to prevent exposing `AuthToken` via JSON is with the `json:"-"` tag, which tells the 
[json](http://golang.org/pkg/encoding/json/) to always ignore this field. Let's call this `SafePerson`, because you 
never have to worry about exposing AuthToken via JSON:

{% highlight golang %}
// SafePerson omits AuthToken in JSON
type SafePerson struct {
	FirstName string
	LastName  string
	AuthToken string `json:"-"`
}
{% endhighlight %}

This works really well, until you need to include the field some of the time, like when showing someone their own record. 
In that case, you could try the `json:",omitempty"` tag, which only includes the field if it has a value. It'd be up 
to you to clear it out when it doesn't belong in the JSON. This is `PrettySafePerson`, because it's safe if you're 
careful:

{% highlight golang %}
// PrettySafePerson omits AuthToken if it doesn't have a value
type PrettySafePerson struct {
	FirstName string
	LastName  string
	AuthToken string `json:",omitempty"`
}
{% endhighlight %}

This is great, but... here's the thing. That one time you forget to clear an `AuthToken` for someone other than the 
Person that it represents might give you quite the headache. I'd rather err on the side of safety, where forgetting 
something means not showing data that I meant to.

Here's where we can take advantage of Go's [type embedding](https://golang.org/doc/effective_go.html#embedding). By 
embedding a `Person` inside another struct, we borrow all of the `Person`'s fields, and can override some without 
affecting the `Person`. We'll build our API with this pattern, which always omit sensitive fields unless told otherwise.

{% highlight golang %}
// PublicPerson decorates a Person, hiding AuthToken by default
type PublicPerson struct {
	*Person
	AuthToken string `json:",omitempty"`
}

// IncludeAuthToken unhides the Person's AuthToken
func (p *PublicPerson) IncludeAuthToken() {
	p.AuthToken = p.Person.AuthToken
}
{% endhighlight %}

`PublicPerson`'s `AuthToken` field overrides the embedded `Person.AuthToken`, and because of the `json:",omitempty"`
key, it'll be hidden from JSON unless you set it. `Person.AuthToken` still has the original value, you can include it 
in the JSON by calling `IncludeAuthToken()`.

[Try it out in the Go Playground](https://play.golang.org/p/Du5ztx6zjC)

[View Source on Github](https://gist.github.com/wblakecaldwell/f24a6ef7ab74c08bdec5)
