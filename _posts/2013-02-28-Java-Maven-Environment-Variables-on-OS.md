---
layout: post
post_number: 2
title: "Java: Maven Environment Variables on OS X"
categories: Java
---

Mac OS X comes with [Apache Maven](http://maven.apache.org "Apache Maven") preinstalled. However, you're still going to need 
to set some environment variables to get up and running.

Verify that maven installed in /usr/share/maven:

{% highlight bash %}
ls /usr/share/maven
{% endhighlight %}

Add the following line to your ~/.profile

{% highlight bash %}
# Set the M2_HOME environment variable
export M2_HOME=/usr/share/maven

# Put maven on your path
export PATH=$M2_HOME/bin:$PATH
{% endhighlight %}

This won't take effect until you either reboot or open a new terminal window. You can reload the file in the current terminal 
window with the following command:

{% highlight bash %}
source ~/.profile
{% endhighlight %}
