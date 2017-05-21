---
layout: post
title: "Managing ASP.NET MVC Routes"
date: 2017-05-21
disqus_identifier: 5
---
tl;dr: Use UrlHelper or HtmlHelper extensions to avoid magic strings and to provide intellisense for route names and parameters.

[Source code](https://github.com/TheOtherTimDuncan/RoutingDemo)

Peter Vogel had an interesting article a while back titled [The Best Advice You'll Get on ASP.NET MVC Routing](https://visualstudiomagazine.com/articles/2016/07/15/best-advice-aspnet-mvc-routing.aspx). While I agree on the concept, I disagree on the approach. What would my advice be then?

First, I no longer care to use the included HtmlHelper.ActionLink extensions or many of the other html helpers and prefer to use the actual html instead. That is why I use UrlHelper extensions. My approach could easily be adapted as a wrapper around HtmlHelper.ActionLink if you prefer.

Second, I prefer to use attribute routing instead of the default convention-based routing. This approach works with either routing setup, and if anything is of more value with convention-based routing since you are even more reliant on magic strings with that approach.

Finally, for the source code, I just started with the default template for an MVC application and then added the bare minimum to serve as a proof of concept. In a future post, I'll explain how I set up an MVC application in a more real-world scenario.

The first step in my approach - after getting attribute routing working - is to add a public const to each controller with the route name. I then change the [Route] attribute to use that const.

```csharp
[RoutePrefix(HomeController.Route)]
public class HomeController : Controller
{
    public const string Route = "Home";
}
```

Next, I add the UrlHelper extension.

```csharp
public static class UrlHelperExtensions
{
    public static string HomeAction(this UrlHelper urlHelper)
    {
        return urlHelper.Action(nameof(HomeController.Index), HomeController.Route);
    }
}
```

If you're using C# 6.0, you can take advantage of [nameof](https://docs.microsoft.com/en-us/dotnet/articles/csharp/language-reference/keywords/nameof) to get the action name. If you're not, this approach becomes even more valuable.

Other than getting rid of or centralizing magic strings, the other value of this approach is parameters. If any of your controller actions take a parameter, include it as part of the UrlHelper extension and gain the advantage of intellisense as well. I changed the Contact action in the example project to look like this:

```csharp
[RoutePrefix(HomeController.Route)]
public class HomeController : Controller
{
    public const string Route = "Home";

    /* Snipped for brevity */

    [Route("~/Contact")]
    public ActionResult Contact(ContactQuery query)
    {
        ViewBag.Message = $"Your contact page for {query.ContactName}.";

        return View();
    }
}

public class ContactQuery
{
    public string ContactName
    {
        get;
        set;
    }
}
```

and the corresponding UrlHelper action extension:

```csharp
 public static class UrlHelperExtensions
{
    public static string HomeContactAction(this UrlHelper urlHelper, string contactName)
    {
            return urlHelper.Action(nameof(HomeController.Contact), HomeController.Route, new ContactQuery()
            {
                ContactName = contactName
            });
    }
}
```

So in summary, this approach will either get rid of magic strings for your urls or at least centralize them. If you have [MvcBuildViews](https://blogs.msdn.microsoft.com/jimlamb/2010/04/20/turn-on-compile-time-view-checking-for-asp-net-mvc-projects-in-tfs-build-2010/) turned on, you can also catch broken urls on build. The disadvantage of this approach is creating a UrlHelper extension for urls that are only used in one place can seem like overkill, but I still consider the extra effort worth it.
