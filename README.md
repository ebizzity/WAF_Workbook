# Azure Application Gateway WAF and Traffic Workbook

An Azure Monitor Workbook template for reviewing Application Gateway traffic, WAF blocks, and request-derived gateway performance. It includes interactive selectors and four tabs: Overview, All traffic, Blocked traffic, and Gateway metrics.

## Files

- `azure-application-gateway-waf-workbook.json` — workbook template to import into Azure Monitor.

## Requirements

- An Azure Monitor Log Analytics workspace that receives Application Gateway diagnostic logs in **resource-specific** mode.
- The workspace must contain the `AGWAccessLogs` and `AGWFirewallLogs` tables.
- Enable the Application Gateway access and firewall diagnostic categories and route them to the workspace.
- The signed-in user needs permission to see subscriptions and resources in Azure Resource Graph, and access to query the selected Log Analytics workspace.

This template does not query the legacy `AzureDiagnostics` table. If your logs are configured for the legacy destination, switch to resource-specific tables or adapt the queries and column names.

## Import or update

1. In the Azure portal, open **Monitor** > **Workbooks** and create a workbook, or open the workbook you want to update.
2. Open **Advanced Editor**.
3. Replace the editor contents with `azure-application-gateway-waf-workbook.json`, then apply/save the changes.
4. Return to read mode and choose a subscription, workspace, Application Gateway selection, and time range.

For a deployed workbook, save it to the desired Azure subscription/resource group and set the appropriate access controls before sharing it.

## Workbook selectors

- **TimeRange** — time range for the log queries; defaults to the last 24 hours.
- **Subscription** — multi-select subscription scope for discovering workspaces and Application Gateways. All available subscriptions are selected by default.
- **Workspace** — Log Analytics workspace used by the workbook's log queries. Select the workspace receiving the gateway diagnostics.
- **Application Gateway** — multi-select gateway filter. All discovered gateways are selected by default.

Changing the subscription selector scopes the workspace and gateway lists. The selected gateway resource IDs filter every log query.

## Tabs

### Overview

- Summary tiles for request volume, gateway count, HTTP 4xx/5xx responses, end-to-end latency, WAF activity, and blocked transactions.
- Traffic disposition pie chart comparing blocked transactions with not-blocked access-log requests.
- HTTP response-code family distribution.
- Top blocking WAF rules.
- Request and WAF block event trends.

The traffic disposition chart is an estimate: it compares access-log rows with distinct blocked WAF transaction IDs for the selected time range and gateway scope. The two sources can differ in coverage or granularity; the chart clamps blocked counts to total access-log requests. Interpret it as an indicator, not an exact billing or security ratio.

### All traffic

- Request volume over time by HTTP response class.
- HTTP response distribution.
- Request volume by listener and backend pool.
- End-to-end and WAF evaluation latency trends.
- Per-gateway request/error/latency summary and a recent-request detail grid.

`TimeTaken` represents end-to-end request time and may include client network time, gateway processing, and backend response time. It is not gateway-only processing latency.

### Blocked traffic

- Blocking-event counts by rule and policy scope.
- Blocked transaction details, correlated with `Matched` events for the same gateway and transaction ID where available.
- Source IP summary, including block-event counts, distinct transactions, rule IDs, and paths.

The detailed grid includes the action, rule ID, rule-set version, policy scope, `Message`, `DetailedMessage`, `DetailedData`, transaction ID, and related access-log/backend fields. `DetailedData` can expose request content; treat it as sensitive and limit workbook access accordingly.

### Gateway metrics

Request-log panels for end-to-end and WAF evaluation latency, backend response codes, backend latency, connection time, and failed requests by pool.

These are values derived from `AGWAccessLogs`, not all Azure Monitor platform metrics. For native metrics such as healthy/unhealthy host count, capacity units, CPU utilization, and throughput, use the Application Gateway resource's **Metrics** blade. Not all native metrics are exported to Log Analytics.

## Interpretation notes

- WAF log actions have different meanings. A blocking action represents a blocked request; `Matched` can be a contributing rule event and is not by itself proof that the request was blocked. Detection-mode events are logged but passed through.
- One blocked transaction can produce multiple WAF rows when multiple rules match. Rule-event counts and distinct transaction counts are therefore not interchangeable.
- Blocked-event queries include `Blocked`, `Detected and Blocked`, and `JSChallengeBlock`. Review the underlying `Action` values and WAF mode when investigating individual records.
- Query time ranges use the workbook time picker. `TimeGenerated` is stored in UTC.
- Empty panels can indicate no matching traffic/events, an incorrect workspace or gateway selection, missing diagnostic categories, or logs that have not arrived yet.

## References

- [Azure Application Gateway diagnostics](https://learn.microsoft.com/azure/application-gateway/application-gateway-diagnostics)
- [AGWAccessLogs table reference](https://learn.microsoft.com/azure/azure-monitor/reference/tables/agwaccesslogs)
- [AGWFirewallLogs table reference](https://learn.microsoft.com/azure/azure-monitor/reference/tables/agwfirewalllogs)
- [Azure Workbooks resource parameters](https://learn.microsoft.com/azure/azure-monitor/visualize/workbooks-resources)
