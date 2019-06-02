---
layout: post
title: Azure DevOps - CI/CD
---

[Azure DevOps](https://azure.microsoft.com/cs-cz/services/devops/) nabízí hned několik služeb pro správu vašeho projektu, které jsou mezi sebou propojené. Jedny z těchto služeb jsou taky CI/CD.

[Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) - je souhr úkolů, které integrují a validují práci jednotlivých vývojářů na projektu. Můžeme si to představit jako souhrn úkolů, které započnou od commitnutí změn do repozitáře, následný build a analýza kódu, až po spuštění automatických testů.

[Continuous Delivery](https://en.wikipedia.org/wiki/Continuous_delivery) - má potom za úkol doručit vytvořený build v CI někam na server.

Pro nastavení ***CI*** se musíme proklikat v Azure DevOps do záložky ***Pipelines*** a následně ***Builds***. Při vytváření nové build pipeline musíme první zvolit zdroj pro stahování zdrojových kódu. Máme hned několik možností - Azure DevOps Git, Gighub, SVN,...

![CI_CD](/images/posts/azure-devops-ci-cd/CI_RepoSource.png)

Já mám uložené zdrojové kody přímo v ***Azure DevOps Repos***, tak vyberu zdroj - ***Azure DevOps Git***. 
Dále si musím vybrat šablonu pro build, která nám vyhovuje. Jelikož se jedná o ASP.NET aplikaci, tak zvolíme šablonu ***ASP.NET***, která zahrnuje kroky - build, test.

![CI_CD](/images/posts/azure-devops-ci-cd/CI_Templates.png)

A dále nám vyskočí souhrn kroků vybrané šablony - nuget restore, build, test, publish artifacts, které můžeme libovolně dále upravovat.
Pokud bychom do pipeline chtěli přidat ještě analýzu kódu pomoci [SonarCloud](https://sonarcloud.io/about), tak stačí k Agentovi přidat další tasky - ***Prepare analysis on SonarCloud a Run Code Analysis***, které slouží pro integraci se zmíněným SonarCloudem.

Finální build pipeline potom bude vypadat následovně. Build pipeline se bude potom automaticky spouštět na základě změn ve dříve vybraném repozitáři a branche.

![CI_CD](/images/posts/azure-devops-ci-cd/CI_Final.png)

Tak a teď bychom potřebovali zaintegrované změny a buildy nějak dostat na náš server, aby změny byly viditelné.

To už se odehrává v záložce ***Releases***, kde vytvoříme novou release pipeline.
Musíme nastavit dva kroky. První z nich je abychom vybrali šablonu pro to, co se má vykonat za akci. V našem případě půjde o ***Azure App Service deployment***, které vyžaduje připojení na naše Azure předplatné a již vytvořenou službu Web App service, do které chcete aplikaci vypublikovat.

![CI_CD](/images/posts/azure-devops-ci-cd/CD_Artifacts.png)

A druhý krok je nastavit zdroj artefaktů. V našem případě budeme chtít vytvořené artefakty z naší CI build pipeline.¨

![CI_CD](/images/posts/azure-devops-ci-cd/CD_Template.png)

Pokud chceme, aby se release prováděl po každé úspěšné CI build pipelině, musíme ***zapnout trigger*** v nastavení artefaktů.

![CI_CD](/images/posts/azure-devops-ci-cd/CD_Final.png)

Vše uložímé a můžeme vyzkoušet celý CI/CD proces provedením změn v repozitáři..