# Demos

## Preparing

PIM for Tenant Root as Contributor
PIM for needed Azure Lighthouse Groups

## Demo 1

### Scoped PIM to Windows Admin Center Access

Target to Barn

Leave to activate

### Arc + SSH + SSH Agent for keys

Get *az ssh* command from portal

```bash:
az login
az ssh arc --subscription "X" --resource-group "rg-hybridmanagement-prod-001" --name "Barn" --local-user administrator
```

### WAC

Scheduled tasks
Files
Remote Desktop

## Demo 2

Play around Azure Automation Account
  
* Runbooks
  * Output
  * Schedules
* Variables and secrets
* Identities

Filter to error logs of Azure Automation Account

```kql:
// Runbook completed successfully with errors 
// List all jobs that completed with errors. 
// To create an alert for this query, click '+ New alert rule'
AzureDiagnostics 
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and StreamType_s == "Error" 
| where RunbookName_s == 'Get-FirstLinesOfFile'
| project TimeGenerated , RunbookName_s , StreamType_s , _ResourceId , ResultDescription , JobId_g 

```

Create alert
Discuss targets

## Demo 3

Discuss about PIM and Azure Lighthouse permissions

```kql:
// Resource Explorer
// Get All servers from ALL customers
Resources
| where (type == 'microsoft.compute/virtualmachines') or (type == 'microsoft.hybridcompute/machines') 


```

Walk quickly the script

Install Patch remover

Check status of CSE
