# SOC Lab - Windows Authentication Monitoring

Azure confirmed creation in East US under the personal resource group and workspace. The workbook uses four queries against the existing Analytics custom table, with no optional storage account and auto-refresh off. The portal required no upgrade or additional service. The pricing lookup was unavailable, so the expected negligible incremental query cost is an estimate and posted charges were not verified.

The fixed November 10 UTC event filter intersects the workbook's 30-day relative query window. The historical view is bounded by data retention and will not continue to display the batch indefinitely. To reproduce, open a blank Sentinel workbook scoped to the appropriate workspace, apply workbook.json through Advanced Editor, and verify the four queries before saving.

Verified totals: 75 events, 61 successful, 14 failed. The 30-minute trend chart showed both series with the same totals. The failure chart showed eight failures for lab-target and one each for six other accounts. The correlation grid showed eight failures for lab-target from 198.51.100.24 to lab-file-01.example.test between 00:30:42.486 and 00:32:54.802 UTC, followed by success at 00:34:00 UTC.

Phase 4 detection evidence remains in phase4/. The Defender results simulation matched the sequence. No scheduled analytics rule, live alert, or incident was created.
