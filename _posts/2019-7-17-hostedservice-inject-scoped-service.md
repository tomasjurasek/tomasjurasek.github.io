---
layout: post
title: IHostedService - závislosti služeb zaregistrovaných jako Scoped
---

[Hosted služby](https://docs.microsoft.com/cs-cz/dotnet/standard/microservices-architecture/multi-container-microservice-net-applications/background-tasks-with-ihostedservice) slouží pro zpracovávání dlouhotrvajících úloh na pozadí kontextu aplikace. 

Pro vytvoření hosted service je potřeba aby třída dědila z [IHostedService]((https://docs.microsoft.com/cs-cz/dotnet/api/microsoft.extensions.hosting.ihostedservice?view=aspnetcore-2.2)), nebo přímo z [BackgroundService](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.backgroundservice?view=aspnetcore-2.2).
Poté třídu zaregistrujeme jako hosted service:
```cs
services.AddHostedService<T>();
```

## Problém

Do hosted služby občas potřebujeme závislosti služeb, které mají registrovaný životní cyklus jako *'Scoped'*. Hosted služba nevytváří vlastní scope, a tím pádem služby zaregistrované jako *'Scoped'* budou vyhazovat exception typu:

>***System.InvalidOperationException: ‘Cannot consume scoped service ‘Service’ from singleton ‘Microsoft.Extensions.Hosting.IHostedService’.***

## Řešení

Pro použítí závislostí služeb typu 'Scoped' si musíme vytvořit vlastní scope pomocí ***IServiceScopeFactory***, kde si službu vytáhneme přes - ***GetRequiredService< T>()***

**Ukázka**
```cs
public class DefaultHostedService : IHostedService
{
    private readonly IServiceScopeFactory _serviceScopeFactory;

    public DefaultHostedService(IServiceScopeFactory serviceScopeFactory)
    {
        this._serviceScopeFactory = serviceScopeFactory;
    }

    public Task StartAsync(CancellationToken cancellationToken)
    {
        using (var scope = _serviceScopeFactory.CreateScope())
        {
            var service = scope.ServiceProvider.GetRequiredService<IService>();
            //logic
        }
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }
}
```