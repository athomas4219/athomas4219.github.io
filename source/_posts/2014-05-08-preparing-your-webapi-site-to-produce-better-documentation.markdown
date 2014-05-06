---
layout: post
title: "Preparing Your WebAPI Site To Produce Better Documentation"
date: 2014-05-08 13:59:43 -0400
comments: true
categories: 
---

### Introduction

A WebAPI site produced from the Visual Studio 2013 template comes with an MVC area designed to provide rich help pages. For the most part, this functionality comes for free with the template. However, using the following instructions can help produce a richer help experience for the user by allowing you to use multiple lines in your method descriptions. You are then free to include more information about the method, including expected return codes or additional descriptive text.

### Walkthrough

**Note:** if you are creating a brand new site, the help area will only be created if you select the Web API project template when creating a new project.

First, we need to configure the project to produce the XML file used by the help system.

* Right-click on your WebAPI project and select `Properties`
* Click on the `Build` tab
* In the `Output` section, click on the checkbox entitled `XML documentation file`
* In the text box, enter: `App_Data\XmlDocument.xml`
* Save the project and close the project properties page.

Next, we need to configure the help system to actually consume the XML file created from the step above and produce the output web pages.

* Find the file `Areas->HelpPage->App_Start->HelpPageConfig.cs`
* Uncomment the line reading:

{% codeblock lang:c# Areas\HelpPage\App_Start\HelpPageConfig.cs %}

config.SetDocumentationProvider(new XmlDocumentationProvider(HttpContext.Current.Server.MapPath("~/App_Data/XmlDocument.xml")));

{% endcodeblock %}

* Add configuration to set the response types like the following:

{% codeblock lang:c# Areas\HelpPage\App_Start\HelpPageConfig.cs %}

config.SetActualResponseType(typeof(MyReturnObject), "MyControllerName", "Get");
config.SetActualResponseType(typeof(MyReturnObject), "MyControllerName", "Post");

{% endcodeblock %}

Next, we'll do a little cosmetic updating of some HTML files.

* Find the file `Areas->HelpPage->Views->Help->Index.cshtml`
* Change the `<title>` of the page to match your application
* Remove the `<header>` section
* Provide a relevant introduction

In later steps, we're going to update our help controller to allow us to enter comments that contain many lines. In order to allow that in the HTML, we need to update a few markup files.

* Find the file `Areas->HelpPage->Views->Help->DisplayTemplates->ApiGroup.cshtml`
* Update the line reading:

{% codeblock lang:c# Areas\HelpPage\Views\Help\DisplayTemplates\ApiGroup.cshtml %}

<p>@api.Documentation</p>

{% endcodeblock %}

to:
	
{% codeblock lang:c# Areas\HelpPage\Views\Help\DisplayTemplates\ApiGroup.cshtml %}

<p><pre>@api.Documentation</pre></p>

{% endcodeblock %}

Next, we're going to add a couple CSS styles that will help the page look a little cleaner.

* Find the file `Areas->HelpPage->HelpPage.css`
* Add the following to the bottom of the file:

{% codeblock lang:css Areas\HelpPage\HelpPage.css %}

.api-documentation pre {
    background-color: transparent;
    border: transparent;
}

.parameter-documentation pre {
    background-color: transparent;
    border: transparent;
}

{% endcodeblock %}

Next, it's time to update the controller to allow us to use multiple lines in our method descriptions.

* Find the file `Areas->HelpPage->Controllers->HelpController.cs`
* Replace the `Index` method with the following:

{% codeblock lang:c# Areas\HelpPage\Controllers\HelpController.cs %}

public ActionResult Index()
{
    var descriptions = Configuration.Services.GetApiExplorer().ApiDescriptions;

    var rex = new System.Text.RegularExpressions.Regex(@"\n\s+");

    foreach (var description in descriptions)
    {
        if (!String.IsNullOrEmpty(description.Documentation))
        {
            description.Documentation = rex.Replace(description.Documentation, "\n");
        }
    }

    return View(descriptions);
}

{% endcodeblock %}

* Replace the `Api` method with the following:

{% codeblock lang:c# Areas\HelpPage\Controllers\HelpController.cs %}

public ActionResult Api(string apiId)
{
    if (!String.IsNullOrEmpty(apiId))
    {
        HelpPageApiModel apiModel = Configuration.GetHelpPageApiModel(apiId);
        if (apiModel != null)
        {
            var rex = new System.Text.RegularExpressions.Regex(@"\n\s+");
            apiModel.ApiDescription.Documentation = rex.Replace(apiModel.ApiDescription.Documentation, "\n");

            foreach (var param in apiModel.ApiDescription.ParameterDescriptions)
            {
                param.Documentation = rex.Replace(param.Documentation, "\n");
            }

            return View(apiModel);
        }
    }

    return View("Error");
}

{% endcodeblock %}

Now your help pages should allow you to use multiple lines in your method descriptions, as well as looking a little cleaner when viewed in a browser. You can test your new help pages by navigating to `http://localhost/MyApplication/help/`.
