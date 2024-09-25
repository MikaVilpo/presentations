# Azure and Friends #23

Held 25.9.2024 at Elisa, Helsinki, Finland

## Azure ARGh! What can you do with Azure Resource Graph

Session delivered 25.9.2024.

## Demos

### Demo 1

Super fast VM Fetching
PowerShell does not have multitenant support in this version:

```PowerShell:
Measure-Command {
[System.Collections.Generic.List[PSObject]]$VMs = @()
$subs = Get-AzSubscription
foreach ($s in $subs) {
  Set-AzContext -SubscriptionObject $s | Out-Null
  $vm = Get-AzVM | Where-Object {$_.tags['ElisaUpdates'] -eq 'NOT SET'}
  $VMs.Add($vm)
 }
 $VMs | Format-Table
 }
```

```kql:
resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| where tags['ElisaUpdates'] =~ "NOT SET"
```

### Demo 2

```kql:
// Resource Types
resources
| summarize
    UniqueResourceTypes = dcount(type),
    VirtualMachineCount = countif(type == "microsoft.compute/virtualmachines"),
    SQLServerCount = countif(type in ("microsoft.sql/servers","microsoft.sql/managedinstances","microsoft.sqlvirtualmachine/sqlvirtualmachines"))
by resourceGroup, subscriptionId
| join kind=inner (
    resourcecontainers
    | where type == 'microsoft.resources/subscriptions'
    | project subscriptionId, subscriptionName = name)
    on subscriptionId
| project subscriptionName, subscriptionId, resourceGroup, UniqueResourceTypes, VirtualMachineCount, SQLServerCount


// Other Sources

policyresources
| where kind =~ 'policystates' and properties.complianceState != 'Compliant'
| project Scope=properties.policyAssignmentScope, Policy=properties.policyAssignmentName, State=properties.complianceState

securityresources
| summarize count() by type

securityresources
| where type == 'microsoft.security/assessments'
| where properties.status.code != 'Healthy'
| project tenantId, subscriptionId, resourceGroup, displayName=tostring(properties.displayName)
| summarize count() by displayName
| order by ['count_'] desc

maintenanceresources
| limit 50
```

### DEMO 3

```kql:
// Change

// Resource container
resourcecontainers
| summarize count() by type

resourcecontainerchanges
| limit 1

resourcecontainerchanges
| summarize count=count() by subscriptionId, resourceGroup, tostring(properties.targetResourceType), tostring(properties.changeAttributes.clientType), tostring(properties.changeAttributes.changedByType), tostring(properties.changeAttributes.changedBy)

// Resource with most changes
resourcechanges
| summarize count=count() by subscriptionId, resourceGroup, tostring(properties.targetResourceType), tostring(properties.changeAttributes.clientType), tostring(properties.changeAttributes.changedByType), tostring(properties.changeAttributes.changedBy)
| order by ['count'] desc

resourcechanges
| summarize count=count() by tostring(properties.changeAttributes.changedBy)
| order by ['count'] desc

// All changes during last 24h
resourcechanges
| extend changeTime = todatetime(properties.changeAttributes.timestamp), targetResourceId = tostring(properties.targetResourceId),
changeType = tostring(properties.changeType), correlationId = properties.changeAttributes.correlationId, 
changedProperties = properties.changes, changeCount = properties.changeAttributes.changesCount
| where changeTime > ago(1d)
| order by changeTime desc
| project changeTime, targetResourceId, changeType, correlationId, changeCount, changedProperties

// VM Size
resourcechanges
| extend vmSize = properties.changes["properties.hardwareProfile.vmSize"], changeTime = todatetime(properties.changeAttributes.timestamp), targetResourceId = tostring(properties.targetResourceId), changeType = tostring(properties.changeType) 
| where isnotempty(vmSize) 
| order by changeTime desc 
| project changeTime, targetResourceId, changeType, properties.changes, previousSize = vmSize.previousValue, newSize = vmSize.newValue

// Delete
resourcechanges
| extend changeTime = todatetime(properties.changeAttributes.timestamp), targetResourceId = tostring(properties.targetResourceId),
  changeType = tostring(properties.changeType), correlationId = properties.changeAttributes.correlationId
| where changeType == "Delete"
| order by changeTime desc
| project changeTime, resourceGroup, targetResourceId, changeType, correlationId

quotaresourcechanges
| limit 10

// Health changes
healthresourcechanges
| extend state = properties.changes["properties.availabilityState"], changeTime = todatetime(properties.changeAttributes.timestamp), targetResourceId = tostring(properties.targetResourceId), changeType = tostring(properties.changeType) 
| where isnotempty(state) 
| order by changeTime desc 
| project changeTime, targetResourceId, changeType, properties.changes, prevState=state.previousValue, newState=state.newValue
```

### DEMO 4

```kql:
// Caveats

resources
| project Owner = tags.Owner
| where isnotnull(Owner)
| count 

resources
| project Owner = tags.owner
| where isnotnull(Owner)
| count
```