---
layout: post
title: DotVVM - použití komponenty Validator
---

Ještě pořád neznáte framework [DotVVM](https://www.dotvvm.com/)?

Komponenta [Validator](https://www.dotvvm.com/docs/controls/builtin/Validator/latest) slouží pro zobrazení validačních hlášek, které jsou součástí [Validation.Target](https://www.dotvvm.com/docs/tutorials/basics-validation-target/latest). Pokud není [Validation.Target](https://www.dotvvm.com/docs/tutorials/basics-validation-target/latest) nastavený, validuje se celý ViewModel.

**Proměnné**
* *InvalidCssClass* - nastaví css třídu při nevalidním stavu.
* *SetToolTipText* - nastaví validační hlášku jako tooltip na kontrolce
* *ShoErrorMessageText* - zobrazí validační zprávu
* *Value* - hodnota, které se validuje

**Použítí**

Validator můžeme použít jako samostatnou komponentu a proměnné nastavit přímo.
```cs
<dot:Validator Value="{value: Text}" InvalidCssClass="error" .. />
```

Nebo můžeme všechny proměnné použít na jakékoliv DotVVM kontrolce.

```cs
 <dot:TextBox Text="{value: Text}" Validator.Value="{value: Text}" InvalidCssClass="error" .. />
```

Pokud chcete udělat kompletní výpis všech validačních hlášek, můžete použít komponentu [ValidationSummary](https://www.dotvvm.com/docs/controls/builtin/ValidationSummary/2.0).

```cs
<dot:ValidationSummary />
```


**Příklad**

Na stránce máme dva formuláře, které budeme postupně zobrazovat a data budeme validovat jen v aktuálním formuláři, který je právě zobrazený.

Vytvoříme dva objekty, které reprezentují formuláře s validacemi a metodu, která přepíná formuláře. Pokud jsou data validní, tak se zobrazí další formulář.
```cs
public class DefaultViewModel : MasterPageViewModel
{
    public Form_1 Form_1 { get; set; } = new Form_1();
    public Form_1 Form_2 { get; set; } = new Form_2();

    public int CurrentForm { get; set; } = 1;
    public void Next()
    {
        if (CurrentForm == 1)
        {
            CurrentForm += 1;
        }
    }
}

public class Form_1
{
    [Required]
    public string FirstName { get; set; }
    [Required]
    public string LastName { get; set; }
}

public class Form_2
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }
}
```

U jednotlivých formulářů musíme nastavit [Validation.Target](https://www.dotvvm.com/docs/tutorials/basics-validation-target/latest), aby jsme při kliknutí na tlačítko *Další* validovali jen aktuální formulář.

```html
<div Visible="{value: _root.CurrentForm == 1}" Validator.ShowErrorMessageText="true">
    <form Validation.Target="{value: Form_1}">
        <div>
            <dot:TextBox Text="{value: Form_1.FirstName}" Validator.Value="{value: Form_1.FirstName}" />
        </div>
        <div>
            <dot:TextBox Text="{value: Form_1.LastName}" Validator.Value="{value: Form_1.LastName}" />
        </div>
        <dot:Button Click="{command: Next()}" Text="Další" />
    </form>
</div>
<div Visible="{value: _root.CurrentForm == 2}" Validator.ShowErrorMessageText="true">
    <form Validation.Target="{value: Form_2}">
        <div>
            <dot:TextBox Text="{value: Form_2.Email}" Validator.Value="{value: Form_2.Email}" />
        </div>
        <dot:Button Click="{command: Next()}" Text="Další" />
    </form>
</div>

```



