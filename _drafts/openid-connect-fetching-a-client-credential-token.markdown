---
layout: post
title: "OpenID Connect: Fetching a Client Credential Token"
date: 2014-09-08 15:04:09 -04:00
comments: true
categories: sso
---

My work duties have recently grown to include the architecture, installation, configuration, and care and feeding or our brand new single sign on (SSO) server, Ping Identity's [Ping Federate](https://www.pingidentity.com/en/products/pingfederate.html).

I've pored over documentation and with the benefit of a Ping Identity architect have come up with a bunch of samples to be distributed within the company. Just because I can't distribute those samples doesn't mean that everyone can't benefit from my hard fought knowledge.

The first snippet that I have is the most simple: using the HttpClient to fetch a token through the [OAuth 2 Client Credential Flow](http://documentation.pingidentity.com/display/PF72/OAuth+2.0).

{% highlight C# linenos %}
using(HttpClient client = new HttpClient())
{
    var bytes = System.Text.Encoding.UTF8.GetBytes("your client ID" + ":" + "your client secret");
    var basicAuthenticationHeader = System.Convert.ToBase64String(bytes);

    var content = new StringContent("grant_type=client_credentials&scope=edit");
    content.Headers.ContentType = new MediaTypeHeaderValue("application/x-www-form-urlencoded");

    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:9031/as/token.oauth2"),
        Method = HttpMethod.Post,
        Content = content,
    };

    request.Headers.Authorization = new AuthenticationHeaderValue("Basic", basicAuthenticationHeader);

    var results = client.SendAsync(request).Result.Content.ReadAsStringAsync().Result;

    dynamic token = JsonConvert.DeserializeObject<dynamic>(results);
}
{% endhighlight %}

Once you have the token, you can use the `token.access_token` as a bearer token in an Authorization header to call other OAuth2 secured web services.
