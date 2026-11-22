# Assumptions and open questions

The design below is concrete enough to review, but several decisions depend on Salesforce capabilities the brief does not specify. Confirm these before implementation.

## Working assumptions

| Topic | Working assumption | If the assumption is false |
| --- | --- | --- |
| Peak edit rate | 10 record edits/second sustained during an outage. Each edit changes one record and needs one Salesforce API request. | Recalculate the backlog and drain time using measured peak rate and actual requests per edit. Bulk APIs may change the request count, but are not assumed here. |
| Webhook payload | Webhooks carry changed fields, as in the example. | If they carry full snapshots, only fields with reliable per-field versions can safely be merged; otherwise a stale snapshot can overwrite a newer field. |
| Salesforce event identity | `eventId` is stable across redelivery; `occurredAt` and `changedBy` are supplied and meaningful. | If event IDs are not stable, define a deterministic event key; if timestamps or actors are not trustworthy, use Salesforce record versions or another confirmed ordering/origin signal. |
| Portal edit identity | The portal backend assigns a UTC timestamp and a monotonically increasing per-record sequence in the same database transaction as the edit. The authenticated user is the actor. | If writes can come from multiple writers or regions, use a centrally ordered version or database change sequence. |
| Record identity and creates | Each new record is distinct until the integration has a stable cross-system key. Never merge records based only on name, phone, or email. Salesforce supports an idempotent upsert or lookup by a stable portal key for portal-originated creates. | If both systems independently create the same real-world entity, duplicates remain until the client defines a matching key/rule. If create retries cannot be made idempotent, ambiguous timeouts require lookup/reconciliation before retry. |
| Contact/customer creation | A contact's customer reference can be resolved through the stored cross-system ID mapping; if the customer mapping is not ready, hold the contact work item until it is. | Confirm whether Salesforce can create a contact only after its customer exists, and what to do if the referenced customer was not synced. |
| Cross-system conflict order | UTC timestamps from Salesforce and the portal are close enough to compare, with a deterministic source/event-ID tie-break. | If clock skew cannot be bounded, timestamps alone cannot establish real-world order. Agree on a version/precedence contract or surface conflicts for resolution. |
| Loop signal | Salesforce webhooks identify the dedicated integration user in `changedBy`, and outbound calls use that identity. | If the actor is not echoed, Salesforce must provide a reliable source marker or correlation field. Without either signal, an echo cannot always be distinguished from a human edit of the same value. |
| Missed webhook recovery | Salesforce provides a paginated, complete change feed/audit source for the outage window, including every edit, actor, timestamp, and stable event/version marker. | A latest-record snapshot can restore current values but cannot reconstruct intermediate edits or actors. The history requirement cannot be met for missed events without a complete change feed or retained audit source; treat this as a blocker, not as a successful recovery. |
| API request cost | One outbound request per changed record; retries count as requests. Reconciliation calls also count against the 100 requests/second cap. | Recompute capacity and outage catch-up using observed API calls per edit and reconciliation page size. |
| Durable storage | The portal database can atomically persist a local edit, its field versions/history, and an outbound work item. A durable queue or database-backed outbox survives process restart. | If queue and record writes cannot share a transaction, use a transactional outbox or equivalent recovery mechanism; never rely on in-memory queue state. |

## Questions to ask before locking the design

1. Does Salesforce expose a stable event ID, an authoritative record version, and a timestamp that reflects the edit rather than delivery time?
2. Does a webhook include only changed fields or a full record? Can it include a field-level version for each field?
3. Can the integration use a dedicated Salesforce identity, and will webhook `changedBy` identify it? If not, is there a supported origin/correlation field?
4. Can Salesforce replay every change (not just return each record's latest snapshot) since a cursor after a six-hour webhook gap, including intermediate edits and actors? What are the retention, pagination, and request costs?
5. Are Salesforce record reads and writes individually rate-limited at 100 requests/second, and are there other tenant-wide or daily limits?
6. What stable cross-system key can identify records created on either side? How should the system handle two independent creates for the same real-world entity?
7. Can Salesforce upsert by a stable portal key so a create request retried after an ambiguous timeout does not create a duplicate?
8. How should a contact create behave if its customer record has not yet been mapped on the other system?
9. Is 10 edits/second a reasonable peak assumption? How many distinct records can one user action change?
10. What retention period is required for field-change history and webhook idempotency keys?
11. What should users see when two edits to the same field are close together or cannot be ordered confidently?

## Decision rule

Do not silently treat an assumption as a Salesforce feature. If an assumption is rejected, update the matching section in `docs/design.md`, recalculate the capacity, and record the decision in a new commit.
