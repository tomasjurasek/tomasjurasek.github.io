---
layout: post
title: Azure Blob Storage - StartCopyFromUri
---

Funkce `StartCopyFromUri` ve službě Azure Blob Storage slouží pro nepřímé kopírování vašich blobů v rámci jednotlivých Storage Accountů - tzv. pomocí odkazu na cizí Blob, můžete pomocí `StartCopyFromUri` zahájit kopírování Blobu na váš Storage Account, který asynchroně provádí služba Azure Blob Storage. U kopírovaného Blobu potom jen můžete kontrolovat proměnou `CopyStatus`, která znázorňuje stav kopírování.


```cs
using Azure.Storage.Blobs;
using System;
using System.Threading.Tasks;

namespace AzureStorageBlobCopy
{
    class Program
    {
        static async Task Main(string[] args)
        {
            var connectionString = "";
            var containerName = "";
            var blobName = "";
            var sourceUrl = "";

            var blobServiceClient = new BlobServiceClient(connectionString);
            var blobContainerClient = await blobServiceClient.CreateBlobContainerAsync(containerName);
            var blobClient = blobContainerClient.Value.GetBlobClient(blobName);
            await blobClient.StartCopyFromUriAsync(new Uri(sourceUrl));

            while (true)
            {
                var properties = await blobClient.GetPropertiesAsync();
                if(properties.Value.CopyStatus == Azure.Storage.Blobs.Models.CopyStatus.Success)
                {
                    //Logic
                    break;
                }
                
                await Task.Delay(1000);
            }
        }
    }
}
```


