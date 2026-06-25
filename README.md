<p align="right">
  <a href="#english">English</a> · <a href="#中文">中文</a> · <a href="#日本語">日本語</a>
</p>

---

# dropage-deploy

<a name="english"></a>

## English

Upload an HTML page or zip archive to [dropage.online](https://dropage.online) and get a public URL that expires in 1 hour.

### Triggers

Use this skill when the user asks to:

- "deploy this page"
- "host this online"
- "share this HTML"
- "upload to web"
- "get a link"

### Usage

Send an `.html` file or a `.zip` archive (with `index.html` at the root) to deploy:

```bash
curl.exe -s -F "file=@your-file.html" https://dropage.online/api/upload
```

### Success Response (HTTP 201)

```json
{
  "success": true,
  "url": "https://dropage.online/{id}/",
  "expires_at": "...",
  "id": "..."
}
```

### Failure Response (HTTP 4xx)

```json
{
  "success": false,
  "error": "reason"
}
```

### Limits

| Item | Limit |
| --- | --- |
| File types | `.html` or `.zip` only |
| Max size | 10 MB |
| Zip requirement | Must contain `index.html` at the root level |
| Rate limit | 3/min, 30/30min, 45/hr per IP |

### Notes

- On success, the public URL and expiration time will be displayed.
- On HTTP 429 (rate limited), wait and retry.

---

<a name="中文"></a>

## 中文

将 HTML 页面或 zip 压缩包上传到 [dropage.online](https://dropage.online)，获取一个 1 小时内有效的公开链接。

### 触发条件

当用户说出以下内容时使用此技能：

- "部署这个页面"
- "在线托管"
- "分享这个 HTML"
- "上传到网上"
- "给我一个链接"

### 使用方法

发送 `.html` 文件或 `.zip` 压缩包（根目录需包含 `index.html`）进行部署：

```bash
curl.exe -s -F "file=@your-file.html" https://dropage.online/api/upload
```

### 成功响应 (HTTP 201)

```json
{
  "success": true,
  "url": "https://dropage.online/{id}/",
  "expires_at": "...",
  "id": "..."
}
```

### 失败响应 (HTTP 4xx)

```json
{
  "success": false,
  "error": "reason"
}
```

### 限制

| 项目 | 限制 |
| --- | --- |
| 文件类型 | 仅支持 `.html` 或 `.zip` |
| 最大大小 | 10 MB |
| 压缩包要求 | 根目录必须包含 `index.html` |
| 频率限制 | 每分钟 3 次，每 30 分钟 30 次，每小时 45 次 (按 IP) |

### 说明

- 成功时会显示公开链接和过期时间。
- 遇到 HTTP 429（频率限制）时请稍后重试。

---

<a name="日本語"></a>

## 日本語

HTML ページまたは zip アーカイブを [dropage.online](https://dropage.online) にアップロードし、1 時間有効な公開 URL を取得します。

### トリガー条件

ユーザーが以下のような発言をした場合にこのスキルを使用します：

- "このページをデプロイして"
- "オンラインでホストして"
- "このHTMLを共有して"
- "ウェブにアップロードして"
- "リンクを取得したい"

### 使い方

`.html` ファイルまたは `.zip` アーカイブ（ルートに `index.html` が必要）を送信してデプロイ：

```bash
curl.exe -s -F "file=@your-file.html" https://dropage.online/api/upload
```

### 成功レスポンス (HTTP 201)

```json
{
  "success": true,
  "url": "https://dropage.online/{id}/",
  "expires_at": "...",
  "id": "..."
}
```

### 失敗レスポンス (HTTP 4xx)

```json
{
  "success": false,
  "error": "reason"
}
```

### 制限事項

| 項目 | 制限 |
| --- | --- |
| ファイル形式 | `.html` または `.zip` のみ |
| 最大サイズ | 10 MB |
| ZIP 要件 | ルートレベルに `index.html` が必要 |
| レート制限 | 1分あたり 3 回、30分あたり 30 回、1時間あたり 45 回（IP ごと） |

### 備考

- 成功時には公開 URL と有効期限が表示されます。
- HTTP 429（レート制限）の場合は、しばらく待ってから再試行してください。
