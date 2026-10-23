# n8n Workflow Sanitizer

[Leer en español](README.es.md)

An **n8n** workflow that removes private data from the JSON of any other n8n workflow, so you can publish it (GitHub, a portfolio, a client delivery) without leaking credential IDs, webhook IDs, tokens or real test data.

## Why

When you export a workflow from n8n, the JSON usually contains much more than the logic of your automation:

- IDs of the credentials you used.
- Webhook IDs and the ID of your n8n instance.
- Data table and Google Sheets/Docs IDs, with their private URLs.
- API keys or tokens typed directly in a header or a field.
- Pinned (test) data, which often holds real phone numbers, names or messages.

Cleaning all that by hand is slow and easy to get wrong. This workflow does it in one execution and tells you exactly what it changed.

## How it works

```
Start
  │
  ▼
PUT JSON HERE                    You paste the workflow JSON here
  │
  ▼
Sensitive credential detector    Scans everything and lists the findings (changes nothing)
  │
  ▼
Hide sensitive credentials       Redacts each finding on a copy and clears the pinned data
  │
  ▼
Prepare JSON File                Builds the report and the file name
  │
  ▼
Convert to File                  Downloadable .json file
```

The original workflow is never modified: the redaction is applied to an independent copy.

## What it detects

| What | Replaced with |
|---|---|
| Fields named `password`, `token`, `apiKey`, `client_secret`, `private_key`... | `[REDACTED_SECRET]` |
| `{ name, value }` pairs whose name looks secret (HTTP headers, query params, Set fields): `Authorization`, `apikey`, `x-api-key`, `token`... | `[REDACTED_SECRET]` |
| Credential IDs (`credentials.*.id`) | `[REDACTED_CREDENTIAL_ID]` |
| Webhook IDs | `[REDACTED_WEBHOOK_ID]` |
| Data table IDs (and their cached URL) | `[REDACTED_DATATABLE_ID]` |
| Google Sheets/Docs `documentId` (and its cached URL) | `[REDACTED_DOCUMENT_ID]` |
| Any other `cachedResultUrl` | `[REDACTED_URL]` |
| `instanceId` and `versionId` | `[REDACTED_INSTANCE_ID]`, `[REDACTED_VERSION_ID]` |
| Connection strings (`postgres://user:pass@host`, MySQL, MongoDB) | `[REDACTED_CONNECTION_STRING]` |
| URLs with embedded credentials | `[REDACTED_URL_CREDENTIALS]` |
| JWT tokens | `[REDACTED_JWT]` |
| `Basic ...` and `Bearer ...` header values | `[REDACTED_BASIC_AUTH]`, `[REDACTED_BEARER_TOKEN]` |
| Private keys (`-----BEGIN ... PRIVATE KEY-----`) | `[REDACTED_PRIVATE_KEY]` |
| Pinned data of every node | Cleared (`pinData: {}`) |

Things that are **kept on purpose**: n8n expressions (`={{ $env.API_KEY }}`, `=Bearer {{ $json.token }}`), empty values, credential names and normal text such as sticky notes that mention "Basic authentication".

## Usage

1. Import `n8n-workflow-sanitizer.json` into your n8n.
2. Copy the JSON of the workflow you want to clean (select all nodes and copy, or download the workflow).
3. Open the **PUT JSON HERE** node and replace the whole content of the field with your JSON. It ships with a demo workflow full of fake secrets, so you can run it first and see how it works.
4. Execute the workflow.
5. Open **Prepare JSON File** and read the `report`: it lists every redacted path and its type (never the original values).
6. In **Convert to File**, download the file. It is named after your workflow, for example `whatsapp-message-buffer-sanitized.json`.
7. **Review the file** before publishing it.

To keep the pinned data, set `CLEAR_PIN_DATA` to `false` at the top of the **Hide sensitive credentials** node.

## Example

Before (demo workflow):

```json
{ "name": "apikey", "value": "FAKE-API-KEY-1234567890" }
```

```json
"documentId": {
  "__rl": true,
  "value": "FAKE_SPREADSHEET_ID_0001",
  "cachedResultUrl": "https://docs.google.com/spreadsheets/d/FAKE_SPREADSHEET_ID_0001/edit"
}
```

After:

```json
{ "name": "apikey", "value": "[REDACTED_SECRET]" }
```

```json
"documentId": {
  "__rl": true,
  "value": "[REDACTED_DOCUMENT_ID]",
  "cachedResultUrl": "[REDACTED_URL]"
}
```

## Limitations

This is a **heuristic** tool. It reduces the risk, it does not remove it.

- It cannot recognize a secret stored under a generic name (for example a plain text field called `value` with no `name` that explains it).
- Tokens inside a URL query string (`?key=...`) are not detected.
- Phone numbers, emails, names, private domains or IP addresses typed in parameters or Code nodes are not detected.
- IDs of Drive files and folders are not detected.
- Text patterns can produce false positives: a value that looks like a credential is redacted even if it is not one.
- n8n keeps the executions of this workflow, and they contain the JSON you pasted. Delete them after using it.

## Requirements

- A recent version of n8n (the **Convert to File** node v1.1 and, for the demo, Data Tables).
- No credentials are needed.

## License

Free to use and adapt.
