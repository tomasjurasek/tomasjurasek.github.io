---
layout: post
title: Scalability of the Azure App Service - Health check
---
The [Azure App Service](https://docs.microsoft.com/en-US/azure/app-service/overview) is the Azure Service, where you can host a web application. The Azure App Service provides you tons of features which can help you build a scalable cloud application.

The [health check](https://docs.microsoft.com/en-US/azure/azure-monitor/platform/autoscale-get-started#health-check-path) is a feature of Azure App Service, which constantly pings the custom endpoint and monitors the response status code of your application. 


## How it works

By default, the health check pings your custom endpoint at all instances of the App Service.

 When the health check gets an error five times - status code different than 200-299 then the concrete instance will set as an '*unhealthy*' and the traffic will be turned to a different '*healthy*' instance. 
 
 If the '*unhealthy*' instance becomes '*healthy*' is returned otherwise, after 1 hour the app is re-deployed on a new instance.

> For the correct recycling, your App Service plan must be scaled out to 2 or more instances.

The rule of five failed health check you can change by the app setting **WEBSITE_HEALTHCHECK_MAXPINGFAILURES**, where the number of the failing ping must be between 2 and 10. The default value is 5.

Also, you can change the maximum percent of how many instances can be recycled at the same time by the app setting **WEBSITE_HEALTHCHECK_MAXUNHEALTYWORKERPERCENT**, where the number must be between 0 and 100. The default value is 50.


## How to set up and monitor
You need to set *Enable* and *Path* to the custom endpoint of your application.

![HealthCheck](/images/posts/health-check-1.PNG)

Then you can monitor your health check on the Metrics. On this metric, you can create an alert.

![HealthCheckMetric](/images/posts/health-check-2.PNG)

**Links**

[https://azure.github.io/AppService/2020/08/24/healthcheck-on-app-service.html](https://azure.github.io/AppService/2020/08/24/healthcheck-on-app-service.html)

[https://docs.microsoft.com/en-US/azure/azure-monitor/platform/autoscale-get-started](https://docs.microsoft.com/en-US/azure/azure-monitor/platform/autoscale-get-started)