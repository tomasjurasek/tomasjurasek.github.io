---
layout: post
title: ASP.NET Core - mockování HTTP závislostí v unit testech
---

Určitě jste se s podobnou situací setkali už nesšetněkrát - potřebujete napsat unit testy pro nějakou servisu/službu ve vaší aplikaci, ale služba obsahuje další závislosti. Například volá databázi, HTTP požadavky, pracuje se souborovým systémem a jiné.

V tomhle článku si ukážeme, jak se namockovat HTTP závislosti pro lepší izolaci testované služby.


ASP.NET Core nabízí [mechanismus](https://docs.microsoft.com/cs-cz/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.1) pro práce s `HttpClient` - nemusíte se starat o jeho životní cyklus sami, ale vše vyřesí interní implementace `IHttpClientFactory`.

### Registrace IHttpClientFactory
Pro registraci `IHttpClientFactory`, stačí zaregistrovat službu pomocí extension metody `.AddHttpClient()`, která zaregistruje veškeré závislosti.
```cs
services.AddHttpClient();
```

Poté kdekoliv v aplikaci si můžete vyžádat `IHttpClientFactory` službu pomocí DI. Služba obsahuje metodu `.CreateClient()`, která vrátí instanci HttpClient.

### Mockování IHttpClientFactory
Namockujeme si službu `IHttpClientFactory` pomocí mockovacího frameworku [NSubstitude](https://nsubstitute.github.io/).
```cs
 var httpClientFactoryMock = Substitute.For<IHttpClientFactory>();
 ```

 Dále si naimplementujeme fake implementaci `DelegatingHandler`, který se stará o o zpracování requestu a vrací příslušný response.

```cs
public class FakeHttpMessageHandler : DelegatingHandler
{
   private HttpResponseMessage _fakeResponse;

   public FakeHttpMessageHandler(HttpResponseMessage responseMessage)
   {
      _fakeResponse = responseMessage;
   }

   protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
   {
      return await Task.FromResult(_fakeResponse);
   }
}
```

Poté stačí tuhle fake implementaci podstrčit namockované `IHttpClientFactory`.
```cs
var fakeHttpMessageHandler = new FakeHttpMessageHandler(new HttpResponseMessage() 
{
    StatusCode = HttpStatusCode.OK,
});

var httpClient = new HttpClient(fakeHttpMessageHandler);

httpClientFactoryMock.CreateClient().Returns(httpClient);
```

A jako poslední krok zbývá vložit namockovanou instanci `IHttpClientFactory` do služby, kterou chcete testovat.


