# 10-minute design walkthrough

1. **Assumptions and open questions — 1 minute.** Lead with the 10 edits/second estimate, the timestamp/ordering assumption, the dedicated integration identity, and the changed-since recovery question.
2. **Systems and boundaries — 1 minute.** Show Salesforce webhooks entering the portal and the portal outbox worker calling Salesforce. Point out that the 100 requests/second limit covers calls, retries, and reconciliation.
3. **Build order and data — 2 minutes.** Explain why identity/version/history come before workers. Show inbox, per-field versions, append-only history, outbox, and cursor.
4. **Inbound and outbound paths — 2 minutes.** Walk through durable webhook acceptance and the portal save/outbox transaction. Emphasize process restart safety and field-level merging.
5. **Correctness rules — 2 minutes.** Use one duplicate, one old event, two edits to different fields, and an integration-user echo to explain deduplication, ordering, independent fields, and loop prevention.
6. **Outage math and monitoring — 2 minutes.** Show `10 × 21,600 = 216,000`, `80 − 10 = 70`, and `216,000 ÷ 70 ≈ 51.4 minutes`. State that reconciliation and retries consume the same cap and the estimate must be recalculated if actual API behavior differs.

Keep the questions visible. If asked about an unconfirmed Salesforce capability, say which assumption the design uses and what changes if it is unavailable.
