---
title: "KQL Query Patterns"
description: "Kusto Query Language patterns for Azure Monitor, Log Analytics, and Application Insights."
category: "KQL"
---

## Azure Monitor / Log Analytics

### Container Insights (AKS)

```kql
// Pod restart count in last 24h
KubePodInventory
| where TimeGenerated > ago(24h)
| where RestartCount > 0
| summarize TotalRestarts = sum(RestartCount) by Name, Namespace
| order by TotalRestarts desc
```

```kql
// OOMKilled pods
ContainerLog
| where TimeGenerated > ago(24h)
| where LogEntry contains "OOMKilled"
| project TimeGenerated, ContainerID, LogEntry
```

```kql
// Node CPU usage over time
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "K8SNode" and CounterName == "cpuUsageNanoCores"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart
```

### Application Insights

```kql
// Failed requests grouped by operation
requests
| where timestamp > ago(1h)
| where success == false
| summarize FailedCount = count() by operation_Name, resultCode
| order by FailedCount desc
```

```kql
// Slow requests (>2 seconds)
requests
| where timestamp > ago(24h)
| where duration > 2000
| project timestamp, name, duration, resultCode, url
| order by duration desc
| take 50
```

```kql
// Dependency failures (SQL, HTTP, etc.)
dependencies
| where timestamp > ago(1h)
| where success == false
| summarize FailCount = count() by type, target, resultCode
| order by FailCount desc
```

```kql
// Exception breakdown
exceptions
| where timestamp > ago(24h)
| summarize Count = count() by type, outerMessage
| order by Count desc
| take 20
```

### Resource Diagnostics

```kql
// Azure Activity Log - failed operations
AzureActivity
| where TimeGenerated > ago(7d)
| where ActivityStatusValue == "Failed"
| project TimeGenerated, OperationNameValue, Caller, ResourceGroup, Properties
| order by TimeGenerated desc
```

```kql
// Cosmos DB RU consumption
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DOCUMENTDB"
| where TimeGenerated > ago(1h)
| summarize TotalRU = sum(todouble(requestCharge_s)) by bin(TimeGenerated, 5m), databaseName_s, collectionName_s
| render timechart
```

## Common Patterns

### Time Bucketing

```kql
// Events per 5-minute intervals
MyTable
| where TimeGenerated > ago(6h)
| summarize Count = count() by bin(TimeGenerated, 5m)
| render timechart
```

### Percentiles

```kql
// Response time percentiles
requests
| where timestamp > ago(1h)
| summarize
    p50 = percentile(duration, 50),
    p90 = percentile(duration, 90),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99)
  by bin(timestamp, 5m)
| render timechart
```

### Dynamic JSON Parsing

```kql
// Parse JSON from a string column
MyTable
| extend parsed = parse_json(DynamicColumn)
| extend userId = tostring(parsed.user.id)
| extend action = tostring(parsed.action)
```

### Alerting Threshold Pattern

```kql
// Error rate > 5% in the last 15 min
requests
| where timestamp > ago(15m)
| summarize Total = count(), Failed = countif(success == false)
| extend ErrorRate = (Failed * 100.0) / Total
| where ErrorRate > 5
```
