---
layout: post
title: Share an infrastructure between the Table Storage and the Cosmos DB
---

This is a tutorial which provides you step by step how to share your infrastructure between the Table Storage and the Cosmos DB.

> Note: Your Cosmos DB must be set up with the Table API.

Why you want to migrate to Cosmos DB:
* Limit of the requests
* Throughput
* Scaling
* Indexing
* Size of the Entity


I have an application that is using the old Azure Storage SDK - **WindowsAzure.Storage** which is deprecated.

This is my ArticleRepository with two methods - Get and InsertOrMerge.


```cs
public class Article : TableEntity
{
    public string Text { get; set; }
}

public class ArticleRepository
{
    private const string _partitionKey = "Article";
    private readonly CloudTable _table;

    public ArticleRepository(string connectionString)
    {
        var storageAccount = CloudStorageAccount.Parse(connectionString);
        var tableClient = storageAccount.CreateCloudTableClient();
        _table = tableClient.GetTableReference("Articles");
    }

    public async Task<Article> Get(string id)
    {
        var tableOperation = TableOperation.Retrieve<Article>(_partitionKey, id);
        var result = await _table.ExecuteAsync(tableOperation);
        return (Article)result.Result;
    }

    public async Task<Article> InsertOrMerge(string text)
    {
        var articleEntity = new Article
        {
            PartitionKey = _partitionKey,
            RowKey = Guid.NewGuid().ToString(),
            Text = text
        };
        var tableOperation = TableOperation.InsertOrMerge(articleEntity);
        var result = await _table.ExecuteAsync(tableOperation);

        return (Article)result.Result;
    }
}
```

First of all we need to upgrade the Azure Storage SDK to the newest SDK - because this old SDK is deprecated. The new Azure Storage SDK is called **Microsoft.Azure.Cosmos.Table** which shares the infrastructure and services with the Cosmos DB. There are no breaking changes so you need only remove the old SDK package and install the newest SDK.

Then when we want to start using the Cosmos DB just change the **connectionString** from the Storage Account to the Cosmos DB.

