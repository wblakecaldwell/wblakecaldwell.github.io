---
layout: post
post_number: 7
title: "Builder Pattern Instead of Error-Prone Constructors"
categories: [Java, Software Design]
---

When designing your classes, rather than following the path of the [unreadable telescoping constructor](http://www.captaindebug.com/2011/05/telescoping-constructor-antipattern.html), or leaving yourself open for bugs where the caller incorrectly passes a value into the wrong parameter because you have several of the same type, consider the following [Builder pattern](http://en.wikipedia.org/wiki/Builder_pattern).

{% highlight Java %}
public class Person
{
  // required
  private String firstName;
  
  // required
  private String lastName;
  
  // optional
  private String url;

  // private constructor to defer responsibility to Builder.
  private Person()
  {
  }
  
  public String getFirstName()
  {
    return this.firstName;
  }
  
  public void setFirstName(final String firstName)
  {
    if(StringUtils.isBlank(firstName))
    {
      throw new IllegalArgumentException("Person.firstName must not be blank");
    }
    this.firstName = firstName;
  }

  public String getLastName()
  {
    return this.lastName;
  }

  public void setLastName(final String lastName)
  {
    if(StringUtils.isBlank(lastName))
    {
      throw new IllegalArgumentException("Person.lastName must not be blank");
    }
    this.lastName = lastName;
  }

  public String getUrl()
  {
    return this.url;
  }
  
  public void setUrl(final String url)
  {
    this.url = url;
  }

  // builder class to ensure that all required fields are set while avoiding clunky, "telescopic" constructors
  public static class Builder
  {
    private Person built;

    public Builder()
    {
      built = new Person();
    }

    public Builder setFirstName(final String firstName)
    {
      built.setFirstName(firstName);
      return this;
    }

    public Builder setLastName(final String lastName)
    {
      built.setLastName(lastName);
      return this;
    }

    public Person build()
    {
      if(built.firstName == null)
      {
        throw new IllegalStateException("Person.firstName is required");
      }
      if(built.lastName == null)
      {
        throw new IllegalStateException("Person.lastName is required");
      }
      return built;
    }
  }
}
{% endhighlight %}


Having a private constructor will prevent anyone but the Person's Builder from instantiating a Person. The setters in the Builder return the Builder, which allows [method chaining](http://en.wikipedia.org/wiki/Method_chaining), providing a [DSL](http://en.wikipedia.org/wiki/Domain-specific_language)-like [self-documenting](http://en.wikipedia.org/wiki/Self-documenting) interface. The Builder's build() method is responsible for making sure that all properties are set before returning the Person.

Of course, this is a simple example with only two fields, so the alternative isn't exactly error prone:

{% highlight Java %}
Person blogger = new Person("Blake", "Caldwell");
blogger.setUrl("http://blakecaldwell.org");
{% endhighlight %}

But, once you add a few more required fields, the following makes your code easier to follow, while still ensuring all required fields are set.

{% highlight Java %}
Person blogger = new Person.Builder()
    .setFirstName("Blake")
    .setLastName("Caldwell")
    .build();
blogger.setUrl("http://blakecaldwell.org");
{% endhighlight %}

I first came across this pattern via a [post](http://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-part-two-crud/) by [Petri Kainulainen](http://www.petrikainulainen.net). Check out [his blog](http://www.petrikainulainen.net/blog/) for great posts about [Spring Data JPA](http://www.petrikainulainen.net/spring-data-jpa-tutorial/) and other topics.
