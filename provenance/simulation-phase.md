# Phase 4 detection simulation

Microsoft Defender evaluated the unsaved scheduled rule against the existing personal workspace. The query correlates eight or more 4625 failures within 15 minutes before a 4624 success for the same account, source IP, and destination host. It returns the successful event timestamp as TimeGenerated.

Configuration tested: every 5 hours, 72-hour lookback, query result threshold greater than zero. The chart displayed 50 historical evaluations, with a peak of one matching row at the November 10, 2026 04:00 UTC evaluation. The underlying successful logon occurred at 00:34 UTC that day. An overlapping lookback can match the same sequence on multiple evaluations.

This was a results simulation in an unsaved wizard. No scheduled rule, live alert, incident, or workbook was created. No events were replayed. The trial expiration previously reported was December 10, 2026 at 4:59:59 p.m. Phoenix time. Current trial coverage and incremental costs for workbook queries and scheduled rule evaluation were not independently verified. The documentation lookup failed.
