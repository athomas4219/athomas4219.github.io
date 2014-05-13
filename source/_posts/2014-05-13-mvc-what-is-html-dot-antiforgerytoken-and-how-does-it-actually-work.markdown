---
layout: post
title: "MVC: What is Html.AntiForgeryToken and how does it actually work?"
date: 2014-05-13 10:28:34 -0400
comments: true
categories:
---

The anti-forgery token found in MVC is a way to prevent [cross site request forgery](http://en.wikipedia.org/wiki/Csrf) (CSRF) attacks. Without going into too much detail, a CSRF attack occurs when a user visits an untrusted site and enters some information that is then posted back to a site to which the user has already authenticated. Protection against this exploit has shipped with MVC since at least version 4 and is very easy to implement.

## How to Implement the Anti-forgery Token

Protecting your web site is as easy as adding some code to your view and decorating the view's controller action with an attribute. **Note:** if either one of these steps is not done, your web site won't be protected and may or may not cause an exception at run time.

First, you add a call to `Html.AntiForgeryToken()` inside any `form` tag that you want to protect.

**Note:** if you, like me, sometimes get so focused on not forgetting to add the anti-forgery token that it is the first code that you add to the page, you may find that you add it outside of the `using` statement, causing an exception at run time:

{% codeblock %}

The required anti-forgery form field "__RequestVerificationToken" is not present.

{% endcodeblock %}

If this occurs, simply move your anti-forgery token inside the `using` statement, as seen in the sample below. It belongs inside the `form` tag on the page.

The following is an example of a Razor view for a form that contains a name and email field. The only additional magic here is to add the line containing `@Html.AntiForgeryToken()`.

{% codeblock lang:c# Views\Home\Sample.cshtml %}

@model AntiForgerySample.Models.SampleViewModel

@{
    ViewBag.Title = "Sample";
}

<h2>AntiForgeryToken Sample</h2>

@using (Html.BeginForm("Sample", "Home", FormMethod.Post, new { role = "form" }))
{
    @Html.AntiForgeryToken()

    @Html.ValidationSummary()

    <div class="form-group">
        @Html.LabelFor(m => m.Name)
        @Html.EditorFor(m => m.Name)
    </div>
    <div class="form-group">
        @Html.LabelFor(m => m.Email)
        @Html.EditorFor(m => m.Email)
    </div>
    <button type="submit" class="btn btn-default">Save</button>
}

{% endcodeblock %}

Next, you decorate your controller action with the `ValidateAntiForgeryToken` attribute. The following is the controller action to support the above view.

{% codeblock lang:c# Controllers\HomeController.cs %}

[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Sample(FormCollection formCollection)
{
    var model = new Models.SampleViewModel();

    if (TryUpdateModel(model, formCollection))
    {
        // Do some things with the model, then redirect to the home page
        return RedirectToAction("Index", "Home");
    }

    return View();
}

{% endcodeblock %}

## So how does it actually work?

Under the covers, the call to `Html.AntiForgeryToken()` does two things. First, it causes a hidden input to be created in the form, like this:

{% codeblock lang:html %}

<input name="__RequestVerificationToken" type="hidden" value="MRRR4ga6eu8EiCeHKdJAiZgF1EO4OC52U3L6UUe_nqIfe7I7jONPmsJ2Vc_chDi7gA7drUYy4hyehi_UDkso2wrbmLgaKXGypM0rAfHec7g1" />

{% endcodeblock %}

Second, it causes a cookie also called `__RequestVerificationToken` to be created with the same value.

When the page gets posted to the server, the `ValidateAntiForgeryToken` attribute causes the MVC framework to marry the hidden form parameter with the cookie value. If they are not the same, or either one is missing, the server knows that the post did not come from our own server and that we shouldn't trust the input, causing an exception to be thrown.

The magic that occurs here is that the browser will only send the cookie if it's posting to the domain that created the cookie. Thus, if a user inputs data into an untrusted site, which then posts back to our site, the anti-forgery token on the page will not match the one previously created and stored in the cookie. When MVC smells something fishy, it throws an exception, protecting our user from an attack.
