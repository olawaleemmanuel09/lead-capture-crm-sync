# Lead Capture & CRM Sync — n8n Workflow

Production-grade n8n automation that captures form leads and syncs them across a full sales stack in seconds — zero manual data entry, zero missed leads.

**Flow:** Tally form → Webhook → Parse & Validate → HubSpot (upsert) → Gmail confirmation + Slack alert + Google Sheets log (in parallel)

![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71) ![License](https://img.shields.io/badge/license-MIT-blue) ![Level](https://img.shields.io/badge/level-intermediate-green)

## What It Does

| Step | Action |
|---|---|
| 1 | A visitor submits a Tally form; Tally fires a webhook to n8n instantly |
| 2 | A Code node parses the payload by field label, normalizes name casing, lowercases + validates the email, and resolves dropdown option IDs to text |
| 3 | Invalid emails are filtered out before they ever touch the CRM |
| 4 | HubSpot **upserts** the contact by email — existing contacts are updated, never duplicated |
| 5 | The lead receives a personalized HTML confirmation via Gmail |
| 6 | The team is alerted in Slack (`#leads`) with the full lead details |
| 7 | Every lead is appended to a Google Sheet as a permanent backup / reporting layer |

Typical end-to-end latency: under 10 seconds from form submit to Slack alert.

## Why It Matters

Service businesses lose deals to slow follow-up. This workflow drops response time from hours to seconds, keeps the CRM permanently in sync, and removes manual entry errors — the kind of automation that directly pays for itself in recovered leads.

## Stack

- **n8n** (self-hosted or cloud) — orchestration
- **Tally** — form + webhook source
- **HubSpot** (free tier) — CRM, Private App token auth
- **Gmail** — lead confirmation email (Google OAuth2)
- **Slack** — team notifications (bot token)
- **Google Sheets** — backup log (Google OAuth2)

## Repository Contents

```
workflows/lead-capture-crm-sync.json   # Import-ready n8n workflow
docs/setup-guide.pdf                   # Full setup guide: accounts, credentials, node-by-node explanation
```

## Quick Start

1. **Import** `workflows/lead-capture-crm-sync.json` into n8n (Workflows → Import from File).
2. **Create accounts / credentials** — the PDF in `docs/` walks through every step:
   - Tally form with fields labeled exactly: `Full Name`, `Email`, `Company`, `Service Interest`, `Message`
   - HubSpot Private App token with `crm.objects.contacts.read` + `crm.objects.contacts.write`
   - Slack bot token with `chat:write`, `chat:write.public`, `channels:read`; channel named `leads`
   - Google Cloud OAuth client for Gmail + Sheets
3. **HubSpot custom properties** — create two contact properties (Settings → Properties → Contact): `service_interest` (single-line text) and `lead_message` (multi-line text). The workflow maps into them.
4. **Google Sheet** — create a sheet with row-1 headers: `Timestamp | Full Name | Email | Company | Service Interest | Message | Source`, then select it in the Sheets node.
5. **Assign credentials** on each node (none are embedded — n8n credential IDs are instance-specific by design).
6. **Connect Tally**: copy the Webhook node's Production URL into Tally → Integrations → Webhooks, and **activate** the workflow.

## Design Decisions

- **Label-based parsing** — fields are matched by Tally label, not array position, so reordering form fields never breaks the workflow.
- **Native upsert over search-and-branch** — HubSpot's create-or-update by email handles dedupe in one API call: fewer nodes, fewer failure points.
- **Validation gate before side effects** — junk submissions are dropped before any CRM write, email, or alert fires.
- **Cross-node references** (`$('Parse & Clean Lead').item.json.*`) — downstream nodes read the cleaned data directly, independent of what sits between them in the chain.
- **No embedded credentials** — the JSON is safely shareable; every secret is assigned locally at import time.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Fields arrive empty | Tally field label doesn't exactly match — rename in Tally |
| Webhook never fires | Workflow not activated, or Tally holds a stale/test URL |
| `PROPERTY_DOESNT_EXIST` from HubSpot | Create the two custom contact properties (step 3) |
| Slack `channel_not_found` | Channel isn't named `leads`, or missing `channels:read` — re-install the Slack app after adding scopes |
| Sheet columns misaligned | Row-1 header typo — headers must match exactly |

Full table with causes in `docs/setup-guide.pdf`.

## License

MIT — use it, fork it, deploy it for clients.
