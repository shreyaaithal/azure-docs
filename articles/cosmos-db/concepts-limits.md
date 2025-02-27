---
title: Azure Cosmos DB service quotas
description: Azure Cosmos DB service quotas and default limits on different resource types.
author: abhijitpai
ms.author: abpai
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 10/25/2021
---

# Azure Cosmos DB service quotas

[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

This article provides an overview of the default quotas offered to different resources in the Azure Cosmos DB.

## Storage and database operations

After you create an Azure Cosmos account under your subscription, you can manage data in your account by [creating databases, containers, and items](account-databases-containers-items.md).

### Provisioned throughput

You can provision throughput at a container-level or a database-level in terms of [request units (RU/s or RUs)](request-units.md). The following table lists the limits for storage and throughput per container/database. Storage refers to the combined amount of data and index storage.

| Resource | Default limit |
| --- | --- |
| Maximum RUs per container ([dedicated throughput provisioned mode](account-databases-containers-items.md#azure-cosmos-containers)) | 1,000,000 by default. You can increase it by [filing an Azure support ticket](create-support-request-quota-increase.md) |
| Maximum RUs per database ([shared throughput provisioned mode](account-databases-containers-items.md#azure-cosmos-containers)) | 1,000,000 by default. You can increase it by [filing an Azure support ticket](create-support-request-quota-increase.md) |
| Maximum RUs per partition (logical & physical) | 10,000 |
| Maximum storage across all items per (logical) partition | 20 GB |
| Maximum number of distinct (logical) partition keys | Unlimited |
| Maximum storage per container | Unlimited |
| Maximum storage per database | Unlimited |
| Maximum attachment size per Account (Attachment feature is being deprecated) | 2 GB |
| Minimum RU/s required per 1 GB | 10 RU/s<br>**Note:** this minimum can be lowered if your account is eligible to our ["high storage / low throughput" program](set-throughput.md#high-storage-low-throughput-program) |

> [!NOTE]
> To learn about best practices for managing workloads that have partition keys requiring higher limits for storage or throughput, see [Create a synthetic partition key](synthetic-partition-keys.md).

### Minimum throughput limits

A Cosmos container (or shared throughput database) must have a minimum throughput of 400 RU/s. As the container grows, Cosmos DB requires a minimum throughput to ensure the database or container has sufficient resource for its operations.

The current and minimum throughput of a container or a database can be retrieved from the Azure portal or the SDKs. For more information, see [Provision throughput on containers and databases](set-throughput.md). 

The actual minimum RU/s may vary depending on your account configuration. You can use [Azure Monitor metrics](monitor-cosmos-db.md#view-operation-level-metrics-for-azure-cosmos-db) to view the history of provisioned throughput (RU/s) and storage on a resource. 

#### Minimum throughput on container 

To estimate the minimum throughput required of a container with manual throughput, find the maximum of:

* 400 RU/s 
* Current storage in GB * 10 RU/s
* Highest RU/s ever provisioned on the container / 100

Example: Suppose you have a container provisioned with 400 RU/s and 0 GB storage. You increase the throughput to 50,000 RU/s and import 20 GB of data. The minimum RU/s is now `MAX(400, 20 * 10 RU/s per GB, 50,000 RU/s / 100)` = 500 RU/s. Over time, the storage grows to 200 GB. The minimum RU/s is now `MAX(400, 200 * 10 RU/s per GB, 50,000 / 100)` = 2000 RU/s. 

**Note:** the minimum throughput of 10 RU/s per GB of storage can be lowered if your account is eligible to our ["high storage / low throughput" program](set-throughput.md#high-storage-low-throughput-program).

#### Minimum throughput on shared throughput database 
To estimate the minimum throughput required of a shared throughput database with manual throughput, find the maximum of:

* 400 RU/s 
* Current storage in GB * 10 RU/s
* Highest RU/s ever provisioned on the database / 100
* 400 + MAX(Container count - 25, 0) * 100 RU/s

Example: Suppose you have a database provisioned with 400 RU/s, 15 GB of storage, and 10 containers. The minimum RU/s is `MAX(400, 15 * 10 RU/s per GB, 400 / 100, 400 + 0 )` = 400 RU/s. If there were 30 containers in the database, the minimum RU/s would be `400 + MAX(30 - 25, 0) * 100 RU/s` = 900 RU/s. 

**Note:** the minimum throughput of 10 RU/s per GB of storage can be lowered if your account is eligible to our ["high storage / low throughput" program](set-throughput.md#high-storage-low-throughput-program).

In summary, here are the minimum provisioned RU limits. 

| Resource | Default limit |
| --- | --- |
| Minimum RUs per container ([dedicated throughput provisioned mode](./account-databases-containers-items.md#azure-cosmos-containers)) | 400 |
| Minimum RUs per database ([shared throughput provisioned mode](./account-databases-containers-items.md#azure-cosmos-containers)) | 400 RU/s for first 25 containers. Additional 100 RU/s for each container afterward. |

Cosmos DB supports programmatic scaling of throughput (RU/s) per container or database via the SDKs or portal.    

Depending on the current RU/s provisioned and resource settings, each resource can scale synchronously and immediately between the minimum RU/s to up to 100x the minimum RU/s. If the requested throughput value is outside the range, scaling is performed asynchronously. Asynchronous scaling may take minutes to hours to complete depending on the requested throughput and data storage size in the container.  

### Serverless

[Serverless](serverless.md) lets you use your Azure Cosmos DB resources in a consumption-based fashion. The following table lists the limits for storage and throughput burstability per container/database.

| Resource | Limit |
| --- | --- |
| Maximum RU/s per container | 5,000 |
| Maximum storage across all items per (logical) partition | 20 GB |
| Maximum number of distinct (logical) partition keys | Unlimited |
| Maximum storage per container | 50 GB |

## Control plane operations

You can [provision and manage your Azure Cosmos account](how-to-manage-database-account.md) using the Azure portal, Azure PowerShell, Azure CLI, and Azure Resource Manager templates. The following table lists the limits per subscription, account, and number of operations.

| Resource | Default limit |
| --- | --- |
| Maximum database accounts per subscription | 50 by default. You can increase it by [filing an Azure support ticket](create-support-request-quota-increase.md) up to 1,000 max.|
| Maximum number of regional failovers | 1/hour by default. You can increase it by [filing an Azure support ticket](create-support-request-quota-increase.md)|

> [!NOTE]
> Regional failovers only apply to single region writes accounts. Multi-region write accounts do not require or have any limits on changing the write region.

Cosmos DB automatically takes backups of your data at regular intervals. For details on backup retention intervals and windows, see [Online backup and on-demand data restore in Azure Cosmos DB](online-backup-and-restore.md).

## Per-account limits

### Provisioned throughput

| Resource | Default limit |
| --- | --- |
| Maximum number of databases | 500 |
| Maximum number of containers per database with shared throughput |25 |
| Maximum number of containers per database or account with dedicated throughput  | 500 |
| Maximum number of regions | No limit (All Azure regions) |

### Serverless

| Resource | Limit |
| --- | --- |
| Maximum number of containers per account  | 100 |
| Maximum number of regions | 1 (Any Azure region) |

## Per-container limits

Depending on which API you use, an Azure Cosmos container can represent either a collection, a table, or graph. Containers support configurations for [unique key constraints](unique-keys.md), [stored procedures, triggers, and UDFs](stored-procedures-triggers-udfs.md), and [indexing policy](how-to-manage-indexing-policy.md). The following table lists the limits specific to configurations within a container. 

| Resource | Default limit |
| --- | --- |
| Maximum length of database or container name | 255 |
| Maximum stored procedures per container | 100 <sup>*</sup>|
| Maximum UDFs per container | 50 <sup>*</sup>|
| Maximum number of paths in indexing policy| 100 <sup>*</sup>|
| Maximum number of unique keys per container|10 <sup>*</sup>|
| Maximum number of paths per unique key constraint|16 <sup>*</sup>|
| Maximum TTL value |2147483647|

<sup>*</sup> You can increase any of these per-container limits by creating an [Azure Support request](create-support-request-quota-increase.md).

## Per-item limits

Depending on which API you use, an Azure Cosmos item can represent either a document in a collection, a row in a table, or a node or edge in a graph. The following table shows the limits per item in Cosmos DB. 

| Resource | Default limit |
| --- | --- |
| Maximum size of an item | 2 MB (UTF-8 length of JSON representation) |
| Maximum length of partition key value | 2048 bytes |
| Maximum length of ID value | 1023 bytes |
| Maximum number of properties per item | No practical limit |
| Maximum length of property name | No practical limit |
| Maximum length of property value | No practical limit |
| Maximum length of string property value | No practical limit |
| Maximum length of numeric property value | IEEE754 double-precision 64-bit |
| Maximum level of nesting for embedded objects / arrays | 128 |
| Maximum TTL value |2147483647|

There are no restrictions on the item payloads like number of properties and nesting depth, except for the length restrictions on partition key and ID values, and the overall size restriction of 2 MB. You may have to configure indexing policy for containers with large or complex item structures to reduce RU consumption. See [Modeling items in Cosmos DB](how-to-model-partition-example.md) for a real-world example, and patterns to manage large items.

## Per-request limits

Azure Cosmos DB supports [CRUD and query operations](/rest/api/cosmos-db/) against resources like containers, items, and databases. It also supports [transactional batch requests](/dotnet/api/microsoft.azure.cosmos.transactionalbatch) against multiple items with the same partition key in a container.

| Resource | Default limit |
| --- | --- |
| Maximum execution time for a single operation (like a stored procedure execution or a single query page retrieval)| 5 sec |
| Maximum request size (for example, stored procedure, CRUD)| 2 MB |
| Maximum response size (for example, paginated query) | 4 MB |
| Maximum number of operations in a transactional batch | 100 |

Once an operation like query reaches the execution timeout or response size limit, it returns a page of results and a continuation token to the client to resume execution. There is no practical limit on the duration a single query can run across pages/continuations.

Cosmos DB uses HMAC for authorization. You can use either a primary key, or a [resource tokens](secure-access-to-data.md) for fine-grained access control to resources like containers, partition keys, or items. The following table lists limits for authorization tokens in Cosmos DB.

| Resource | Default limit |
| --- | --- |
| Maximum primary token expiry time | 15 min  |
| Minimum resource token expiry time | 10 min  |
| Maximum resource token expiry time | 24 h by default. You can increase it by [filing an Azure support ticket](create-support-request-quota-increase.md)|
| Maximum clock skew for token authorization| 15 min |

Cosmos DB supports execution of triggers during writes. The service supports a maximum of one pre-trigger and one post-trigger per write operation.

## Metadata request limits

Azure Cosmos DB maintains system metadata for each account. This metadata allows you to enumerate collections, databases, other Azure Cosmos DB resources, and their configurations for free of charge.

| Resource | Default limit |
| --- | --- |
|Maximum collection create rate per minute|    100|
|Maximum Database create rate per minute|    100|
|Maximum provisioned throughput update rate per minute|    5|
|Maximum throughput supported by an account for metadata operations | 240 RU/s |

## Limits for autoscale provisioned throughput

See the [Autoscale](provision-throughput-autoscale.md#autoscale-limits) article and [FAQ](autoscale-faq.yml#lowering-the-max-ru-s) for more detailed explanation of the throughput and storage limits with autoscale.

| Resource | Default limit |
| --- | --- |
| Maximum RU/s the system can scale to |  `Tmax`, the autoscale max RU/s set by user|
| Minimum RU/s the system can scale to | `0.1 * Tmax`|
| Current RU/s the system is scaled to  |  `0.1*Tmax <= T <= Tmax`, based on usage|
| Minimum billable RU/s per hour| `0.1 * Tmax` <br></br>Billing is done on a per-hour basis, where you are billed for the highest RU/s the system scaled to in the hour, or `0.1*Tmax`, whichever is higher. |
| Minimum autoscale max RU/s for a container  |  `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100)` rounded to nearest 1000 RU/s |
| Minimum autoscale max RU/s for a database  |  `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100,  4000 + (MAX(Container count - 25, 0) * 1000))`, rounded to nearest 1000 RU/s. <br></br>Note if your database has more than 25 containers, the system increments the minimum autoscale max RU/s by 1000 RU/s per additional container. For example, if you have 30 containers, the lowest autoscale maximum RU/s you can set is 9000 RU/s (scales between 900 - 9000 RU/s).

## SQL query limits

Cosmos DB supports querying items using [SQL](./sql-query-getting-started.md). The following table describes restrictions in query statements, for example in terms of number of clauses or query length.

| Resource | Default limit |
| --- | --- |
| Maximum length of SQL query| 256 KB |
| Maximum JOINs per query| 5 <sup>*</sup>|
| Maximum UDFs per query| 10 <sup>*</sup>|
| Maximum points per polygon| 4096 |
| Maximum included paths per container| 500 |
| Maximum excluded paths per container| 500 |
| Maximum properties in a composite index| 8 |

<sup>*</sup> You can increase any of these SQL query limits by creating an [Azure Support request](create-support-request-quota-increase.md).

## MongoDB API-specific limits

Cosmos DB supports the MongoDB wire protocol for applications written against MongoDB. You can find the supported commands and protocol versions at [Supported MongoDB features and syntax](mongodb/feature-support-32.md).

The following table lists the limits specific to MongoDB feature support. Other service limits mentioned for the SQL (core) API also apply to the MongoDB API.

| Resource | Default limit |
| --- | --- |
| Maximum MongoDB query memory size (This limitation is only for 3.2 server version) | 40 MB |
| Maximum execution time for MongoDB operations (for 3.2 server version)| 15 seconds|
| Maximum execution time for MongoDB operations(for 3.6 and 4.0 server version)| 60 seconds|
| Maximum level of nesting for embedded objects / arrays on index definitions | 6 |
| Idle connection timeout for server side connection closure* | 30 minutes |

\* We recommend that client applications set the idle connection timeout in the driver settings to 2-3 minutes because the [default timeout for Azure LoadBalancer is 4 minutes](../load-balancer/load-balancer-tcp-idle-timeout.md).  This timeout will ensure that idle connections are not closed by an intermediate load balancer between the client machine and Azure Cosmos DB.

## Try Cosmos DB Free limits

The following table lists the limits for the [Try Azure Cosmos DB for Free](https://azure.microsoft.com/try/cosmosdb/) trial.

| Resource | Default limit |
| --- | --- |
| Duration of the trial | 30 days (a new trial can be requested after expiration) <br> After expiration, the information stored is deleted. |
| Maximum containers per subscription (SQL, Gremlin, Table API) | 1 |
| Maximum containers per subscription (MongoDB API) | 3 |
| Maximum throughput per container | 5000 |
| Maximum throughput per shared-throughput database | 20000 |
| Maximum total storage per account | 10 GB |

Try Cosmos DB supports global distribution in only the Central US, North Europe, and Southeast Asia regions. Azure support tickets can't be created for Try Azure Cosmos DB accounts. However, support is provided for subscribers with existing support plans.

## Azure Cosmos DB free tier account limits

The following table lists the limits for [Azure Cosmos DB free tier accounts.](optimize-dev-test.md#azure-cosmos-db-free-tier)

| Resource | Default limit |
| --- | --- |
| Number of free tier accounts per Azure subscription | 1 |
| Duration of free-tier discount | Lifetime of the account. Must opt-in during account creation. |
| Maximum RU/s for free | 1000 RU/s |
| Maximum storage for free | 25 GB |
| Maximum number of shared throughput databases | 5 |
| Maximum number of containers in a shared throughput database | 25 <br>In free tier accounts, the minimum RU/s for a shared throughput database with up to 25 containers is 400 RU/s. |

In addition to the above, the [Per-account limits](#per-account-limits) also apply to free tier accounts. To learn more, see how to [free tier account](free-tier.md) article.

## Next steps

Read more about Cosmos DB's core concepts [global distribution](distribute-data-globally.md) and [partitioning](partitioning-overview.md) and [provisioned throughput](request-units.md).

Get started with Azure Cosmos DB with one of our quickstarts:

* [Get started with Azure Cosmos DB SQL API](create-sql-api-dotnet.md)
* [Get started with Azure Cosmos DB's API for MongoDB](mongodb/create-mongodb-nodejs.md)
* [Get started with Azure Cosmos DB Cassandra API](cassandra/manage-data-dotnet.md)
* [Get started with Azure Cosmos DB Gremlin API](create-graph-dotnet.md)
* [Get started with Azure Cosmos DB Table API](table/create-table-dotnet.md)
* Trying to do capacity planning for a migration to Azure Cosmos DB? You can use information about your existing database cluster for capacity planning.
    * If all you know is the number of vcores and servers in your existing database cluster, read about [estimating request units using vCores or vCPUs](convert-vcore-to-request-unit.md) 
    * If you know typical request rates for your current database workload, read about [estimating request units using Azure Cosmos DB capacity planner](estimate-ru-with-capacity-planner.md)

> [!div class="nextstepaction"]
> [Try Azure Cosmos DB for free](https://azure.microsoft.com/try/cosmosdb/)
