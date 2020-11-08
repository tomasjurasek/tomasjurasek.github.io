---
layout: post
title: Azure Blob Storage - Lifecycle Management
---
The [Lifecycle Management](https://azure.microsoft.com/cs-cz/blog/azure-blob-storage-lifecycle-management-now-generally-available/) is a feature of the [Blob Storage](https://docs.microsoft.com/en-US/azure/storage/blobs/storage-blobs-introduction) which can manage access tier or delete your blobs.

This feature is used to optimize the pricing of your Blob Storage by changing the **Access tier** or **Delete** your blobs. Pricing of the blob depends on the selected access tier.

You can define a **rule** with a time period and a filter that can run the selected operations on your blobs.

### Change the Access tier

A blob can contain one of these *access tiers*. 

#### Hot 
* High availability
* It Is used for most frequently blob access
* $0,0196 per month for GB

#### Cool
* Access to this tier of the blob can take some time
* The blob must be stored for minimum 30 days
* It is used for the last backup or the not frequented blob
* $0,01 per month for GB

#### Archive
* Offline access (dehydration)
* The blob must be stored for minimum 180 days
* $0,0018 per month for GB

### Delete
Delete blobs by the rule. This operation is free.

