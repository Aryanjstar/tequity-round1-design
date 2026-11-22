# Tequity Round 1: Two-way record sync

This repository contains the design for synchronizing customer and contact records between a Tequity portal and a client's Salesforce CRM.

## Contents

- [`Round1-Tequity.html`](Round1-Tequity.html) — source problem statement.
- [`docs/decisions.md`](docs/decisions.md) — assumptions and design decisions that need confirmation.
- [`docs/design.md`](docs/design.md) — implementation-ready system design.
- [`docs/walkthrough.md`](docs/walkthrough.md) — outline for the requested 10-minute review.

## Scope

This submission is a design document. The brief makes code optional; this repository does not claim to contain a production service or a tested Salesforce integration. Deletes, Salesforce login screens, a visual UI, a second region, and tests against a real Salesforce instance are out of scope.

## Review

Start with the assumptions and open questions in `docs/decisions.md`, then read `docs/design.md`. Items that depend on Salesforce API capabilities are marked as assumptions or questions rather than treated as confirmed features.
