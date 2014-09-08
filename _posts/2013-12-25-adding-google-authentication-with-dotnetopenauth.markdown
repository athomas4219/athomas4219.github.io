---
layout: post
title: "Adding Google Authentication with DotNetOpenAuth"
date: 2013-12-25 10:50:49 -0400
comments: true
categories: 
---

Visual Studio 2013 ships with a very nice template to use [DotNetOpenAuth](http://dotnetopenauth.net/) for authentication with Google, Twitter, Facebook, and Microsoft. However, it's a little complicated and comes with a lot of cruft. Rather than having too much in my ASP.NET MVC project, I prefer to start with an empty project and add features as I need them. If you are like me, you can use the following instructions to add Google authentication using DotNetOpenAuth.

#### Creating the Project

First, I created an empty MVC project. Once done, I used NuGet to add the `DotNetOpenAuth` package, which added quite a few other package dependencies. I also added the `Microsoft ASP.NET Web Optimization Framework` package.

Next, open your `web.config` file and add a section for forms authentication:

{% highlight xml linenos %}
<system.web>
	<authentication mode="Forms">
		<forms loginUrl="~/Account/Logon" timeout="15" slidingExpiration="true" />
	</authentication>
</system.web>
{% endhighlight %}

Next, we're going to add some controllers and views to our project. These controllers will display a welcome page, which will be unsecured, some methods used for logon, and a home page which will be secured.

#### Creating the Welcome Controller

Right click on your `Controllers` folder and add a new controller called `WelcomeController`. This controller will only have an `Index` method because it will only have a single page with the link to our logon method.

Under the `Views` folder, right click on the `Welcome` folder and add a new view called `Index`. You can add whatever markup you want, but be sure to add a link to our logon controller action somewhere on the page:

{% highlight C# linenos %}
@Html.ActionLink("Sign In", "Logon", "Account")
{% endhighlight %}

#### Updating the Default Route

Open the `RouteConfig.cs` file under the `App_Start` folder. Change the default controller from `Home` to `Welcome`. When you're done, your route should look like:

{% highlight C# linenos %}
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Welcome", action = "Index", id = UrlParameter.Optional }
);
{% endhighlight %}

#### Creating the Account Controller

Right click on your `Controllers` folder and add a new controller called `AccountController`. This controller will have three methods, one to perform a redirect to the Google authentication server, a method to process the return call, and a `LogOff` method to clear the authentication cookie.

Add the following methods to your Account controller:

{% highlight C# linenos %}
public ActionResult Logon(string returnUrl)
{
    var rp = new OpenIdRelyingParty();
    var request = rp.CreateRequest("https://www.google.com/accounts/o8/id", Realm.AutoDetect, new Uri(Request.Url, Url.Action("Authenticate")));

    if (request != null)
    {
        if (!String.IsNullOrEmpty(returnUrl))
        {
            request.AddCallbackArguments("returnUrl", returnUrl);
        }

        var fetch = new FetchRequest();

        fetch.Attributes.AddRequired("http://axschema.org/contact/email");
        fetch.Attributes.AddRequired("http://axschema.org/namePerson/first");
        fetch.Attributes.AddRequired("http://axschema.org/namePerson/last");
        fetch.Attributes.AddRequired("http://schemas.openid.net/ax/api/user_id");

        request.AddExtension(fetch);

        return request.RedirectingResponse.AsActionResultMvc5();
    }

    return View();
}
{% endhighlight %}

You can see that we are requesting additional information from the Google authentication server: email address, first name, last name, and the user's unique ID.

Also add the `Authenticate` method:

{% highlight C# linenos %}
public ActionResult Authenticate(string returnUrl)
{
    var rp = new OpenIdRelyingParty();
    var response = rp.GetResponse();

    if (response != null)
    {
        switch (response.Status)
        {
            case AuthenticationStatus.Authenticated:
                string identifier = response.ClaimedIdentifier;

                var fetch = response.GetExtension<FetchResponse>();

                string email = null;
                string first = null;
                string last = null;
                string userId = null;

                if (fetch != null)
                {
                    email = fetch.GetAttributeValue("http://axschema.org/contact/email");
                    first = fetch.GetAttributeValue("http://axschema.org/namePerson/first");
                    last = fetch.GetAttributeValue("http://axschema.org/namePerson/last");
                    userId = fetch.GetAttributeValue("http://schemas.openid.net/ax/api/user_id");
                }

                // Add your own code to perform any checks in your system to find or create the user returned from Google

                FormsAuthenticationTicket authTicket = new FormsAuthenticationTicket(1,
                    FormsAuthentication.FormsCookieName, 
                    DateTime.Now,
                    DateTime.Now.AddDays(90),
                    true, 
                    "User data. Perhaps your user's unique ID?");
        
                string encryptedTicket = FormsAuthentication.Encrypt(authTicket);    

                HttpCookie authCookie = new HttpCookie(FormsAuthentication.FormsCookieName, encryptedTicket);

                if (authTicket.IsPersistent) 
                {     
                      authCookie.Expires = authTicket.Expiration; 
                }

                System.Web.HttpContext.Current.Response.Cookies.Add(authCookie);  
                
                if (Url.IsLocalUrl(returnUrl))
                {
                    return Redirect(returnUrl);
                }
                else
                {
                    return RedirectToAction("Index", "Home");
                }

                break;

            default:
                this.ModelState.AddModelError(String.Empty, "An error occurred during login.");

                break;
        }
    }

    return View("Logon");
}
{% endhighlight %}

Take note of a couple of things in the `Authenticate` method:
* You need to add your own code in this method to check your user store for an existing user, or create it if it doesn't already exist.
* We are creating our authentication ticket manually to give us more control over the cookie. You can store a little bit of information in the user data of the ticket. I store the ID of the user that's logged in, then use that information in a controller base class to lookup the user on each page.

Finally, add our brief `LogOff` method:

{% highlight C# linenos %}
public ActionResult LogOff()
{
    FormsAuthentication.SignOut();
    return RedirectToAction("Index", "Welcome");
}
{% endhighlight %}

#### Creating the Home Controller

Right click on your `Controllers` folder and add a new controller called `HomeController`. This controller will have at least one method to display the index page, but may have more, depending on your application.

Under the `Views` folder, right click on the `Home` folder and add a new view called `Index`. This page cannot be viewed without logging in, so is the first page you'll create in your application that will have secured content.

Open your new `HomeController.cs` file and add the `[Authorize]` attribute, so that your `HomeController` looks like the following. The `[Authorize]` attribute is what will ensure that this page cannot be viewed by unauthenticated users.

{% highlight C# linenos %}
[Authorize]
public class HomeController : BaseController
{
    //
    // GET: /Home/
    public ActionResult Index()
    {
        return View();
    }
}
{% endhighlight %}

If you've done everything correctly, once you run the project you'll be presented with your welcome screen. When you click on the `Sign In` link, you be redirected to the Google logon page, then back to your application's home page.
