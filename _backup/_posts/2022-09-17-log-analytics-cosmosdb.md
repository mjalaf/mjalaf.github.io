---
title: Azure Logs Analytics for CosmosDB
---

Cosmos DB allows to enable and store logs in Logs Analytics, this is helpful for troubleshooting and performance analysis. 

## Enabling Log Analytics on CosmosDB

1-Access to Cosmos DB Account and in the Left panel go to Monitoring and then Diagnostic Settings

![Cosmos Analytics](/assets/img/CosmosLogA/1.jpg)

2- A message indicates that if we want to enable "Full Text Query", lets click on Not Now

![Cosmos Analytics](/assets/img/CosmosLogA/2.jpg)

3- If there is not Diagnostics Settings enabled, click on "Add Diagnostic Settings"

![Cosmos Analytics](/assets/img/CosmosLogA/1a.jpg)

4- Select the Logs to store on Logs Analytics
![Cosmos Analytics](/assets/img/CosmosLogA/1b.jpg)

 This is the first place where we need to be very careful on the logs that we want to store, I worked with many customers optimizing logs and this is one of the first places where I used to review.
 
 In my scenario, I am using SQL API, so I don’t need to select the options for (Casandra, Gremlin, Table, Mongo), if you are using any of those APIs, just select that one. 
 

| Category | Description | 
| -------- | -------- | 
| DataPlaneRequest     | The DataPlaneRequests table captures every data plane operation for the Cosmos DB account. Data Plane requests are operations executed to create, update, delete or retrieve data within the account. | 
| QueryRuntimeStatistics | This table details query operations executed against a SQL API account. By default, the query text and its parameters are obfuscated to avoid logging PII data with full text query logging available by request.   | 
| PartitionKeyStatistics |  This table provides outlier logical partition keys that have consumed more storage space than others, which can be valuable in debugging storage skews. | 
| PartitionKeyRUConsumption |This table details the RU (Request Unit) consumption for logical partition keys in each region, within each of their physical partitions. This data can be used to identify hot partitions from a request volume perspective.  | 
| ControlPlaneRequest | This table details all control plane operations executed on the account, which include modifications to the regional failover policy, indexing policy, IAM role assignments, backup/restore policies, VNet and firewall rules, private links as well as updates and deletes of the account.  | 

**Important:** This is an area where we need to be cautious, while keeping those logs is very helpful, they will generate some cost on Logs Analytics Workspace (it will depend on the Database usage) but it is important to review the Logs consumption for CosmosDB on Log analytics

Here is a query that can be helpful to verify the volume of logs stored.

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DOCUMENTDB"
|summarize count() by Category
| order by count_ desc
```
![Cosmos Analytics](/assets/img/CosmosLogA/5.jpg)

In my example, you can see that on DataPlanRequest were stored more than 112 Millon logs in 24 hours

## Accessing to the Logs in Cosmos DB
To be able to access the Logs, there is an option under the minoring section on the left panel called Logs
![Cosmos Analytics](/assets/img/CosmosLogA/1.jpg)

Once accessing Logs, Log Analytics will provide some pre-defined queries

We can observe a separation between "Alerts" and "Diagnostics" 

For "Alerts" Queries, Cosmos DB present the following options

![Cosmos Analytics](/assets/img/CosmosLogA/2-a.jpg)

For "Diagnostic" Queries, Cosmos DB present the following options

![Cosmos Analytics](/assets/img/CosmosLogA/2-b.jpg)

Let’s load the first query presented by cosmos db
![Cosmos Analytics](/assets/img/CosmosLogA/2-c.jpg)

```
// Top operations by consumed Request Units (RUs) in last 24 hours 
// Identify top operations on Cosmos resources by count and consumed RU per operation. 
// To create an alert for this query, click '+ New alert rule'
AzureDiagnostics
| where TimeGenerated >= ago(24h)
| where Category == "DataPlaneRequests"
| summarize numberOfOperations = count(), totalConsumedRU = sum(todouble(requestCharge_s)) by databaseName_s, collectionName_s, OperationName, requestResourceType_s, requestResourceId_s, _ResourceId
| extend averageRUPerOperation = totalConsumedRU / numberOfOperations 
| order by numberOfOperations
```
In this query we can see that the category is "DataPlaneRequests"

The query syntax  is KQL - Kusto Query Language

On the left panel, we can see the Logs available for CosmosDB

![Cosmos Analytics](/assets/img/CosmosLogA/6.jpg)

## Enabling   Full Text Query
By default, Cosmos DB doesn’t  enable the full text query, this option helps to improve queries. 

Default Logs (without full text query)
```
Select c.p1, c.p2, c.p3 form c where c.p1="123"
```

Enabeling Full Text Query
```
Select c.id, c.firstname, c.lastname form c where c.id="123"
```

To enable Full text Query
1- Click on Diagnostic Settings one more time 
2- A message indicates  that if we want to enable "Full Text Query", let’s click on Go to Feature and Enable 

![Cosmos Analytics](/assets/img/CosmosLogA/2.jpg)

3- Click on Diagnostics full-text query and then Enable it

![Cosmos Analytics](/assets/img/CosmosLogA/7.jpg)

In future post I will explain some troubleshooting techniques

**References:**
[Log Analytics Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
[Log Analytics Tutural](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-tutorial)
[Kusto query language](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
