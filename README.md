# n8n Workflows

Self-hosted n8n with external Python/JS task runners, plus workflows exported as code.

## Contents

| Path | Description |
| --- | --- |
| `workflows/quote-triage.json` | Webhook (`POST /webhook/quote-triage`) → enrich via HTTP → Python scoring → route to expedited or standard queue. |
| `workflows/global-error-handler.json` | Error Trigger workflow; set as the error workflow for `quote-triage`. |
| `docker-compose.yml` | n8n + external task runners. |
| `.env.example` | Template for required environment variables. |

## Setup

```powershell
Copy-Item .env.example .env   # then set a strong N8N_RUNNERS_AUTH_TOKEN
docker volume create n8n_data
docker compose up -d
```

Open http://localhost:5678.

## Importing workflows

Import the error handler first so the `errorWorkflow` reference in `quote-triage` resolves.

From the editor: **Workflows → Import from File**. Or via CLI:

```powershell
docker compose cp workflows n8n:/tmp/workflows
docker compose exec n8n n8n import:workflow --separate --input=/tmp/workflows
```

## Exporting changes

```powershell
docker compose exec n8n n8n export:workflow --all --separate --pretty --output=/tmp/export
docker compose cp n8n:/tmp/export/. workflows/
```

## Test request

```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:5678/webhook/quote-triage `
  -ContentType 'application/json' `
  -Body '{"customer_id":"DE-4092","sku":"LX20-LINEAR-GUIDE","qty":50,"notes":"Need 3D CAD confirmation and urgent delivery"}'
```
