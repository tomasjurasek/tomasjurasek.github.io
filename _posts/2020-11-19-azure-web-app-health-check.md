---
layout: post
title: Scalability of the Azure Web App - Health check
---
[Azure App Service](https://docs.microsoft.com/en-US/azure/app-service/overview) is the Azure Service, where you can host a web application. The Azure App Service provides you tons of features which can help you build a scalable cloud application.

The [health check](https://docs.microsoft.com/en-US/azure/azure-monitor/platform/autoscale-get-started#health-check-path) is a feature of Azure Web App, which constantly pings the custom endpoint and monitors the response status code of your application. 


## How it works

By default, the health check pings all instances of the App Service. When the health check gets a five times status code different than 200-299 then the concrete instance will be recycled and the traffic will be turned to a different *health* instance.

The rule of five failed health check you can change by app setting **WEBSITE_HEALTHCHECK_MAXPINGFAILURES**, where the number of the failing ping must be between 2 and 10. The default value is 5.

Also, you can change the maximum percent of how many instances can be recycled at the same time by app setting **WEBSITE_HEALTHCHECK_MAXUNHEALTYWORKERPERCENT**, where the number must be between 0 and 100. The default value is 50.


## How to set up and monitor
You need to set Enabled and Path to the custom endpoint of your application.

![HealthCheck](/images/posts/health-check-1.PNG)

Then you can monitor your health check on the Metrics. On this metric, you can create an alert.

![HealthCheckMetric](/images/posts/health-check-2.PNG)