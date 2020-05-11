---
layout: post
post_number: 10
title: "Conditionally Run JUnit Integration Tests with Spring"
categories: Java
---

Ideally, your unit test suites require no external dependencies - those should be mocked out. However, sometimes you want extra assurance that your code works with live endpoints via integration tests.

For example, you might want to make sure that your code can successfully navigate your corporate proxy and firewall to make external web requests. For this, we'll write tests that only run when a command line parameter is defined.

This demonstration assumes that you're using [Spring's JUnit Runner](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#testcontext-junit4-runner) and [Maven](http://maven.apache.org) for running your tests. The [@IfProfileValue](http://docs.spring.io/spring/docs/3.2.4.RELEASE/javadoc-api/org/springframework/test/annotation/IfProfileValue.html) annotation will tell the test runner to only include this test suite if our configured [ProfileValueSource](http://docs.spring.io/spring/docs/3.2.4.RELEASE/javadoc-api/org/springframework/test/annotation/ProfileValueSource.html) returns "true" for "live-external-tests-enabled".

{% highlight java %}
package org.blakecaldwell;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.annotation.IfProfileValue;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@IfProfileValue(name="live-external-tests-enabled", value="true")
public class IntegrationTest
{
    @Test
    public void testExternalConnection()
    {
        Assert.fail("testExternalConnection failed!");
    }
}
{% endhighlight %}

Let's verify that this test suite is ignored by default with the maven command:

{% highlight console %}
mvn test
{% endhighlight %}

You'll see that you now have a skipped test:

{% highlight console %}
Results:

Tests run: 1337, Failures: 0, Errors: 0, Skipped: 1
{% endhighlight %}

Now, try it again, but this time with our test included:


{% highlight console %}
mvn test -Dlive-external-tests-enabled=true
{% endhighlight %}

You'll now see that the test was run:

{% highlight console %}
Results :

Tests in error: 
  testExternalConnection(org.blakecaldwell.IntegrationTest): Failed to load ApplicationContext

Tests run: 1337, Failures: 0, Errors: 1, Skipped: 0
{% endhighlight %}

The `@IfProfileValue` annotation can also be used above an individual test.
