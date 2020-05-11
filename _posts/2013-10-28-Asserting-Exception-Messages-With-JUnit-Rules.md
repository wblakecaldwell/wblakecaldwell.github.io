---
layout: post
post_number: 11
title: "Asserting Exception Messages With JUnit Rules"
categories: Java
---

If you're not familiar with [JUnit's @Rule feature](https://github.com/junit-team/junit/wiki/Rules) for asserting exceptions in your tests, then read on - you're about to start using it.

## Assert Exception Type

It's very simple to assert that a given `type` of exception is thrown in a JUnit test case with the following:

{% highlight java %}
public void testExceptionTypeIsThrown()
{
    throw new RuntimeException("Error!");
}
{% endhighlight %}

## Assert Exception Message (The Old Way)

But, what if you want to be more specific, and check the message itself? I've always done the following:

{% highlight java %}
/**
 * Test the exception message the old way.
 */
@Test
public void testExceptionMessageTheOldWay1()
{
    String message = null;
    try
    {
        throw new RuntimeException("Error!");
    }
    catch(Exception e)
    {
        message = e.getMessage();
    }
    Assert.assertEquals("Error!", message);
}
{% endhighlight %}

Heres's another variant you're probably familiar with:

{% highlight java %}
/**
 * Test the exception message the old way.
 */
@Test
public void testExceptionMessageTheOldWay2()
{
    try
    {
        // this string test was necessary to avoid an "unreachable code" warning in Eclipse
        if ("A".equals("A"))
        {
            throw new RuntimeException("Error!");
        }
        Assert.fail("Exception was expected.");
    } catch (Exception e)
    {
        Assert.assertEquals("Error!", e.getMessage());
    }
}
{% endhighlight %}

## Assert Exception Message With JUnit Rules

The above methods always felt like hacks. I recently came across [JUnit's @Rule feature](https://github.com/junit-team/junit/wiki/Rules), which saves tons of code and is much easier to read. You first define your public ExpectedException instance, and give it a @Rule annotation. Then, in each test case that wants to use it, you set what type of exception you're expecting, and optionally a substring to look for in the exception message:

{% highlight java %}
/**
 * Our expected exception rule. This must be public.
 */
@Rule
public ExpectedException exception = ExpectedException.none();

/**
 * Test the exception message with a single expectMessage substring match.
 */
@Test
public void testExceptionIsThrownFullText()
{
    exception.expect(RuntimeException.class);
    exception.expectMessage("Error!");

    throw new RuntimeException("Error!");
}
{% endhighlight %}

Since `expectMessage` is looking for substrings, you can use several of them to test more complicated exception messages:

{% highlight java %}
/**
 * Our expected exception rule. This must be public.
 */
@Rule
public ExpectedException exception = ExpectedException.none();

/**
 * Test the exception message with multiple expectMessage substrings.
 */
@Test
public void testExceptionIsThrownMultipleSubstrings()
{
    exception.expect(RuntimeException.class);
    exception.expectMessage("Error: There was an issue processing ");
    exception.expectMessage(" of the notes.");

    throw new RuntimeException("Error: There was an issue processing 5 of the notes.");
}
{% endhighlight %}

## More Advanced: Custom Matchers

In [my next post]({% post_url 2013-10-29-Asserting-Exceptions-With-Junit-Rules-Custom-Matchers %}), I'll i
describe how to implement a custom 
[Matcher](https://code.google.com/p/hamcrest/wiki/Tutorial) for more complicated
Exception assertions.
