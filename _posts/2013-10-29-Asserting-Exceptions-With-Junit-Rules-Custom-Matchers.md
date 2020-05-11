---
layout: post
post_number: 12
title: "Asserting Exceptions With JUnit Rules: Custom Matchers"
categories: Java
---

In my [previous post]({% post_url 2013-10-28-Asserting-Exception-Messages-With-JUnit-Rules %}), I demonstrated how to 
use [JUnit's](https://github.com/junit-team/junit/wiki) [Rules feature](https://github.com/junit-team/junit/wiki/Rules)
to assert expected assertions in your unit tests. In this post, I'll show you how to write custom 
[Matchers](https://code.google.com/p/hamcrest/wiki/Tutorial) that will help give you more power when 
inspecting your exceptions.

## Maven Dependencies

This demo uses the following Maven dependencies:

{% highlight xml %}
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-library</artifactId>
    <version>1.3</version>
</dependency>
{% endhighlight %}

## Custom Exception

We'll start with a custom exception which does little more than remember the error code at the time the exception was thrown.

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

## Our Exception Matcher

When we pass our Matcher to JUnit's [ExpectedException](http://junit.org/javadoc/4.10/org/junit/rules/ExpectedException.html) instance, 
we're given a chance to match the exception itself, not the message. In this case, we're going to 
write a Matcher that makes sure that the exception's error code was as expected. We can only match 
on an instance of our  _ ErrorCodeException_, so we'll save some effort and extend 
[TypeSafeMatcher](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/TypeSafeMatcher.html).

[From the documentation](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/TypeSafeMatcher.html):

> TypeSafeMatcher : Convenient base class for Matchers that require a non-null value of a specific type. This simply implements the null check, checks the type and then casts.

{% highlight java %}
import org.hamcrest.Description;
import org.hamcrest.TypeSafeMatcher;

/**
 * ErrorCodeMatcher - matches when an ErrorCodeException has the same
 * error code as expected.
 */
public class ErrorCodeExceptionMatcher extends TypeSafeMatcher<ErrorCodeException>
{
    /**
     * The error code we're expecting.
     */
    private int expectedErrorCode;

    /**
     * Constructor.
     * 
     * @param expectedErrorCode
     *            the error code that we're expecting
     */
    public ErrorCodeExceptionMatcher(int expectedErrorCode)
    {
        this.expectedErrorCode = expectedErrorCode;
    }

    /**
     * Describe the error condition.
     */
    @Override
    public void describeTo(Description description)
    {
        description.appendText("Error code doesn't match");
    }

    /**
     * Test if the input exception matches the expected exception.
     */
    @Override
    protected boolean matchesSafely(ErrorCodeException exceptionToTest)
    {
        return exceptionToTest.getErrorCode() == this.expectedErrorCode;
    }

}
{% endhighlight %}

## Example Tests With ExpectedException and Our Custom Matcher

With the components in place, let's start testing. Of course, if you only have one or two error test 
cases, then a custom Matcher might take more work than it saves, but you end up with code that any 
developer should be able to read, which might reduce maintenance costs.

{% highlight java %}
package org.blakecaldwell.tests.junitrules;

import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ExpectedException;

/**
 * Test fixture to demonstrate JUnit's ExpectedException @Rule with a
 * custom matcher.
 */
public class ExpectedExceptionRuleCustomMatcherTest
{
    /**
     * ExpectedException must be public.
     */
    @Rule
    public ExpectedException exception = ExpectedException.none();

    /**
     * Test exception error code 500 using our custom Matcher.
     */
    @Test
    public void testErrorCode500UsingMatcher()
    {
        exception.expect(new ErrorCodeMatcher(500));

        // Run your code that you expect will throw an
        // ErrorCodeException with
        // error code 500
        throw new ErrorCodeException(500);
    }

    /**
     * Test exception error code 404 using our custom Matcher.
     */
    @Test
    public void testErrorCodeUsingMatcher()
    {
        exception.expect(new ErrorCodeMatcher(404));

        // Run your code that you expect will throw an
        // ErrorCodeException with
        // error code 404
        throw new ErrorCodeException(404);
    }
}
{% endhighlight %}

## Asserting Exceptions The Old Way

To demonstrate the benefits of using custom Matchers with JUnit's ExcpectedException, here are the 
alternatives that you're probably familiar with, with inline comments explaining why they're not ideal.

{% highlight java %}
package org.blakecaldwell.tests.junitrules;

import org.junit.Assert;
import org.junit.Test;

/**
 * Tests that demonstrate how to assert exceptions without
 * ExpectedException @Rule and custom Matchers.
 */
public class ExceptionAssertingOldWayTest
{
    /**
     * Test that an ErrorCodeException was thrown with the "expected"
     * parameter. The limitation here is that you can only test the
     * type of Exception, not its parameters or the message.
     */
    @Test(expected = ErrorCodeException.class)
    public void testErrorCodeExceptionThrownOldWay1()
    {
        // Run your code that you expect will throw an
        // ErrorCodeException with
        // error code 500
        throw new ErrorCodeException(500);
    }

    /**
     * Test exception error code 404 by capturing the Exception and
     * inspecting it inside the catch.
     * 
     * Remarks: We have to make sure to assert failure after our code
     * was supposed to throw the exception, or we won't know if it
     * never does. This nuance makes this method error prone for
     * future modifications.
     */
    @Test
    public void testErrorCodeExceptionThrownOldWay2()
    {
        try
        {
            // The "A".equals("A") is here just for this demo to
      // prevent an "Unreachable code" warning in Eclipse, since
      // we're throwing the exception directly instead of
      // calling code that throws it.
            if ("A".equals("A"))
            {
                // Run your code that you expect will throw an
                // ErrorCodeException with error code 404
                throw new ErrorCodeException(404);
            }

            // if we've gotten to this point, then no Exception was
            // thrown
            Assert.fail("Test case failed: no exception thrown");
        }
        catch (Exception ex)
        {
            Assert.assertEquals(
                    "Expected 404 error code exception",
                    404,
                    ((ErrorCodeException) ex).getErrorCode());
        }
    }

   /**
    * Test exception error code 404 by capturing the Exception and
    * inspecting it outside the try/catch.
    * 
    * Remarks: This is safer than asserting inside the catch (as
    * above), since if the exception is never thrown, our assertion
    * won't be valid, but still is not a very elegant solution.
    */
    @Test
    public void testErrorCodeExceptionThrownOldWay3()
    {
        Exception caughtException = null;
        try
        {
            // Run your code that you expect will throw an
            // ErrorCodeException
            // with error code 404
            throw new ErrorCodeException(404);
        }
        catch (Exception ex)
        {
            caughtException = ex;
        }

        // if the exception was never thrown, or was thrown and is an
        // incorrect
        // type or wrong code, this will fail
        Assert.assertEquals(
                "Expected 404 error code exception",
                404,
                ((ErrorCodeException) caughtException).getErrorCode());
    }
}
{% endhighlight %}
