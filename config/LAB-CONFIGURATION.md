# Sanitized lab configuration

| Component | Observed configuration |
|---|---|
| Azure scope | Personal subscription, existing East US resource group and Log Analytics workspace |
| Table | `SimulatedWindowsAuth_CL`, Analytics plan, 30-day Analytics and total retention at deployment |
| Direct DCR | East US, built-in ingestion endpoint, single destination to the personal workspace and custom table |
| Authorization | Existing Entra app with a temporary certificate and Monitoring Metrics Publisher scoped to the lab DCR. Both access mechanisms were removed after verification. |
| Source | One JSON batch of 75 simulated Windows logons. The batch was submitted once. |
| Workbook | SOC Lab – Windows Authentication Monitoring, existing workspace scope, no optional storage, auto-refresh off |
| Rule | Unsaved Defender simulation only. See the detection KQL and phase notes. |

These notes describe the observed configuration. Resource IDs, endpoint URLs, app identifiers, and credentials are omitted. Use the [original JSON](../data/simulated-windows-auth.json) to inspect field names and sample values. The 13 fields are TimeGenerated (datetime), Computer (string), EventID (int), Activity (string), Account (string), AccountType (string), LogonType (int), IpAddress (string), WorkstationName (string), Status (string), SubStatus (string), IsSimulated (bool), and Scenario (string).

The documented personal trial end was December 10, 2026 at 4:59:59 p.m. Phoenix time. Posted charges and current trial coverage were not independently confirmed. For a reproduction lab, check your billing scope and retention before creating resources. Remove temporary access after upload. When finished with a dedicated environment, review the workbook, DCR, table, Sentinel workspace, and app registration before removing resources you no longer need.
