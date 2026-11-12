# Microsoft Sentinel SOC Analyst lab

See [ingested findings](ingested/FINDINGS.md) for Azure ingestion verification and actual results. The files below this note describe the earlier inline-data phase.

# Windows authentication investigation in Azure Logs

## Scope and provenance

This lab used KQL in the personal East US workspace `law-sentinel-soc-personal-eastus`. The data was simulated. All 75 source events were represented inline using six original fields. EventTime is the source JSON's TimeGenerated field renamed for analysis. EventID, Account, IpAddress, Computer, and LogonType retain their original values. The unmodified JSON is included with all fields and the IsSimulated flag.

Source generator: https://github.com/pir8g33k/Sentinel-Data-Generator at commit `1a2c3761176e4e7cfe1b8d22837f3fae42ad92c0`. The generator is MIT-licensed. The JSON SHA-256 is `2c342044f99da3fdfcb1eac4fdb0988ef3d140490ccf117e648e5046b3c737c0`. The events were generated locally with controlled inputs. KQL was executed in Azure Logs and results were captured for verification. The investigation can be reproduced using the steps below.

## Reproduce the investigation

1. Open the personal Log Analytics workspace, select **Logs**, and choose **KQL mode**. Close any welcome or Queries hub prompts.
2. Open one numbered KQL file at a time. Paste the complete contents into a blank editor and click **Run**. Each query is self-contained. The datatable is part of the query and requires no data connector or ingestion step.
3. Compare the **Results** grid to the findings below. The trend query opens the **Chart** tab. The evidence directory contains six cropped screenshots of actual Azure output. Cropping removes unrelated portal chrome.
4. Start with counts to establish completeness. Then identify repeat failures, add the source and destination to narrow the lead, review chronological evidence, and run the correlation to verify the ordering and threshold.

| Query | How it works | Observed Azure result |
|---|---|---|
| 01 Counts | `summarize` groups by event ID and `count()` totals each group. `order by` sorts the rows. | 4624: 61. 4625: 14. |
| 02 Repeated accounts | `where` selects failures, then filters the account totals to two or more. | One account, lab-target, with eight failures. |
| 03 Failures by source | `summarize` groups by account, IP, and destination. `min()` and `max()` show the first and last failure in each group. | Seven groups. The lab-target group had eight failures. Six other groups had one failure each. |
| 04 Timeline | `order by EventTime asc` arranges events from earliest to latest. `project` selects the columns to display. | The grid returned 75 events. |
| 05 Failures then success | `let` names intermediate data. `project` renames event times for each side of the join. `join kind=inner` pairs successes and failures on the same account, IP, and host. A time filter retains only failures in the five minutes before each success. `summarize` counts those failures per success and the final filter requires at least five. | One match. Eight failures for lab-target, 198.51.100.24, lab-file-01.example.test from 00:30:42.486 through 00:32:54.802 UTC, followed by success at 00:34:00 UTC on November 10, 2026. |
| 06 Trend | `bin()` rounds event time down to five-minute buckets. `iff()` labels outcomes. `render timechart` displays the grouped counts. | Azure rendered success and failure series, including a failure bucket of eight around 00:30 UTC. Empty buckets are not explicitly populated with zeroes. |

## Interpret the evidence

**4624** records a successful Windows logon. **4625** records a failed Windows logon. Logon type **3** means network logon. In the simulation, eight type-3 failures preceded one type-3 success for the same account, IP, and host. The tight timing and matching fields make this a candidate password-guessing pattern. The events were deliberately generated, and the output provides no evidence of an actual compromise.

In a real SOC investigation, verify raw event fields and failure status, normalize usernames and timestamps, and establish the account and source IP baseline. Check whether the IP maps to a trusted shared gateway, workstation, scanner, or approved service. Correlate with identity-provider, endpoint, and network telemetry. Look for post-logon behavior, including remote access, process launches, privilege changes, or lateral movement. A corrected password, a retrying service, or shared addressing can produce similar evidence. A five-minute window and a five-failure threshold are lab choices that should be tuned to the environment. For production-scale correlation, bound the time range and use stable event identifiers to distinguish duplicate records and successes at identical timestamps.

## Portfolio description

**Microsoft Sentinel SOC Lab — Windows Authentication Triage with KQL.** A reproducible Azure Logs investigation of 75 simulated Windows authentication events supplied through inline KQL. The analysis establishes a login baseline, groups failures by account and network context, and correlates repeated failures with subsequent success. The package includes the original dataset, executed queries, and Azure result evidence.

## Interview explanation

“I practiced authentication triage in Azure Logs using a simulated dataset. The input was a simulated KQL datatable. I first compared success and failure counts, then grouped failures by account, source IP, and destination. I joined failures to successes on those fields and enforced a five-minute lookback. That returned one simulated candidate with eight failures before a success. In production I'd verify the source and account context and review post-logon activity before escalating.”

## Resources and cleanup

This inline run created no Azure resources, custom tables, DCRs, DCEs, credentials, role assignments, or persistent workbooks. No logs were ingested and no Sentinel analytics rule was tested. The earlier Entra app registration `sentinel-soc-lab-uploader` was unused. It can be reviewed for removal separately. If the personal resource group is no longer needed, review its contents before deleting it. Leave the WGU directory and workspace separate.
