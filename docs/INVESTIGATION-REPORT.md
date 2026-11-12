# SOC investigation report · Repeated Windows logon failures followed by success

| Item | Assessment |
|---|---|
| Case | Simulated authentication triage |
| Evidence window | November 10, 2026 UTC |
| Source | 75 simulated events indexed in `SimulatedWindowsAuth_CL` |
| Disposition | Expected simulation pattern verified. No real incident or compromise was established. |

## Executive summary

A network logon succeeded after eight failures for the same simulated account, source IP, and destination host. KQL correlation identified one matching success. This sequence would merit triage in a production SOC because a successful logon after repeated failures can follow password guessing. The same pattern can also result from legitimate retries or stale credentials.

## Evidence and methodology

The batch was reconciled before analysis. Both the source and Azure yielded 75 unique hashes across the same 13 event fields, with UTC timestamps normalized to microsecond precision. The sorted row-hash digest matched on both sides. Azure-generated metadata was excluded. The matching counts and digest support zero missing, extra, duplicate, or mismatched rows within the tested batch window. See the [reconciliation KQL](../kql/ingested/07-source-verification.kql) and [Azure result](../evidence/ingested/07-source-verification.jpg).

The six investigations established event counts, identified repeated failures by account, grouped failures by account/IP/host, ordered the event timeline, correlated failures to a later success, and charted counts in 30-minute buckets. The correlation used an inner join on account, IP, and host and then kept only failures in the 15 minutes before each success. The exploratory query required five failures. The workbook and Defender simulation used eight.

## Observed findings

| Signal | Observation |
|---|---|
| Dataset totals | 61 successes (4624), 14 failures (4625) |
| Failure distribution | Eight for the target combination. Six other combinations had one each. |
| Account, IP, and host | `lab-target`, `198.51.100.24`, `lab-file-01.example.test` |
| First and last failure | 00:30:42.486 UTC and 00:32:54.802 UTC |
| Subsequent successful logon | 00:34:00 UTC, 65.198 seconds after the last failure |
| Logon type | Type 3, network logon, for the correlated sequence |
| Peak failure bucket | 00:30–00:59 UTC, 10 failures and 14 successes across all accounts |

The saved workbook displayed the same counts and sequence. Defender's built-in simulation evaluated the eight-failure query with a five-hour interval and 72-hour lookback. Its 50-evaluation chart showed a matching result at the 04:00 UTC evaluation. This is evidence of a simulated evaluation. There was no deployed scheduled rule, generated alert, or incident. [Workbook evidence](../evidence/workbook/13-workbook-correlation.jpg) · [Simulation evidence](../evidence/detection/11-rule-simulation.jpg)

## Analyst recommendations for an equivalent production signal

1. Confirm account ownership, expected source and destination, logon type, and failure codes. Establish whether the source represents a shared gateway, service, or individual endpoint.
2. Compare the sequence with account and host baselines and check for legitimate explanations such as a corrected password or a service using stale credentials.
3. Correlate endpoint, network, and identity telemetry. Look for unexpected post-logon processes, remote access, privilege changes, or access to sensitive data. Escalate using the organization's incident procedures when corroborating evidence supports it.
4. Tune failure thresholds and time windows against baseline activity. Use stable event identifiers where available, and test overlapping scheduling windows for duplicate alerts before operational rollout.

No containment was required for the simulated case.
