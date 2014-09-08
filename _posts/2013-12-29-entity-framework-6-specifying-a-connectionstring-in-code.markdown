---
layout: post
title: "Entity Framework 6: Specifying a ConnectionString in Code"
date: 2013-12-29 08:37:31 -0400
comments: true
categories: 
---

I've been working recently with Entity Framework 6. So far I find it pleasant enough, although not as intuitive (at least for me) as the likes of NHibernate.

One of the things I wanted to do was to determine the connection string in code, rather than specifying it in my web.config file. You can use the following code snippet to set your connection string.

{% highlight C# linenos %}
public class MyContext : DbContext
{
	public MyContext() : base()
	{
		Database.Connection.ConnectionString = myAlternateConnectionString;
	}
}
{% endhighlight %}
