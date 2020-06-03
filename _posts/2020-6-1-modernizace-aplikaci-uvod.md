---
layout: post
title: Modernizace aplikací - Úvod
---

Mikroslužby, škálovatelné distribuované systémy běžící v docker konteinerech v Kubernetes.

Tohle se v poslední době objevuje v článcích jako jediná možné cesta jak modernizovat Vaše aplikace. Nic ale není *černobíle*, ani problematika modernizace a refaktoringu.

> Každý systém *nemůže* být postavený na mikroslužbách. A ani by neměl.

Mikroslužby jsou jen špička ledovce. Hodí se jen na malé procento všech technologických projektů - kritické scénáře.

> Je potřeba myslet i na těch 90% systémů, které se nachází pod hladinou ledovce.

Pokud se Vám daří zůstat střízlivý ohledně přepisu do mikroslužeb, tak máte napůl vyhráno. Držte se v kontextu Vašeho businessu a pokuste se myslet více do budoucna, aby nebylo potřeba Váš systém znovu za pár let přepisovat. Jako nedávno zahlásil Uber, že mikroslužby pro ně jsou dost velké sousto (cca 4000 mikroslužeb) a vracejí se spíše ke "službám" - [odkaz](https://twitter.com/GergelyOrosz/status/1247132806041546754).

> Není kam spěchat. Podle cyklu technologií se může stát, že za pár let bude trendy mít monolitický systém hostovaný na vlastních serverech.

Postupně si budeme ukazovat možné příklady, jak se postavit čelem k modernizaci Vašich systémů a nepropálit natom majlant.

