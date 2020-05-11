---
layout: post
post_number: 3
title: "Java: Embedding Spring Batch Admin Into An Existing Application"
categories: Java
---

For my current project, I need to share [Spring](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/) beans between my front-end web servlets and the [Spring Batch](http://static.springsource.org/spring-batch/) jobs that I'll launch from [Spring Batch Admin](http://static.springsource.org/spring-batch-admin/reference/screenshots.html) (SBA).  This sounds straightforward - configure the SBA servlet to listen on /batch/*, and you should be good to go.  However, of course it's not that easy.

### URL Path Mismatch

The first problem you run into is bad links in the menus, form posts that go nowhere, and missing CSS files. This is because SBA expects to be deployed on /, not /batch, or /myapp/batch which is where it will go if you deploy it inside another app. You can survive in this mode for the most part by correcting menu URLs after you click them, or use [FireBug](http://getfirebug.com) to change form actions before you submit them - for example,  from /batch/files to /myapp/batch/files, but who wants to live [like an animal](http://www.kungfugrippe.com/post/20021002957/like-an-animal)?

### Spring Batch Admin 1.2.1 to the Rescue

Spring Batch Admin version 1.2.1 [adds the ability](https://jira.springsource.org/browse/BATCHADM-108) to set the base servlet path for all links and forms by overriding the _"resourceService"_ bean. I'm sure there are several ways to successfully accomplish this, but here's what worked for me.


#### pom.xml

Add the following repository:

{% highlight xml %}
<repository>
  <id>repository.springframework.maven.release</id>
  <name>Spring Framework Maven Release Repository</name>
  <url>http://maven.springframework.org/release</url>
</repository>
{% endhighlight %}

and the dependencies:

{% highlight xml %}
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-admin-resources</artifactId>
    <version>1.2.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-admin-manager</artifactId>
    <version>1.2.1.RELEASE</version>
</dependency>
{% endhighlight %}

#### web.xml

Configure the Batch Admin Servlet - notice _contextConfigLocation_:
    
{% highlight xml %}
<servlet>
  <servlet-name>Batch Admin Servlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
      <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/batch-admin/batch-admin-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>Batch Admin Servlet</servlet-name>
    <url-pattern>/batch/*</url-pattern>
</servlet-mapping>
{% endhighlight %}

#### /WEB-INF/spring/batch-admin/batch-admin-context.xml

This is the context file that's only used by SBA. Keep in mind that any beans defined in your root Spring context will also be available to the jobs that are launched by SBA.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <!-- Spring Batch Admin Context: Additional context for Spring Batch Admin -->
  <import resource="classpath*:/META-INF/spring/batch/servlet/resources/*.xml" />
  <import resource="classpath*:/META-INF/spring/batch/servlet/manager/*.xml" />
  <import resource="classpath*:/META-INF/spring/batch/servlet/override/*.xml" />
  <import resource="classpath*:/META-INF/spring/batch/bootstrap/**/*.xml" />
  <import resource="classpath*:/META-INF/spring/batch/override/**/*.xml" />

  <!-- For Spring Batch -->
  <bean id="resourceService"
    class="org.springframework.batch.admin.web.resources.DefaultResourceService">
    <property name="servletPath" value="/batch" />
  </bean>
</beans>
{% endhighlight %}
		
### One Last Gotcha - /myapp/batch
		
The main "Home" link in the SBA action bar navigates to /myapp/batch.  This doesn't work the way I've wired it up, because the SBA servlet is configured to listen to /myapp/batch/, but not /myapp/batch. I tried adding another servlet-mapping to /batch, but the servlet won't answer requests to it.

I'm sure there's a better way to do this via config only - please tell me if you wouldn't mind, but I opted for the more manual way, just so I could move on.

I wired up a servlet in my main web app to listen to /batch, and then have it redirect to /batch/.

#### web.xml - configuration for a servlet in my main app

Pay attention to the last servlet-mapping. This gives /myapp/batch over to Spring MVC for URL mapping.

{% highlight xml %}
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
  </init-param>
  <async-supported>true</async-supported>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>appServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>

<!-- LOOK HERE -->
<servlet-mapping>
  <servlet-name>appServlet</servlet-name>
  <url-pattern>/batch</url-pattern>
</servlet-mapping>
{% endhighlight %}
	

#### HomeController.java

Here's where I redirect /batch to /batch/. Again, I'm sure there's a better way to do this (let me know!). Of course, for this specific code to work, you'll need some dependencies such as spring-webmvc. The bottom line is that I'm just manually listening on /batch and redirecting to /batch/.

{% highlight java %}
@Controller
public class HomeController
{
    /**
     * Redirect the url /batch to /batch/ for the Spring Batch Admin to pick it up.
     * 
     * @return redirect to /batch/
     */
    @RequestMapping(value = "/batch", method = RequestMethod.GET)
    public String redirectBatchToBatchSlash()
    {
        return "redirect:/batch/";
    }
}
{% endhighlight %}
	
### Know a Better Way?

Ideally, I'd like to either get Spring Batch Admin to listen to /batch or /myapp/batch, but I'd settle on having a redirect configured in web.xml. Please tell me if you know of a good approach. I moved on with this good-enough solution.

