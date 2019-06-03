---
layout: post
title: ASP.NET Core - Resource-Based Autorizace
---

V minulém článku jsme si ukázali jak vytvořit a využít policy-based autorizaci pomocí autorizačního atributu **[Authorize(Policy = "PolicyName")]**.

Dnes si ukážeme jak využít policy pro *resource-based* autorizaci - autorizace na úrovni objektů.

Pokud se podíváme do implementace metody [AddAuthorization](https://github.com/aspnet/AspNetCore/blob/7669cf0d036cc0cb3b481aba49253cd35d85c633/src/Security/Authorization/Core/src/AuthorizationServiceCollectionExtensions.cs#L22), kterou registrujeme služby pro autorizaci ve Startup.cs, tak můžeme vidět několik dalších zaregistrovaných služeb. Kromě základních handlerů a providerů pro Autorizaci se zde nachází taky registrace služby [IAuthorizationService](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/DefaultAuthorizationService.cs#L17), která obsahuje dvě metody:
```cs
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, object resource, string policyName) 
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, object resource, IEnumerable<IAuthorizationRequirement> requirements)
```
V první metodě můžeme využít zaregistrované policy a zadat jen její název, a nebo můžete přímo předat kolekci implementací [IAuthorizationRequirement](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/IAuthorizationRequirement.cs), které slouží jako zdroj dat pro vyhodnocení autorizačního handleru.

***Příklad*** - máme zdroj (metodu kontroleru) a vrací detail článku, který může získat jen její autor.

Vytvoříme autorizační requirement a handler:
```cs
public class AuthorAuthorizationRequirement : IAuthorizationRequirement{ }

public class AuthorAuthorizationHandler : AuthorizationHandler<AuthorAuthorizationRequirement, Article>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, AuthorAuthorizationRequirement requirement, Article resource)
    {
        if(context.User.Identity.Name == resource.Author) //Pokud se autentizovaný uživatel shoduje s autorem článku, má právo získat článek
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}
```

A potom hadler i policy zaregistrujeme:
```cs
services.AddAuthorization(options =>
{
     options.AddPolicy("Editor", policy => policy.Requirements.Add(new AuthorAuthorizationRequirement()));
});
services.AddSingleton<IAuthorizationHandler, AuthorAuthorizationHandler>();
```
Pomocí dependency injection získáme základní implementaci *IAuthorizationService* a zavoláme metodu *AuthorizeAsync*.
```cs
[Route("articles")]
[ApiController]
public class ArticlesController : ControllerBase
{
    private readonly IAuthorizationService authorizationService;
    private readonly IArticleService articleService;

    public ArticlesController(IAuthorizationService authorizationService, IArticleService articleService)
    {
        this.authorizationService = authorizationService;
        this.articleService = articleService;
    }
 
    [HttpGet]
    [Route("{id}")]
    public async Task<IActionResult> Get(int id)
    {
        var article = articleService.GetArticle(id); //Získáme článek
        var authResult = await authorizationService.AuthorizeAsync(User, article, "Editor"); //Ověříme zda má právo
        if(authResult.Succeeded)
        {
            return Ok(article);
        }
        return Forbid();
    }
}
```