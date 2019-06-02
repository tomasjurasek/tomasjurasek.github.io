---
layout: post
title: Použití MERGE SQL
---


[Merge](https://docs.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-2017) je SQL příkaz, který umí porovnat dvě tabulky a na základě podmínky provádí INSERT, UPDATE nebo DELETE.

### **Syntaxe:**
```sql
MERGE TARGET_TABLE AS TARGET
USING SOURCE_TABLE AS SOURCE
ON
WHEN MATCHED THEN
WHEN NOT MATCHED BY TARGET THEN
WHEN NOT MATCHED BY SOURCE THEN;
```

### ***Příklad:***
Potřebuji do tabulky vložit záznam pokud neexistuje. V opačném případě upravím jeden sloupec.
První co nás může napadnout je použití samostatných příkazů SELECT a na základě výsledku provést INSERT nebo UPDATE.

```cs
var exists = connection.QueryFirstOrDefault<bool>("SELECT 1 FROM [dbo].[Products] WHERE ProductId1 = @ProductId1 AND ProductId2 = @ProductId2 AND Date = @Date", 
new
{
    up.ProductId1,
    up.ProductId2,
    up.DateTime.Date
});

if (exists)
{
    connection.Execute("UPDATE [dbo].[Products] SET Count = Count+1 WHERE ProductId1 = @ProductId1 AND ProductId2 = @ProductId2 AND Date = @Date", 
    new
    {
        up.ProductId1,
        up.ProductId2,
        up.DateTime.Date
    });
}
else
{
    connection.Execute("INSERT INTO [dbo].[Products] ([ProductId1], [ProductId2], [Date], [Count]) VALUES (@ProductId1, @ProductId2, @Date, @Quantity)", 
    new
    {
        up.ProductId1,
        up.ProductId2,
        up.DateTime.Date,
        up.Quantity
    });
}
```

Abychom si ušetřili několik řádků a zbytečně vytěžovali databázi několika příkazi, můžeme použít MERGE SQL, který dokáže na základě výsledku z porovnání tabulek provést buď INSERT nebo UPDATE.

```cs
await connection.ExecuteAsync(@"
MERGE Products
AS target USING(SELECT @ProductId1, @ProductId2, @Date, @Quantity, @LanguageId) 
AS source(ProductId1, ProductId2, Date, Quantity, LanguageId)

ON(target.ProductId1 = source.ProductId1 AND target.ProductId2 = source.ProductId2 AND target.Date = source.Date AND target.LanguageId = source.LanguageId)

WHEN MATCHED THEN
UPDATE SET Quantity = @Quantity + 1

WHEN NOT MATCHED THEN
INSERT(ProductId1, ProductId2, Date, Quantity, LanguageId) VALUES(source.ProductId1, source.ProductId2, source.Date, source.Quantity, source.LanguageId);", 
new
{
    up.ProductId1,
    up.ProductId2,
    up.DateTime.Date,
    up.LanguageId,
    Quantity = 1,
});
```