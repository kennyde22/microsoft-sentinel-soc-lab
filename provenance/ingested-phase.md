# Ingested authentication investigation

Azure Logs, personal East US workspace `law-sentinel-soc-personal-eastus`, table `SimulatedWindowsAuth_CL`. The source is the 75-event simulated JSON included in this ZIP. The prior inline investigation is preserved separately. The ingested investigation queries are under `ingested/`, with actual Azure Logs screenshots under `ingested/evidence/`.

## Source reconciliation

The original JSON SHA-256 is `2c342044f99da3fdfcb1eac4fdb0988ef3d140490ccf117e648e5046b3c737c0`. The JSON and the Azure result each produced 75 distinct full-record hashes. The sorted hashes, including multiplicity, produced identical SHA-256 digests `487a54c8d83ea3d8845f189d31b0b5058f0794057fe9f4414f37421790a0d6b5`. The comparison uses all 13 source fields in the order specified by the verification query. It normalizes UTC timestamps to six fractional digits and excludes Azure metadata. The result covers the November 10 UTC batch window. No duplicate, missing, extra, or mismatched rows were found in that window.

## Findings

- Azure returned 61 successful logons (4624) and 14 failed logons (4625).
- One account had repeated failures. Eight failures for `lab-target` from `198.51.100.24` to `lab-file-01.example.test` preceded a success on the same account, source, and host within 15 minutes.
- The failures ran from 00:30:42.486 to 00:32:54.802 UTC on November 10, 2026. The success occurred at 00:34:00 UTC. Each of the six other failed account, source, and host combinations had one failure.
- The 00:30 UTC half-hour bucket contained 10 failures and 14 successes. The other three buckets contained three, zero, and one failure, respectively.
- This simulated sequence illustrates a brute-force triage signal. In real telemetry, mistyped passwords or stale credentials could produce similar sequences. Verify logon type, endpoint activity, identity context, and follow-on actions before drawing conclusions about compromise.
- The correlation query applies a five-failure minimum and a 15-minute lookback per successful event. It was executed in Azure Logs. No persistent workbook or scheduled analytics rule was deployed or tested in this phase.

## Access cleanup

Azure Portal confirmed removal of the uploader's Monitoring Metrics Publisher assignment at the DCR scope. Microsoft Entra then confirmed removal of the expired certificate `sentinel-one-time-20261110T043909` by showing **Certificates (0)** and **No certificates have been added for this application**. The application registration and lab resources were preserved. Posted charges were outside this verification.

## Workbook and analytics rule plan

After separate authorization, build the workbook from the verified table queries for successes and failures, repeated accounts, source and host breakdowns, and the failure-to-success sequence. Use a fixed time range covering the simulated batch. For the analytics rule, start with the validated five-failure threshold and 15-minute correlation window. Check whether historical simulation can test the retained batch without re-ingesting events, and record the difference between a query match and a generated alert. Review schedule, lookback, entity mapping, and costs before enabling any rule.
