---
layout: post
post_number: 13
title: "Asserting Exceptions With JUnit Rules: IsEqual Matcher"
categories: Java
---

In my [first post on asserting exceptions with JUint]({% post_url 2013-10-28-Asserting-Exception-Messages-With-JUnit-Rules %}),
I showed how to test that a specific type of exception is thrown with expected substrings in the error message. 
[In my second post]({% post_url 2013-10-29-Asserting-Exceptions-With-Junit-Rules-Custom-Matchers %}),
I showed how we can write a custom [Matcher] to inspect the contents of the exception. In this post, I'll show you 
how to take advantage of the stock 
[IsEqual](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/core/IsEqual.html)  matcher to accomplish the 
same task, but with less work.

## IsEqual Matcher

This Matcher is straightforward - it evaluates whether two objects are equal by calling 
[equals(Object)](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#equals(java.lang.Object))
on one of the objects that isn't null, passing in the other. So, to use it with our custom exception, we'll need
to make sure that our `equals(Object)` method correctly evaluates two of its instances.

The default behavior of `equals(Object obj)` is to check if 
[two objects are the same instance](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#equals(java.lang.Object)):

> The equals method for class Object implements the most discriminating possible equivalence relation on objects; that is, for any non-null reference values x and y, this method returns true if and only if x and y refer to the same object (x == y has the value true).

This won't help us here, because we'll be comparing exceptions thrown in our code with one we instantiated in our unit test. We're going to have to implement `equals(Object)` ourselves.

## Implementing `equals(Object)`

Here's my simple custom exception:

{% highlight java %}
/**
 * Error Code Exception - stores the error code that was generated
 * when this exception was thrown.
 */
public class ErrorCodeException extends RuntimeException
{
    private static final long serialVersionUID = 1L;

    /**
     * The error code that triggered the exception.
     */
    private int errorCode;

    /**
     * Constructor.
     * 
     * @param errorCode
     *            the error code that triggered the exception
     */
    public ErrorCodeException(int errorCode)
    {
        this.errorCode = errorCode;
    }

    /**
     * Get the error code that triggered the exception.
     */
    public int getErrorCode()
    {
        return errorCode;
    }
}
{% endhighlight %}

You have to be careful when implementing either `equals(Object)` or 
[hashCode()](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode()). The rule is that if you 
implement either of these methods, then you need to implement both. Two objects that are equal must have the same 
[hash code](http://en.wikipedia.org/wiki/Java_hashCode()). If they don't, and you try to add two distinct, but 
equal instances of a class to a [Map](http://docs.oracle.com/javase/7/docs/api/java/util/Map.html), they'd both be 
accepted. This is because internally, the 
[Map stores the items in 'buckets' based on their hash codes](http://eclipsesource.com/blogs/2012/09/04/the-3-things-you-should-know-about-hashcode/), 
and then only has to check equality within a bucket. If two objects have different hash codes, then they won't 
be compared to each other.

Your IDE should have the ability to generate what we need here. If you're using 
[Eclipse](http://www.eclipse.org) ([I recommend the STS version](http://spring.io/tools), right-click in the 
source file, select _"Source"_, and the select _"Generate hashCode() and equals()"_.

![Generate hashCode() and equals() menu](/assets/posts/13/generate hashCode and equals.png)


After selecting that option, choose which private members will be used in the two methods. I recommend selecting 
_"'Use blocks in 'if' statements"_ in order to help 
[wrong code look wrong](http://www.joelonsoftware.com/articles/Wrong.html), should someone modify these methods down the road.

![Generate hashCode() and equals()](/assets/posts/13/select fields for hashCode and equals.png)

Here's our final `ErrorCodeException` class with the newly generated code:

{% highlight java %}
package org.blakecaldwell.tests.junitrules;

/**
 * error code exception - stores the error code that was generated
 * when this exception was thrown.
 */
public class errorcodeexception extends runtimeexception
{
    private static final long serialversionuid = 1l;

    /**
     * the error code that triggered the exception.
     */
    private int errorcode;

    /**
     * constructor.
     * 
     * @param errorcode
     *            the error code that triggered the exception
     */
    public errorcodeexception(int errorcode)
    {
        this.errorcode = errorcode;
    }

    /**
     * get the error code that triggered the exception.
     */
    public int geterrorcode()
    {
        return errorcode;
    }

    /**
     * (non-javadoc)
     * 
     * @see java.lang.object#hashcode()
     */
    @override
    public int hashcode()
    {
        final int prime = 31;
        int result = 1;
        result = prime * result + errorcode;
        return result;
    }

    /**
     * (non-javadoc)
     * 
     * @see java.lang.object#equals(java.lang.object)
     */
    @override
    public boolean equals(object obj)
    {
        if (this == obj)
        {
            return true;
        }
        if (obj == null)
        {
            return false;
        }
        if (getclass() != obj.getclass())
        {
            return false;
        }
        errorcodeexception other = (errorcodeexception) obj;
        if (errorcode != other.errorcode)
        {
            return false;
        }
        return true;
    }
}
{% endhighlight %}

## Verifying `_equals(Object)` and `hashCode()`

Even though we generated this code, we still need to test it. Here's the test fixture for `ErrorCodeException`:

{% highlight java %}
package org.blakecaldwell.tests.junitrules;

import org.junit.Assert;
import org.junit.Test;

/**
 * Test fixture for ErrorCodeException.
 */
public class ErrorCodeExceptionTest
{
    /**
     * Test getErrorCode().
     */
    @Test
    public void testGetErrorCode()
    {
        ErrorCodeException exception = new ErrorCodeException(500);

        Assert.assertEquals(500, exception.getErrorCode());
    }

    /**
     * Test equals(Object) evaluates true for two equal instances.
     */
    @Test
    public void testEqualsWhenEqual()
    {
        ErrorCodeException exception1 = new ErrorCodeException(500);
        ErrorCodeException exception2 = new ErrorCodeException(500);

        Assert.assertTrue(exception1.equals(exception2));

        // another way to test
        Assert.assertEquals(exception1, exception2);
    }

    /**
     * Test equals(Object) evaluates as false for two unequal
     * instances.
     */
    @Test
    public void testEqualsWhenNotEqual()
    {
        ErrorCodeException exception1 = new ErrorCodeException(500);
        ErrorCodeException exception2 = new ErrorCodeException(501);

        Assert.assertFalse(exception1.equals(exception2));

        // another way to test
        Assert.assertNotEquals(exception1, exception2);
    }

    /**
     * Test equals(Object) evaluates as false when we pass in null.
     */
    @Test
    public void testEqualsWhenNull()
    {
        ErrorCodeException exception1 = new ErrorCodeException(500);

        Assert.assertFalse(exception1.equals(null));

        // another way to test
        Assert.assertNotEquals(exception1, null);
    }

    /**
     * Test that two equal objects have the same hash code. Note that
     * two different objects don't need to have different hash codes.
     */
    @Test
    public void testHashCodeForTwoEqualObjects()
    {
        ErrorCodeException exception1 = new ErrorCodeException(500);
        ErrorCodeException exception2 = new ErrorCodeException(500);

        // sanity check - make sure they're equal
        Assert.assertEquals(exception1, exception2);

        // make sure they have the same hash code
        Assert.assertEquals(exception1.hashCode(), exception2.hashCode());
    }
}
{% endhighlight %}


## Using the IsEqual Matcher In Our Unit Tests

Now that we've implemented `equals(Object)` and `hashCode()` for our custom exception, we can use the `IsEqual` Matcher to setup an expectation 
for a specific exception.

{% highlight java %}
package org.blakecaldwell.tests.junitrules;

import org.hamcrest.core.IsEqual;
import org.junit.Assert;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ExpectedException;

/**
 * Test fixture to demonstrate JUnit's ExpectedException @Rule with an
 * IsEqual matcher.
 */
public class ExpectedExceptionRuleIsEqualMatcherTest
{
    /**
     * ExpectedException must be public.
     */
    @Rule
    public ExpectedException exception = ExpectedException.none();

    /**
     * Assert that we're throwing the exact exception that we're
     * expecting by using an IsEqual Matcher.
     */
    @Test
    public void testErrorCode500UsingIsEqualMatcher()
    {
        ErrorCodeException expectedException = new ErrorCodeException(500);

        exception.expect(new IsEqual&lt;ErrorCodeException&gt;(expectedException));

        // Run your code that you expect will throw an
        // ErrorCodeException with error code 500. If this exception
        // is equal to our expectedException above, this test will
        // pass.
        throw new ErrorCodeException(500);
    }

    /**
     * Here's the old way of asserting a specific exception. Yuck!
     */
    @Test
    public void testErrorCode500WithoutJUnitRules()
    {
        ErrorCodeException exception = null;
        try
        {
            throw new ErrorCodeException(500);
        }
        catch (Exception ex)
        {
            exception = (ErrorCodeException) ex;
        }
        Assert.assertEquals(500, exception.getErrorCode());
    }
}
{% endhighlight %}

In the first test, I create an `IsEqual` Matcher with the exception that I want to compare the thrown exception to. No custom Matcher was required, 
and my custom exception is now more useful because of it.

In my second test, I include the 'old way' of checking exceptions to demonstrate how much easier and more readable exception tests are when using 
JUnit's Rules feature.
