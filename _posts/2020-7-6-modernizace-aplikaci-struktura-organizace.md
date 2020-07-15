---
layout: post
title: Modernizace aplikací - Struktura organizace
---

Důležitou roli v modernizaci aplikace hraje samotná komunikační struktura organizace, neboli [Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law).

>"Organizace, které navrhují systémy jsou nuceny vytvářet návrhy, které jsou kopiemi komunikačních struktur těchno organizacích." - Conway, 1967

Ve většině případů to znamená, že v organizaci jsou jednotlivé týmy postavené podle jejich specializace - FE, BE, DB ...


![Struktura Organizace](/images/posts/modernizace-struktura-organizace/struktura-org.jpg)

Tahle struktura vyprodukuje většinou systémy, které reprezentují samotnou organizaci (jednotlivé týmy).


![Struktura Organizace](/images/posts/modernizace-struktura-organizace/struktura-system.jpg)


Pokud chcete doručit systém, který bude rozdělen do nezávislých modulů (mikroslužeb) je potřeba poskládat týmy, které budou reprezentovat nezávislé jednotky a obsahovat veškeré specialisty na danou doménu.

Pokud je cíl organizace vybudovat "nezávislé" moduly (mikroslužby) aplikace, které budou mezi sebou komunikovat, tak bez reorganizace a nezávislosti samotných týmu to nebude pravděpodobně proveditelné. Každý modul (mikroslužby) by měl  obsahovat potřebné specialisty, kteří spolu dodají celek.
![Struktura Organizace](/images/posts/modernizace-struktura-organizace/struktura-business.jpg)



Organizace většinou mají představu o tom, jak by jejich artchitektura systému měla vypadat. Zapomínají ale nato, zda jsou jejich týmy přizpůsobeny těmto představám - struktura týmu reflektuje architektonický směr.
