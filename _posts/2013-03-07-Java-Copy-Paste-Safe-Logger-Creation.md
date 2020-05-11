---
layout: post
post_number: 4
title: "Java: Copy/Paste-Safe Logger Creation"
excerpt: "We've all copied and pasted a Logger from one class to the other, forgetting to change the class name. Here's one fix."
categories: Java
---

We've all copied and pasted a Logger from one class to the other, forgetting to change the class name.

{% highlight java %}
// WRONG
public class SomeNewClass
{
  // OOPS! I forgot to change the class name!
  private final Logger _logger = LoggerFactory.getLogger(SomeOtherClass.class);
  
  // …	
}
{% endhighlight %}
	

Here, use this smarter LoggerFactory which uses the stack trace to figure out what class you really mean. This is safe from copypasta laziness. 

{% highlight java %}
package org.blakecaldwell.logging;

import org.slf4j.Logger;

/**
 * Smart logger that uses reflection to figure out the logged class
 */
public final class ClassLoggerFactory
{
    /**
     * Disallow factory instances.
     */
    private ClassLoggerFactory()
    {
    }

    /**
     * Use the stack trace to determine the appropriate logger.
     * 
     * @return a logger for the direct caller's class.
     */
    public static Logger make()
    {
        Throwable t = new Throwable();
        StackTraceElement directCaller = t.getStackTrace()[1];
        return org.slf4j.LoggerFactory.getLogger(directCaller.getClassName());
    }
}
{% endhighlight %}

Then, to use this logger:

{% highlight java %}
// CORRECT
public class SomeNewClass
{
    // No chance for screwing up anymore!
    private final Logger _logger = ClassLoggerFactory.make();

    // …	
}
{% endhighlight %}
