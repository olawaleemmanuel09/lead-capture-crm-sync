# Lead Capture & CRM Sync

A business integration demo that routes a form enquiry to the people responsible for follow-up. The [eight-node workflow](workflows/lead-capture-crm-sync.json) uses a Tally webhook, field parsing, email validation and a HubSpot contact upsert. Connected steps send a Gmail acknowledgement, notify Slack and log the lead in Google Sheets.

## Inspect the handoff

Use the [setup guide](docs/setup-guide.pdf) with a test form, disposable CRM records and test recipients. Inspect the parsed email before the CRM step, then check each downstream destination separately. Exercise missing fields, a repeated submission and a downstream provider failure. A successful contact write alone cannot establish that every notification was delivered.

The repo demonstrates the integration design. Delivery latency, recovery from partial failures and conversion impact require measured runs.
