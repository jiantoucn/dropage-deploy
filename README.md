<p align="right">
  <a href="#english">English</a> · <a href="#中文">中文</a> · <a href="#日本語">日本語</a>
</p>

<p align="center">
  可在以下平台获取 / Also available on:<br>
  <a href="https://clawhub.ai/jiantoucn/skills/dropage-deploy">ClawHub</a> ·
  <a href="https://skillhub.cloud.tencent.com/skills/dropage-deploy">Tencent SkillHub</a>
</p>

---

# dropage-deploy

<a name="english"></a>

## English

Upload an HTML page or zip archive to [dropage.online](https://dropage.online) and get a public URL. You can choose from multiple expiry times and optional visit limits.

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
# Default: 1 hour expiry, unlimited visits
curl.exe -s -F "file=@your-file.html" https://dropage.online/api/upload
```

```bash
# With custom expiry and visit limit
curl.exe -s -F "file=@your-file.html" -F "expiry=10m" -F "max_visits=1" https://dropage.online/api/upload
```

### Optional Fields

| Field | Values | Default | Description |
| --- | --- | --- | --- |
| `expiry` | `10m`, `1h`, `3h`, `12h` | `1h` | How long the page stays active before expiring |
| `max_visits` | `0` (unlimited), `1`, `10` | `0` | Page becomes inactive after this many visits |

### Success Response (HTTP 201)

```json
{
  "success": true,
  "url": "https://dropage.online/{id}/",
  "expires_at": "...",
  "max_visits": 0,
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
- If `max_visits` is set (1 or 10), the page will be disabled after that many visits.
- Expired records are physically deleted from the server after 24 hours.
- On HTTP 429 (rate limited), wait and retry.

### Examples

```bash
# 10-minute expiry, deleted after 1 visit
curl.exe -s -F "file=@index.html" -F "expiry=10m" -F "max_visits=1" https://dropage.online/api/upload

# 3-hour expiry, unlimited visits
curl.exe -s -F "file=@site.zip" -F "expiry=3h" https://dropage.online/api/upload

# 12-hour expiry, deleted after 10 visits
curl.exe -s -F "file=@page.html" -F "expiry=12h" -F "max_visits=10" https://dropage.online/api/upload
```

---

<a name="中文"></a>

## 中文

将 HTML 页面或 zip 压缩包上传到 [dropage.online](https://dropage.online)，获取一个公开链接。支持多种有效期和可选的访问次数限制。

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
# 默认：1 小时有效，不限访问次数
curl.exe -s -F "file=@your-file.html" https://dropage.online/api/upload
```

```bash
# 自定义有效期和访问次数
curl.exe -s -F "file=@your-file.html" -F "expiry=10m" -F "max_visits=1" https://dropage.online/api/upload
```

### 可选参数

| 参数 | 可选值 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `expiry` | `10m`, `1h`, `3h`, `12h` | `1h` | 页面有效时长 |
| `max_visits` | `0`（不限）, `1`, `10` | `0` | 页面被访问多少次后失效 |

### 成功响应 (HTTP 201)

```json
{
  "success": true,
  "url": "https://dropage.online/{id}/",
  "expires_at": "...",
  "max_visits": 0,
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
- 若设置 `max_visits`（1 或 10），页面在达到指定访问次数后自动失效。
- 过期的记录会在 24 小时后从服务器物理删除。
- 遇到 HTTP 429（频率限制）时请稍后重试。

### 示例

```bash
# 10 分钟有效，访问 1 次后失效
curl.exe -s -F "file=@index.html" -F "expiry=10m" -F "max_visits=1" https://dropage.online/api/upload

# 3 小时有效，不限访问次数
curl.exe -s -F "file=@site.zip" -F "expiry=3h" https://dropage.online/api/upload

# 12 小时有效，访问 10 次后失效
curl.exe -s -F "file=@page.html" -F "expiry=12h" -F "max_visits=10" https://dropage.online/api/upload
```

---

<a name="日本語"></a>

## 日本語

HTML ページまたは zip アーカイブを [dropage.online](https://dropage.online) にアップロードし、公開 URL を取得します。複数の有効期限とオプションのアクセス制限を選択できます。

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
# デフォルト：1時間有効、アクセス無制限
curl.exe -s -F "file=@your-file.html" https://dropage.online/api/upload
```

```bash
# 有効期限とアクセス制限をカスタマイズ
curl.exe -s -F "file=@your-file.html" -F "expiry=10m" -F "max_visits=1" https://dropage.online/api/upload
```

### オプションフィールド

| フィールド | 値 | デフォルト | 説明 |
| --- | --- | --- | --- |
| `expiry` | `10m`, `1h`, `3h`, `12h` | `1h` | ページの有効期限 |
| `max_visits` | `0`（無制限）, `1`, `10` | `0` | このアクセス数に達するとページを無効化 |

### 成功レスポンス (HTTP 201)

```json
{
  "success": true,
  "url": "https://dropage.online/{id}/",
  "expires_at": "...",
  "max_visits": 0,
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
- `max_visits` を設定した場合（1 または 10）、指定されたアクセス数に達するとページは無効になります。
- 期限切れのレコードは 24 時間後にサーバーから物理削除されます。
- HTTP 429（レート制限）の場合は、しばらく待ってから再試行してください。

### 例

```bash
# 10分有効、1回アクセスで削除
curl.exe -s -F "file=@index.html" -F "expiry=10m" -F "max_visits=1" https://dropage.online/api/upload

# 3時間有効、アクセス無制限
curl.exe -s -F "file=@site.zip" -F "expiry=3h" https://dropage.online/api/upload

# 12時間有効、10回アクセスで削除
curl.exe -s -F "file=@page.html" -F "expiry=12h" -F "max_visits=10" https://dropage.online/api/upload
```
