---
layout: post
title: ASP.NET Core 5 MVC - Localized routing & links
---

When you need to localize your application you need to set up some resource files and configure the [UseRequestLocalization](https://docs.microsoft.com/en-US/aspnet/core/fundamentals/localization?view=aspnetcore-5.0) with supported cultures.

Then it can works like this:  
`/en-US/Home/Index`  
`/cs-CZ/Home/Index`

What about when you need to localize your routes and links too? 
Something like this:  
`/en-US/Home/Index`  
`/cs-CZ/Domu/Uvod`

I've created the nuget package [AspNetCore.Mvc.Routing.Localization](https://www.nuget.org/packages/AspNetCore.Mvc.Routing.Localization) which offers to you localize your routes by the `LocalizedRouteAttribute` and links by the tagHelper `<localized-route ...></localized-route>`.

GitHub - [https://github.com/tomasjurasek/AspNetCore.Mvc.Routing.Localization](https://github.com/tomasjurasek/AspNetCore.Mvc.Routing.Localization)