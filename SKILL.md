---
name: dropage-deploy
description: Deploy an HTML page to the internet and return a public URL. Use when the user asks to deploy, host, share, or publish an HTML file or a zip archive containing an index.html. Triggers on phrases like "deploy this page", "host this online", "share this HTML", "upload to web", "get a link".
disable-model-invocation: true
allowed-tools: Bash(curl *)
---

# Deploy to Dropage

Upload an HTML file or zip archive to `dropage.online` and return a public URL that expires in 1 hour.

## Steps

1. Identify the file to deploy. Accept a single `.html` file or a `.zip` archive whose root contains `index.html`.
2. Run the upload command:

```bash
curl.exe -s -F "file=@$ARGUMENTS" https://dropage.online/api/upload
```

3. Parse the JSON response and report the result to the user.

## Response format

Success (HTTP 201):
```json
{"success": true, "url": "https://dropage.online/{id}/", "expires_at": "...", "id": "..."}
```

Failure (HTTP 4xx):
```json
{"success": false, "error": "reason"}
```

## Report to the user

On success, show:
- The public URL
- Expiration time (1 hour from upload)

On failure, show the error reason. If HTTP 429, tell the user to wait and retry.

## Limits

- File types: `.html` or `.zip` only
- Max size: 10 MB
- Zip must contain `index.html` at the root level
- Rate limit: 3 per minute, 30 per half hour, 45 per hour per IP
