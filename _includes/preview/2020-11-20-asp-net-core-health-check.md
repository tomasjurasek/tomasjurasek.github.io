---
layout: post
title: Implementation of the Health check in the ASP.NET Core
---
The healt check is a custom endpoint in your application which provides a status of the application.

You can check
* The application status
* The application dependencies - SQL, CosmosDB, Storage Account, Service Bus, ...


There is exists the project - [AspNetCore.Diagnostics.HealthChecks](https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks) which provides you a collection of ASP.NET Core Health check packages.

## How to create a health check 

Register health check services in `Startup.ConfigureService` by
```cs
 services.AddHealthChecks();
 ```

Then create a health check endpoint in `app.UseEndpoint`.
 ```cs
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
});
```

If you need to check your dependencies for example the SQL Server then install the package `AspNetCore.HealthChecks.SqlServer` from the  [AspNetCore.Diagnostics.HealthChecks](https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks) and register the health check.
 ```cs
services.AddHealthChecks()
         .AddSqlServer("connectionString");
 ```

Now, when you call your health endpoint - /health and the SQL Server will not be responding the endpoint returns - 503 status code.

**Links**

[https://docs.microsoft.com/en-US/aspnet/core/host-and-deploy/health-checks](https://docs.microsoft.com/en-US/aspnet/core/host-and-deploy/health-checks)

[https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks](https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks)