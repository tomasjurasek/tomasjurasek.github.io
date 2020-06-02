---
layout: post
title: Modernizace aplikací - Úvod
---

Mikroslužby, škálovatelné distribuované systémy běžící v docker konteinerech v Kubernetes.

Tohle se v poslední době oběvuje v článcích jako jediná možné cesta jak modernizovat Vaše aplikace. Nic ale není *černobíle*, ani problematiku modernizace a refaktoringu.


> Každý systém *nemůže* být postavený na mikroslužbách.

Mikroslužby jsou jen špička ledovce, které se hodí na malé procentu všech technologických projektů - kritické scénáře.


> Je potřeba myslet i na těch 90% systémů, které se nachází pod hladinou ledovce.

Zkuste nepodlehnout tlaku nejnovějších technologii a držet se v kontextu Vašeho businessu. Myslete na budoucnost, aby nebylo třeba Váš systém znovu za pár let přepisovat. Jako nedávno zahlásil Uber, že mikroslužby pro ně jsou dost velké sousto (cca 4000 mikroslužeb) - [odkaz](http://highscalability.com/blog/2020/4/8/one-team-at-uber-is-moving-from-microservices-to-macroservic.html).


Není kam spěchat. Podle cyklu technologií, se může stát, že za pár let bude trendy mít monolith hostovaný na vlastním serveru.



V dalších článcích se Vám pokusím popsat případy a jejich možné řešení, které žažívám u svých klientů, kteří svoje aplikace modernizují, nebo chtějí migrovat na cloud.
