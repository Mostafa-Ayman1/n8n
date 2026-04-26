# RevOps Automation Workflows for n8n

Production-ready n8n workflow exports for a fully automated Revenue Operations pipeline connecting HubSpot CRM, WhatsApp Business API, Slack, Google Sheets, and Looker Studio.

## Workflows

| ID | Name | Trigger | Description |
|----|------|---------|-------------|
| W-01 | Lead Capture | Webhook POST `/lead-capture` | Validates input, creates/updates HubSpot contacts, triggers scoring |
| W-02 | Lead Scoring Engine | Sub-workflow / Schedule (6h) | Scores contacts (0-100) using demographic + behavioral signals |
| W-03 | Lead Routing | Sub-workflow / Webhook | Routes leads to reps by score tier, creates tasks, sends Slack alerts |
| W-04 | WhatsApp Integration | Sub-workflow / Webhook | Sends welcome templates, processes inbound replies, updates CRM |
| W-05 | Follow-up Sequence | Sub-workflow | Multi-day drip: Day 1/3/7/14 via WhatsApp + Email with reply checks |
| W-06 | Re-engagement Campaign | Schedule (daily 9 AM) / Sub-workflow | Targets 30+ day inactive contacts, marks churned if no response |
| W-07 | Deal Stage Updates | Webhook (deal change) | Audit trails, stage-based actions (Won/Lost/Proposal), 90-day forecast |
| W-08 | VIP Upgrade Trigger | Sub-workflow | VIP flag, senior rep assignment, 30-min SLA task, management alerts |

## Workflow Dependencies

```
W-01 (Lead Capture) --> W-02 (Scoring) --> W-08 (VIP Upgrade)
                                       --> W-03 (Routing) --> W-05 (Follow-up) --> W-06 (Re-engagement)
W-04 (WhatsApp Inbound) --> W-02 (Rescore)
W-07 (Deal Updates) --> W-09 / W-12 (future workflows)
```

## Import Instructions

1. Open your n8n instance
2. Go to **Workflows** > **Import from File**
3. Import each `.json` file in order (W-01 through W-08)
4. After import, note each workflow's ID and set the corresponding environment variables
5. Configure credentials (HubSpot API, WhatsApp API, Slack Bot, SMTP, Google Sheets)
6. Activate workflows in reverse dependency order (W-08 first, W-01 last)

## Environment Variables

Set these in your n8n instance under **Settings > Environment Variables**:

### Workflow IDs (set after import)
- `W02_WORKFLOW_ID` - W-02 Lead Scoring Engine workflow ID
- `W05_WORKFLOW_ID` - W-05 Follow-up Sequence workflow ID
- `W06_WORKFLOW_ID` - W-06 Re-engagement Campaign workflow ID
- `W08_WORKFLOW_ID` - W-08 VIP Upgrade Trigger workflow ID
- `W09_WORKFLOW_ID` - W-09 workflow ID (future)
- `W12_WORKFLOW_ID` - W-12 workflow ID (future)

### HubSpot
- `HUBSPOT_API_KEY` - HubSpot Bearer token
- `HUBSPOT_OWNER_SENIOR_REP` - Owner ID for VIP/hot leads
- `HUBSPOT_OWNER_SALES_REP` - Owner ID for warm leads
- `HUBSPOT_OWNER_SDR` - Owner ID for cold leads

### WhatsApp
- `WHATSAPP_API_URL` - API base URL (default: `https://waba.360dialog.io/v1`)

### Slack Channels
- `SLACK_CHANNEL_VIP` - VIP lead notifications (default: `#vip-leads`)
- `SLACK_CHANNEL_SALES` - Sales team notifications (default: `#sales-leads`)
- `SLACK_CHANNEL_SDR` - SDR queue (default: `#sdr-queue`)
- `SLACK_CHANNEL_WINS` - Deal won celebrations (default: `#wins`)
- `SLACK_CHANNEL_MANAGEMENT` - Management alerts (default: `#management`)

### Email
- `SMTP_FROM_EMAIL` - Sender email for outbound emails

### Google Sheets
- `GOOGLE_SHEET_AUDIT_ID` - Spreadsheet ID for deal audit trail
- `GOOGLE_SHEET_FORECAST_ID` - Spreadsheet ID for forecast data

### Thresholds
- `VIP_THRESHOLD` - Revenue threshold for VIP status (default: `10000`)

## Required n8n Credentials

Configure these in **Settings > Credentials**:

| Credential Name | Type | Used By |
|-----------------|------|---------|
| HubSpot API Key | HTTP Header Auth (`Authorization: Bearer ...`) | W-01 to W-08 |
| WhatsApp API Key | HTTP Header Auth | W-04, W-05, W-06, W-08 |
| Slack Bot Token | HTTP Header Auth (`Authorization: Bearer ...`) | W-03, W-04, W-07, W-08 |
| SMTP Credentials | SMTP | W-05, W-06 |
| Google Sheets | Google Sheets OAuth2 | W-07 |

## Lead Scoring Tiers

| Score Range | Tier | Assignment | Follow-up |
|-------------|------|------------|-----------|
| 0-30 | Cold | SDR | Low-priority queue |
| 31-60 | Warm | Sales Rep | Standard sequence (W-05) |
| 61-80 | Hot | Senior Rep | Priority routing |
| 81-100 | VIP | Senior Rep | Immediate + VIP workflow (W-08) |

## Rate Limiting

All bulk HubSpot operations use `Split In Batches` nodes with 500ms delays to stay within the Starter plan limit of 100 requests per 10 seconds.
