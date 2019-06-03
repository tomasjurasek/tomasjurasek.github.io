---
layout: post
title: Azure DevOps - spouštění EF6 migrací při release procesu
---

[Entity Framework 6 Code First](https://docs.microsoft.com/cs-cz/ef/ef6/modeling/code-first/migrations/index) je přístup, který nám umožňuje popsat datový model pomocí C# tříd a na základě těchto tříd vytvořit migrační script, který se spustí nad databází.
 
V databázi se potom udržuje tabulka ***dbo.__MigrationHistory***, která uchovává informace o aplikovaných migracích.
 

EntityFramework nuget balíček obsahuje migrate.exe, který naleznete na adrese *packages\EntityFramework.{Version}\tools*. Migrate.exe tool umožňuje spustit EF migrace v příkazovém řádku.

Aby migrate.exe fungoval správně, musí se nacházet spolu s assembly, která obsahuje *migration configuration type*.

![EF6](/images/posts/ef6-migrate/migrate.PNG)

## 1. Build

Přidáme dva tasky typu ***Copy Files***. První vykopíruje migrate.exe do vámi určené speciální složky a druhý nakopíruje do stejné složky zkompilovanou aplikaci.



copy migrate.exe           |  copy build 
:-------------------------:|:-------------------------:
![EF6](/images/posts/ef6-migrate/copy1.PNG)  |  ![EF6](/images/posts/ef6-migrate/copy2.PNG)

## 2. Release 

Přidáme task typu ***Command Line***, kde spustíme soubor migrate.exe s parametry connectionString a connectionProviderName. $(connectionstring) je proměnná, jejiž hodnotu máme uloženou v release variables.

![EF6](/images/posts/ef6-migrate/cl.PNG) 