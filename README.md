# olostep-n8n-investment-agent

Local Docker project for running an n8n investment research workflow with Olostep, OpenAI, and Gmail.

The workflow runs on a weekday schedule or manually, researches a comma-separated stock watchlist, asks OpenAI to create one Markdown report for the full watchlist, converts it into an HTML email, and sends it with Gmail.

This project is for research automation only. It does not provide financial advice, price targets, or buy/sell/hold recommendations.

## Files

- `docker-compose.yml` starts a local n8n instance.
- `workflows/olostep-n8n-investment-agent.workflow.json` is the importable n8n workflow.


## Requirements

- Docker and Docker Compose
- Olostep API key
- OpenAI API key
- Gmail OAuth2 credential in n8n

## Start n8n

Generate an encryption key:

```bash
openssl rand -hex 32
```

Paste it into `docker-compose.yml`:

```yaml
- N8N_ENCRYPTION_KEY=your_generated_key_here
```

Then start n8n:

```bash
docker compose up -d
```

Open n8n:

```text
http://localhost:5678
```

The compose file enables community packages and stores n8n data in the `n8n_data` Docker volume.

## Install Required n8n Node

The workflow uses the Olostep community node:

```text
n8n-nodes-olostep
```

In n8n, install it from:

```text
Settings -> Community nodes
```

Install the community node before importing the workflow so n8n can recognize the Olostep node type.

## Create Credentials

Create these credentials in n8n before or after importing the workflow:

- Olostep credential for the `Olostep Search - Investment Sources` node
- OpenAI credential for the `OpenAI - Create Full Watchlist Email Report` node
- Gmail OAuth2 credential for the `Send HTML Email via Gmail` node

The workflow file does not include API keys, OAuth tokens, or exported credential IDs. After import, open each credential-backed node and select your local credential.

## Import Workflow

In the n8n UI, choose:

```text
Import from File
```

Select:

```text
workflows/olostep-n8n-investment-agent.workflow.json
```

The workflow is exported with `active: false`. Keep it inactive until credentials, email settings, and the schedule are reviewed.

## Workflow

The workflow performs these steps:

1. `Weekday Market Open Trigger` runs at `30 9 * * 1-5`, which is 9:30 AM US Eastern time.
2. `Manual Test Trigger` lets you run the workflow manually.
3. `Investment Watchlist` defines tickers, recipient email, sender email, and subject prefix.
4. `Split Tickers` creates one research query per ticker.
5. `Olostep Search - Investment Sources` searches for filings, investor relations pages, earnings transcripts, news, risks, and catalysts.
6. `Combine Ticker Research` merges ticker results into one watchlist payload.
7. `OpenAI - Create Full Watchlist Email Report` creates a single Markdown report.
8. `Build HTML Email Body` converts the Markdown report into a styled HTML email.
9. `Send HTML Email via Gmail` sends the final report.

## Customize

Open the `Investment Watchlist` node and update:

- `tickers`, for example `MSFT,NVDA,AAPL`
- `email_to`
- `email_from`
- `email_subject_prefix`

Open the `Weekday Market Open Trigger` node if you need a different schedule. Docker Compose sets n8n to `America/New_York`, so the current cron expression runs at 9:30 AM US Eastern time on weekdays and follows Eastern daylight saving time.

The schedule does not automatically skip US market holidays. Disable the workflow or add a market-calendar check if you need holiday-aware execution.

Keep the output research-focused. Do not modify prompts to produce personalized financial advice or direct trading recommendations.

## Security Notes

Do not commit API keys, OAuth tokens, passwords, or a real `N8N_ENCRYPTION_KEY`. The committed compose file should keep only the placeholder value.

The current workflow export has been scrubbed of:

- Raw API keys
- n8n credential IDs
- OAuth tokens
- Personal email defaults
- n8n instance metadata

n8n stores configured credentials inside the local `n8n_data` Docker volume. If you later export the workflow from your own n8n instance, review the JSON before committing it because n8n exports may include credential references, node metadata, personal email addresses, or instance IDs.

## Stop n8n

```bash
docker compose down
```

To also remove local n8n state and configured credentials:

```bash
docker compose down -v
```
