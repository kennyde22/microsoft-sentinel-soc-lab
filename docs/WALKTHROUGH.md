# Reproduce the Windows authentication investigation

## 1. Follow the data

The source JSON contains simulated Windows Security events. Event 4624 records a successful logon. Event 4625 records a failed logon. Azure Monitor's Logs Ingestion API accepted the batch through the direct DCR endpoint. The DCR defines the input schema and routes records into the custom Analytics table. The workspace stores and queries the data. Sentinel uses that same workspace for investigation and monitoring.

The temporary Entra uploader used a certificate to authenticate and the Monitoring Metrics Publisher role at the DCR scope to write to its stream. After verifying ingestion, both the temporary credential and the role assignment were removed. The [sanitized configuration](../config/LAB-CONFIGURATION.md) records the boundaries of the setup.

## 2. Prepare an authorized lab

Start with a Sentinel-enabled Log Analytics workspace and a matching custom Analytics table. Create a direct DCR in the workspace's region with a stream declaration matching the source fields and one data flow to the custom table. Verify that the endpoint belongs to the DCR and the destination is the intended workspace. If reproducing the upload in your own environment, use your own temporary authorized identity, grant write access only at the DCR scope, and submit the JSON array once. Keep credentials outside the repository. Verify the batch before retrying and remove temporary access after verification.

For this historical batch, you can review the source without any ingestion using the inline KQL files. A fresh test batch needs timestamps within your environment's accepted ingestion window. The saved reconciliation digest applies to the original JSON and will change if timestamps are modified. The original batch should not be replayed into the existing project workspace.

## 3. Verify the indexed data

Open the workspace's **Logs** blade, select **KQL mode**, and run [07-source-verification.kql](../kql/ingested/07-source-verification.kql). Use a query time range covering November 10, 2026 UTC. Expect 75 total, 75 unique, 61 successes, 14 failures, and `ExactMatch=true`. The query packs the original fields into an ordered array, hashes each row, sorts those hashes, and compares the resulting digest to the source digest. Matching counts alone would miss changed field values.

## 4. Run the six investigations

| Query | What to do and why |
|---|---|
| 01 · Counts | Use `summarize count()` by EventID to establish the success and failure baseline. |
| 02 · Repeated accounts | Use `where` to retain 4625 events, group by account, and keep accounts with two or more failures. The only repeat was lab-target. |
| 03 · Failures by source | Group by account, IP, and destination to distinguish a concentrated pattern from scattered failures. Expect seven combinations. |
| 04 · Timeline | Use `project` to select the evidence columns and `order by` to sort chronologically. Examine failures and successes around 00:30–00:34 UTC. |
| 05 · Failures then success | Use `let` to name the input and `join kind=inner` to match account, IP, and host. Compare timestamps so only failures in the preceding 15 minutes count. The exploratory minimum is five. The result contains eight. |
| 06 · Trend | Use `bin(TimeGenerated, 30m)` to bucket timestamps and `countif()` to total both outcomes per bucket. The 00:30 UTC bucket contained 10 failures overall. |

Run each complete file in [kql/ingested](../kql/ingested). The earlier [inline queries](../kql/inline) are separate exercises. They use a five-minute correlation window and five-minute trend buckets.

## 5. Inspect the workbook

Open **Workbooks** for the lab workspace. In a reproduction workspace, use a blank workbook, open **Advanced Editor**, and apply [workbook.json](../workbook/workbook.json). Verify totals, the trend, account breakdown, and the correlation row before saving. The correlation panel requires at least eight failures in 15 minutes. The timestamps in that panel are UTC. Auto-refresh was off and no optional storage account was configured. The fixed event date and relative query window mean screenshots may be the only way to review the original visualization after retention or query-window expiry.

## 6. Inspect the Defender simulation

In the Sentinel analytics wizard, the [detection KQL](../kql/detection/11-scheduled-rule.kql) returned a matching success under **Test with current data**. The test used an eight-failure threshold, 15-minute correlation, five-hour evaluation interval, 72-hour lookback, and a query result threshold greater than zero. The chart showed a match at the November 10 04:00 UTC evaluation. The time shown for an evaluation is separate from the underlying event timestamp. The wizard was left unsaved. A live test would need separately approved deployment and evidence from an actual rule run.

## Glossary

- **SIEM** · Security information and event management, used to collect and analyze security telemetry.
- **DCR** · Data Collection Rule, which defines how Azure receives and routes logs.
- **KQL** · Kusto Query Language, used here to count, filter, group, and correlate events.
- **Lookback** · The amount of historical data examined during one rule evaluation.
- **Alert / incident** · An alert is a detection output. An incident groups related evidence for investigation.
