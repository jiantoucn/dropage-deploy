---
name: dropage-deploy
description: Deploy an HTML page to the internet and return a public URL. Use when the user asks to deploy, host, share, or publish an HTML file or a zip archive containing an index.html. Triggers on phrases like "deploy this page", "host this online", "share this HTML", "upload to web", "get a link".
disable-model-invocation: true
allowed-tools: Bash(curl *)
---

# Deploy to Dropage

Upload an HTML file or zip archive to `dropage.online` and return a public URL that expires in 1 hour.

## Platform Detection

Before running the upload command, detect the user's platform and use the appropriate curl command:

- **Windows**: `curl.exe`
- **macOS/Linux**: `curl`

## Steps

1. Identify the file to deploy. Accept a single `.html` file or a `.zip` archive whose root contains `index.html`.

2. Detect platform and run the upload command:

### Windows (PowerShell / CMD)

```bash
curl.exe -s -F "file=@<filepath>" https://dropage.online/api/upload
```

### macOS / Linux (Bash / Zsh)

```bash
curl -s -F "file=@<filepath>" https://dropage.online/api/upload
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

## Examples

### Deploy a single HTML file

```bash
# Windows
curl.exe -s -F "file=@index.html" https://dropage.online/api/upload

# macOS/Linux
curl -s -F "file=@index.html" https://dropage.online/api/upload
```

### Deploy a zip archive

```bash
# Windows
curl.exe -s -F "file=@site.zip" https://dropage.online/api/upload

# macOS/Linux
curl -s -F "file=@site.zip" https://dropage.online/api/upload
```

### Deploy from specific path

```bash
# Windows
curl.exe -s -F "file=@C:\Users\me\project\index.html" https://dropage.online/api/upload

# macOS/Linux
curl -s -F "file=@/home/user/project/index.html" https://dropage.online/api/upload
```

## Troubleshooting

### "file not found" error
- Verify the file path is correct
- On Windows, use backslashes or escape forward slashes
- On macOS/Linux, use forward slashes

### HTTP 413 (Payload Too Large)
- File exceeds 10 MB limit
- Compress images or assets before deploying
- Use zip format for multi-file projects

### HTTP 429 (Too Many Requests)
- Rate limit exceeded (3/min, 30/30min, 45/hr per IP)
- Wait and retry after the cooldown period

### HTTP 400 (Bad Request)
- File type not supported (must be .html or .zip)
- Zip file doesn't contain index.html at root level
- Zip contains more than 50 files