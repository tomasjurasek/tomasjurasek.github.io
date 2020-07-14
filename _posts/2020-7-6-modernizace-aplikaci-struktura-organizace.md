---
layout: post
title: Modernizace aplikací - Struktura organizace
---

Důležitou roli v modernizaci aplikace hraje samotná komunikační struktura organizace, neboli [Conway Law](https://en.wikipedia.org/wiki/Conway%27s_law).

>"Organizace, které navrhují systémy jsou nuceny vytvářet návrhy, které jsou kopiemi komunikačních struktur těchno organizacích." - Conway, 1967

Ve většině případá to znamená, že v organizaci máte jednotlivé týmy na kokrétní oblast v jednotlivých témech - FE, BE, DB Doménový experti.


![Struktura Organizace](/images/posts/modernizace-struktura-organizace/struktura-org.jpg)

Tahle struktura vyprodukuje většinou systémy, které reprezentují samotnou organizaci (jednotlivé týmy).


![Struktura Organizace](/images/posts/modernizace-struktura-organizace/struktura-system.jpg)


Pokud chcete doručit systém, který bude rozdělen do nezávislých modulů (mikroslužeb) je potřeba poskládat týmy, které budou reprezentovat nezávislé jednotky a obsahovat veškeré specialisty na danou doménu.


![Struktura Organizace](/images/posts/modernizace-struktura-organizace/struktura-business.jpg)