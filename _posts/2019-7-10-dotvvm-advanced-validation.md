---
layout: post
title: DotVVM - validace formulářů
---
Ještě neznáte framework [DotVVM](https://www.dotvvm.com/)?


[DotVVM](https://www.dotvvm.com/) podporuje model validace, které určitě znáte z jiných ASP.NET technologiích jako MVC, Web API,..

***Příklad -*** máme jednoduchý formulář, kde zadáváme základní informace o uživateli - jméno, příjmení, email, které chceme validovat.
  
---

**1. Data Annotations**

Pomocí jednoduchých data annotations můžeme zvalidovat hodnoty v objektu, které přicházejí na server. DotVVM umí taky [několik attributů](https://www.dotvvm.com/docs/tutorials/basics-client-side-validation/2.0) přeložit přímo na klientské validace, takže pro jednoduché scénáře nepotřebujete žádnou javascriptovou knihovnu na klientské validace.

```cs
public class UserForm
{
    [Required]
    public string FirstName { get; set; }
    [Required]
    public string LastName { get; set; }
    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
```
---
**2. IValidatableObject**

Objekt, který chceme validovat musí dědit z interface IValidatableObject, který implementuje metodu Validate.
```cs
public class UserForm : IValidatableObject
{
    [Required]
    public string FirstName { get; set; }
    [Required]
    public string LastName { get; set; }
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (FirstName.Contains("."))
        {
            yield return new ValidationResult("Error Message", new string[] { nameof(FirstName) });
        }
    }
}
```

---

**3. Složitější validace**

Pro složitější validace - potřebujete ověřit data v databázi a na základě výsledku vyvolat validační chybu, můžete využít extension metodu v DotVVM -  [AddModelError](https://github.com/riganti/dotvvm/blob/c49b55e079b3aeb5fa30d5818675a86f656a33d6/src/DotVVM.Framework/ViewModel/Validation/ValidationErrorFactory.cs#L22), která přiřadí chybovou hlášku k hodnotě.

Potom při zavolání metody Save se hodnota s emailem zkontroluje v databázi a pokud email již existuje, vytvořím chybovou hlášku. 
Na konci je potřeba zavolat metodu [FailOnInvalidModelState](https://github.com/riganti/dotvvm/blob/ce6ce869556e9fc22e8e9286fabd8127569891f3/src/DotVVM.Framework/Hosting/DotvvmRequestContextExtensions.cs#L139) na Contextu, která ukončí request a vrátí response pro stránku.
```cs
public void Save()
{
    var emailExists = userService.CheckIfEmailExists(UserForm.Email);
    if (emailExists)
    {
        this.AddModelError(v => v.UserForm.Email, "Error message");
        Context.FailOnInvalidModelState();
    }
}
```

---

**Výsledný zdrojový kód**

*Dothtml*

```html
<form Validator.InvalidCssClass="error">
    <div>
        <dot:TextBox Text="{value: UserForm.FirstName}" Validator.Value="{value: UserForm.FirstName}" />
    </div>
    <div>
        <dot:TextBox Text="{value: UserForm.LastName}" Validator.Value="{value: UserForm.LastName}" />
    </div>
    <div>
        <dot:TextBox Text="{value: UserForm.Email}" Validator.Value="{value: UserForm.Email}" />
    </div>
    <dot:Button Click="{command: Save()}" Text="Save" />
</form>
```
*ViewModel*
```cs
public class DefaultViewModel : MasterPageViewModel
{
    private readonly IUserService userService;
    public UserForm UserForm { get; set; } = new UserForm();
    public DefaultViewModel(IUserService userService)
    {
        this.userService = userService;
    }


    public void Save()
    {
        var emailExists = userService.CheckIfEmailExists(UserForm.Email);
        if (emailExists)
        {
            this.AddModelError(v => v.UserForm.Email, "The e-mail address is already registered!");
            Context.FailOnInvalidModelState();
        }
    }
}
```


Více o validacích v DotVVM si můžete přečíst [zde](https://www.dotvvm.com/docs/tutorials/basics-validation/2.0).