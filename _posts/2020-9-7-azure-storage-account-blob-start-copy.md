---
layout: post
title: Azure Blob Storage - StartCopyFromUri
---

Funkce `StartCopyFromUri` ve službě [Azure Blob Storage](https://docs.microsoft.com/cs-cz/azure/storage/blobs/storage-blobs-overview) slouží pro nepřímé kopírování vašich blobů v rámci jednotlivých Storage Accountů - tzv. pomocí odkazu na cizí Blob, můžete pomocí `StartCopyFromUri` zahájit asynchroní kopírování Blobu na váš Storage Account, který se děje na pozadí služby [Azure Blob Storage](https://docs.microsoft.com/cs-cz/azure/storage/blobs/storage-blobs-overview). 

Stav kopírovaného Blobu potom jen můžete kontrolovat pomocí proměnné `CopyStatus`, která znázorňuje stav.

Dále můžete pracovat s metodama
* [SyncCopyFromUriAsync](https://docs.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.synccopyfromuriasync?view=azure-dotnet)
* [AbortCopyFromUriAsync](https://docs.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.abortcopyfromuriasync?view=azure-dotnet)


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
            var sourceBlobUrl = "";

            var blobServiceClient = new BlobServiceClient(connectionString);
            var blobContainerClient = await blobServiceClient.CreateBlobContainerAsync(containerName);
            var blobClient = blobContainerClient.Value.GetBlobClient(blobName);
            
            await blobClient.StartCopyFromUriAsync(new Uri(sourceBlobUrl));

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


