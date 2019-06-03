---
layout: post
title: ASP.NET Core - Policy-Based Autorizace
---

Pokud jste dříve pracovali s autorizací v ASP.NET, tak nebylo moc jednoduchých možností jak autorizovat uživatele jinak než podle Rolí - *[Authorize(Roles="Admin")]*.

Při složitejších scénářích jste si museli vše řešit pomocí vlastní infrastruktury. Pomocí *Policy-Based* autorizaci máte více možností jak tyhle scénáře jednoduše řešit.

### ***Policy-Based***

Pomocí Policy-Based můžete jednoduše:
* Autorizovat uživatele více způsoby - Role, Claim,...
* Implementovat vlastní způsob autorizace
* Oddělit logiku autorizace do vlastní knihovny
* Lépe testovat
* Mít méně vlastní infrastruktury

Policy je potřeba zaregistrovat na IServiceCollection - *AddAuthorization* na *AuthorizationOptions*.

```cs
services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOrEditor", policy => policy.RequireRole("Admin", "Editor"));
});
```

Potom stačí nad zdroj uvést *[Authorize(Policy="AdminOrEditor")]*. [AuthorizationPolicyBuilder](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/AuthorizationPolicyBuilder.cs) obsahuje několik základních metod, které se hodí na jednoduché scénáře:

* RequireRole
* RequireClaim
* RequireUserName
* RequireAssertion
* RequireAuthenticatedUser

Pokud se blíže podíváme na implementaci [RequireRole](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/AuthorizationPolicyBuilder.cs#L168), tak zjistíme, že metoda dělá základní validace a přidává do kolekce Requirements implementaci IAuthorozationRequirement -  [RoleAuthorizationRequirement](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/RolesAuthorizationRequirement.cs).

Pomocí těchto tříd můžete jednoduše implementovat vlastní autorizaci.
* [IAuthorizationRequirement](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/IAuthorizationRequirement.cs)   - slouží jako vlastní model vašeho requirementu
* [AuthorizationHandler<TRequirement>](https://github.com/aspnet/AspNetCore/blob/62351067ff4c1401556725b401478e648b66acdc/src/Security/Authorization/Core/src/AuthorizationHandler.cs)  - obsahuje abstraktní metodu HandleRequirementAsync, kterou musíte implementovat a provést vlastní autorizační logiku.

***Příklad*** - máme zdroj, kde mají přístup jen uživatelé, kteří mají email společnosti - @company.com.

Vytvoříme třídu poděděním IAuthorizeRequirement a vytvoříme vlastní requirement.

```cs
public class CompanyDomainRequirement : IAuthorizationRequirement
{
    public CompanyDomainRequirement(string domain)
    {
        Domain = domain;
    }

    public string Domain { get; }
}
```
Potom vytvoříme handler, který bude kontrolovat email uživatele a na základě úspěšné validace se zavolá *context.Succeed(requirement)*.

```cs
public class CompanyEmailHandler : AuthorizationHandler<CompanyDomainRequirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, CompanyDomainRequirement requirement)
    {
        if (context.User.HasClaim(s => s.Type == ClaimTypes.Email))
        {
            var claim = context.User.FindFirst(s => s.Type == ClaimTypes.Email);
            if (claim.Value.Contains($"@{requirement.Domain}"))
            {
                context.Succeed(requirement);
            }
        }
        return Task.CompletedTask;
    }
}
```
A nakonec stačí k vytvořené policy zaregistrovat náš requirement a handler.
```cs
services.AddAuthorization(options =>
{
    options.AddPolicy("CompanyEmailPolicy", policy => policy.Requirements.Add(new CompanyDomainRequirement("company.com")));
});
services.AddSingleton<IAuthorizationHandler, CompanyEmailHandler>();
```
Potom stačí na konkrétním zdroji použít policy, která umožnít přístup jen uživatelů, s firemním emailem - *[Authorize(Policy="CompanyEmailPolicy")]*.