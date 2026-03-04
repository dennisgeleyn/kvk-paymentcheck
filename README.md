# Stamhoofd KBC Payment Sync

A browser-based tool to automatically match KBC bank transfers with open payments in [Stamhoofd](https://www.stamhoofd.be/). Hosted on GitHub Pages — no installation or server required.

## What does it do?

When members pay by bank transfer, they're required to include a structured reference (e.g. `+++477/3051/24887+++`). This tool compares an exported KBC account statement with open payments in Stamhoofd and automatically marks matched payments as paid.

**Workflow:**
1. Export your account statement as a CSV from KBC Online
2. Open the tool in your browser
3. Log in with your password (first visit: set up your credentials)
4. Upload the CSV file
5. Review the matched payments and confirm

## Usage

1. Open `stamhoofd-betaling-sync.html` in your browser
2. **First visit:** enter your Organisation ID, API token, and choose a password — your credentials are encrypted and saved to `localStorage`
3. **Subsequent visits:** enter just your password to unlock
4. Drag and drop your KBC CSV export onto the upload area, or click to browse
5. The tool fetches open transfer payments from the Stamhoofd API, matches them by structured reference, and shows a preview table
6. Click **Confirm** to mark the matched payments as paid

## Authentication

Because this tool is hosted publicly on GitHub Pages, credentials are protected with a password-based encryption scheme:

- Your Organisation ID and API token are encrypted with **AES-256-GCM** using a key derived from your password via **PBKDF2** (310,000 iterations, SHA-256)
- The encrypted blob is stored in `localStorage` — your password is never stored anywhere
- Plain-text credentials exist only in memory during an active session
- The session **auto-locks after 30 minutes** of inactivity, or immediately when you switch away from the tab
- A manual **Lock** button is available in the header at all times
- Failed login attempts trigger an **exponential back-off** (5s → 10s → 20s → 60s max) to slow brute-force attempts against a stolen `localStorage` blob

To change your credentials or password, use the **✏ Change credentials** button in the header. To start over, use the reset link on the login screen (this wipes `localStorage`).

## KBC CSV Export

In KBC Online, go to **Accounts → Account statements → Export** and choose CSV format. The filename typically looks like:

```
export_BE96XXXXXXXXXXXX_YYYYMMDD_HHMM.csv
```

The tool handles semicolon-separated files with CR, LF, or CRLF line endings and an optional UTF-8 BOM. Files larger than 5 MB are rejected.

## Stamhoofd API

The tool uses two endpoints:

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/v191/organization/payments` | Fetch open transfer payments |
| `PATCH` | `/v191/organization/payments/:id` | Mark a payment as succeeded |

An API token can be created via **Stamhoofd → Settings → API access**.

## Security Notes

- No data is sent to any external server — all requests go directly from your browser to the Stamhoofd API
- Structured references are normalised (digits only) before comparison to handle formatting differences
- Each reference can only be matched once per session to prevent duplicate marking
- Payment IDs are validated as UUIDs before being sent to the API
- The tool locks immediately if the browser tab is hidden (e.g. switching apps)

## Technical Notes

- Pure HTML/CSS/JS — no frameworks, no build step, no dependencies
- Crypto via the browser's native [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) — no third-party crypto libraries
- Compatible with KBC CSV exports (semicolon-separated, CR line endings, UTF-8)
- Matches on structured reference (`+++ddd/dddd/ddddd+++`) from the `gestructureerde mededeling` column
- Falls back to `Vrije mededeling` and `Omschrijving` columns when no structured reference is present

## License

MIT
