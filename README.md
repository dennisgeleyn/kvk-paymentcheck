# Stamhoofd KBC Payment Sync

A simple browser-based tool to automatically match KBC bank transfers with open payments in [Stamhoofd](https://www.stamhoofd.be/).

## What does it do?

When members pay by bank transfer, they're required to include a structured reference (e.g. `+++477/3051/24887+++`). This tool compares an exported KBC account statement with open payments in Stamhoofd and automatically marks matched payments as paid.

**Workflow:**
1. Export your account statement as a CSV from KBC Online
2. Open the HTML tool in your browser
3. Enter your Stamhoofd organisation ID and API token
4. Upload the CSV file
5. Review the matched payments and confirm

## Usage

The tool is a single HTML file — no installation or server required.

1. Download `stamhoofd-betaling-sync.html`
2. Open it in your browser
3. Fill in:
   - **Organisation ID** – the UUID of your Stamhoofd organisation
   - **API Token** – a token with write access (created in your Stamhoofd settings)
4. Drag and drop your KBC CSV export onto the upload area, or click to browse
5. The tool fetches open transfer payments from the Stamhoofd API, matches them by structured reference, and shows a preview table
6. Click **Confirm** to mark the matched payments as paid

## KBC CSV Export

In KBC Online, go to **Accounts → Account statements → Export** and choose CSV format. The filename typically looks like:

```
export_BE96XXXXXXXXXXXX_YYYYMMDD_HHMM.csv
```

## Stamhoofd API

The tool uses two endpoints:

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/v191/organization/payments` | Fetch open transfer payments |
| `PATCH` | `/v191/organization/payments/:id` | Mark a payment as succeeded |

An API token can be created via **Stamhoofd → Settings → API access**.

## Security

- No data is sent to any external server — everything happens directly from your browser to the Stamhoofd API
- Your organisation ID and API token are never stored
- Structured references are normalised (digits only) before comparison to handle formatting differences
- Each reference can only be matched once per session to prevent duplicates

## Technical Notes

- Pure HTML/CSS/JS — no frameworks, no dependencies, no installation
- Compatible with KBC CSV exports (semicolon-separated, CR line endings, UTF-8)
- Matches on structured reference (`+++ddd/dddd/ddddd+++`) from the `gestructureerde mededeling` column
- Falls back to `Vrije mededeling` and `Omschrijving` columns when no structured reference is present

## License

MIT
