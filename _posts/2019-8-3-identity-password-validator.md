---
layout: post
title: ASP.NET Core Identity - pravidla pro tvorbu hesla
---

[ASP.NET Core Identity](https://docs.microsoft.com/cs-cz/aspnet/core/security/?view=aspnetcore-2.2) slouží pro správú uživatelů, identity a rolí ve vaší ASP.NET aplikaci. Systém nabízí základní sadu pravidel, které je možné nastavit v [IdentityOptions](https://bit.ly/2yyUjkj). Základní pravidla pro se nachází v [PasswordOptions](https://bit.ly/2yAK92G)
Tyto pravidla můžeme upravovat přímo u registrace [ASP.NET Core Identity](https://docs.microsoft.com/cs-cz/aspnet/core/security/?view=aspnetcore-2.2).

```cs
services.AddIdentity<User, Role>(options => 
{
    options.Password.RequiredLength = 10;
    ...
});
```
Pravidla potom zpracovávají tzv validátory. Pro validaci hesla systém nabízí defaulní implementaci [PasswordValidator<TUser>](https://github.com/aspnet/AspNetCore/blob/bfec2c14be1e65f7dd361a43950d4c848ad0cd35/src/Identity/Extensions.Core/src/PasswordValidator.cs), kde se tyhle pravidla aplikují.

Pokud potřebujete pravidlo pro validování hesla, které identity systém nenabízí, můžete využít interface [IPasswordValidator<TUser>](https://github.com/aspnet/AspNetCore/blob/bfec2c14be1e65f7dd361a43950d4c848ad0cd35/src/Identity/Extensions.Core/src/IPasswordValidator.cs) a naimplementovat si vlastní pravidlo.


***Příklad*** - chceme mít validaci pro heslo, které zakáže, aby heslo bylo stejné jako username.
```cs
public class UserNamePasswordValidator : IPasswordValidator<User>
{
    public Task<IdentityResult> ValidateAsync(UserManager<User> manager, User user, string password)
    {
        if (password.Contains(user.UserName, System.StringComparison.OrdinalIgnoreCase))
        {
            return Task.FromResult(IdentityResult.Failed(new IdentityError
            {
                Code = "UserNameInPassword",
                Description = "Password cannot contain the username"
            }));
        }

        return Task.FromResult(IdentityResult.Success);
    }
}
```


Následně validátor musíme zaregistrovat do identity pipeline.
```cs
services.AddIdentity<User, Role>()
        .AddPasswordValidator<UserNamePasswordValidator>();
```

Potom všechny validátory jsou uloženy ve tříde UserManager v kolekci PasswordValidators.

***Použití:***
```cs
...
var validators = _userManager.PasswordValidators;
            
foreach (var validator in validators)
{
    var result = await validator.ValidateAsync(_userManager, user, password);
    ...
}
...

```

Pomocí validátoru můžete uživateli dát okamžitou zpětnou vazbu pokud heslo není validní, a nemusíte čekat až na pokus uložení do DB.