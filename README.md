<p align="right">
  <a href="#english">English</a> · <a href="#中文">中文</a> · <a href="#日本語">日本語</a>
</p>

---

# dropage-deploy

This repository contains the `dropage-deploy` skill for uploading an HTML page or a supported archive to [dropage.online](https://dropage.online) and returning a public URL.

> This repository contains the skill documentation only; it does not include Dropage server code, client code, or automated tests. The limits and responses below describe the current `SKILL.md` contract.

<a name="english"></a>

## English

### What it does

Upload a single HTML file or an archive containing a web page, then report the public URL, its expiration time, and its optional visit limit. The default public-link lifetime is 1 hour.

### When to use it

Use this skill when the user asks to:

- deploy or publish a page
- host a page online
- share an HTML page
- upload a site to the web
- get a public link

### Supported uploads

- Single `.html` files
- `.zip`, `.tar`, `.tar.gz`, and `.tgz` archives
- Maximum upload size: 10 MB
- Maximum archive entries: 50 files

An archive must contain `index.html` at the root level. If every entry is under one common top-level directory, Dropage strips that directory before checking the root. Do not use `.7z`, `.rar`, `.tar.bz2`, or `.tar.xz`. Avoid Chinese or other special characters in archive filenames to prevent encoding problems.

### Upload options

Both form fields are optional:

| Field | Values | Default |
| --- | --- | --- |
| `expiry` | `1h`, `6h`, `24h`, `7d`, `14d` | `1h` |
| `max_visits` | `0` (unlimited), `1`, `10` | `0` |

When `max_visits` is `1` or `10`, the public page is disabled after that many visits. New uploads use only the five current expiry values. Legacy clients may still send `10m`, `3h`, or `12h`. A successful legacy response adds this warning object:

```json
{
  "warning": {
    "code": "LEGACY_EXPIRY",
    "message": "...",
    "skill_url": "https://dropage.online/dropage-deploy.md"
  }
}
```

Report that the upload succeeded, ask the user to update the installed skill using `skill_url`, and do not retry the successful request.

### Upload commands

Detect the platform first. Windows uses `curl.exe`; macOS and Linux use `curl`.

#### Windows (PowerShell / CMD)

```bash
curl.exe -s -F "file=@<filepath>" -F "expiry=1h" -F "max_visits=0" https://dropage.online/api/upload
```

#### macOS / Linux (Bash / Zsh)

```bash
curl -s -F "file=@<filepath>" -F "expiry=1h" -F "max_visits=0" https://dropage.online/api/upload
```

Omit `expiry` and `max_visits` to use the defaults. Keep a user-requested expiry unchanged; do not silently shorten it to bypass a quota.

### Responses

A successful upload returns HTTP 201:

```json
{
  "success": true,
  "url": "https://dropage.online/a1b2c3d4e5f6/",
  "expires_at": "2026-06-29T14:00:00+08:00",
  "max_visits": 0,
  "id": "a1b2c3d4e5f6"
}
```

Report the public URL, expiration time, and visit limit when one is set.

Failures return HTTP 4xx, for example:

```json
{
  "success": false,
  "error": "reason"
}
```

For HTTP 429 responses, inspect both the JSON `code` and the `Retry-After` header. The general rate limiter can return a generic 429 before daily-quota checks, so not every 429 is a daily quota error.

### Daily quotas for long expiry

The `7d` and `14d` tiers have separate daily quotas. They reset at 00:00 UTC+8 on the next calendar day.

| Expiry | Site-wide successful uploads/day | Successful uploads/IP/day |
| --- | ---: | ---: |
| `7d` | 50 | 5 |
| `14d` | 10 | 1 |

The `1h`, `6h`, and `24h` tiers have no extra daily quota, but every tier still follows the per-IP and site-wide rate limits below. Only successful uploads count toward a daily quota, including pages that later expire or are cleaned up; deleting a page does not refund quota.

Before a `7d` or `14d` upload, tell the user the selected tier's site-wide and per-IP limits and the next reset time. If `DAILY_QUOTA_EXCEEDED` is returned, report these fields:

- `expiry` — the requested tier
- `scope` — `ip` or `global`
- `limit` — the exhausted allowance
- `reset_at` — the server's reset timestamp

A rejected upload creates no public URL or upload record and consumes no additional daily quota. Wait for the reported reset or let the user choose another tier; do not loop retries or silently change `expiry`.

If both the per-IP and site-wide daily allowances are exhausted, the API reports `scope: "ip"` first.

Example quota response:

```json
{
  "success": false,
  "error": "reason",
  "code": "DAILY_QUOTA_EXCEEDED",
  "expiry": "14d",
  "scope": "ip",
  "limit": 1,
  "reset_at": "2026-09-05T16:00:00.000Z"
}
```

Use `-i` when the HTTP status and `Retry-After` header are needed:

```bash
curl.exe -sS -i -F "file=@page.html" -F "expiry=14d" https://dropage.online/api/upload
```

The API's `error` text may be Chinese regardless of the page language; use the structured fields to explain the result in the user's language.

### Limits and retention

| Item | Limit or behavior |
| --- | --- |
| File types | `.html`, `.zip`, `.tar`, `.tar.gz`, `.tgz` only |
| Max size | 10 MB |
| Archive | `index.html` at root after common-directory stripping; max 50 files |
| Expiry | `1h` / `6h` / `24h` / `7d` / `14d` |
| Visit limit | `0` (unlimited) / `1` / `10` |
| Per-IP rate limit | 3/min, 30/30min, 45/hour |
| Site-wide rate limit | 1,000 uploads/hour |
| Uploaded-file retention | Physical files are deleted 30 days after upload; metadata remains |
| Access-log retention | 30 days |

File retention is independent of public-link expiry. Expiry and visit limits can disable public access earlier. Existing uploads keep their expiration times.

### Examples

Deploy a single HTML file with defaults:

```bash
# Windows
curl.exe -s -F "file=@index.html" https://dropage.online/api/upload

# macOS/Linux
curl -s -F "file=@index.html" https://dropage.online/api/upload
```

Deploy an archive:

```bash
# Windows
curl.exe -s -F "file=@site.zip" https://dropage.online/api/upload
curl.exe -s -F "file=@site.tar.gz" https://dropage.online/api/upload

# macOS/Linux
curl -s -F "file=@site.zip" https://dropage.online/api/upload
curl -s -F "file=@site.tar.gz" https://dropage.online/api/upload
```

Set a visit limit:

```bash
# Windows
curl.exe -s -F "file=@index.html" -F "expiry=1h" -F "max_visits=1" https://dropage.online/api/upload

# macOS/Linux
curl -s -F "file=@index.html" -F "expiry=1h" -F "max_visits=1" https://dropage.online/api/upload
```

Choose a long expiry:

```bash
# Windows: 7 days
curl.exe -s -F "file=@index.html" -F "expiry=7d" https://dropage.online/api/upload

# macOS/Linux: 14 days
curl -s -F "file=@index.html" -F "expiry=14d" https://dropage.online/api/upload
```

For 6-hour or 24-hour expiry, use `-F "expiry=6h"` or `-F "expiry=24h"`.

### Troubleshooting

#### File not found

- Verify that the path exists.
- Windows accepts backward slashes or absolute paths, such as `@C:\path\to\file.html`.
- macOS/Linux uses paths such as `@/home/user/file.html`.
- For spaces in a Windows path, quote the form value, such as `@"C:\path with spaces\file.html"`; use the equivalent quoted path on macOS/Linux.

#### Empty file or unsupported type

- The file must be non-empty and use a supported extension, case-insensitively.
- Hidden files whose names start with a dot are not supported.

#### HTTP 400 — unsupported format

Use only `.html`, `.zip`, `.tar`, `.tar.gz`, or `.tgz`. `.htm`, `.xhtml`, `.rar`, `.7z`, `.tar.bz2`, and `.tar.xz` are not supported.

#### HTTP 400 — archive error

- Ensure the archive contains `index.html` at the root after common-directory stripping.
- Keep the archive at or below 50 files.
- Check that the archive is not corrupted or truncated and uses supported compression.
- A `.tar.gz` must be a gzip-compressed tar archive, not a raw gzip-compressed single file.

#### HTTP 413 — payload too large

The upload exceeds 10 MB. Compress or reduce assets, or split a large project into multiple deployments.

#### HTTP 429 — too many requests

- The general per-IP limit is 3/min, 30/30min, and 45/hour; respect `Retry-After`.
- For `DAILY_QUOTA_EXCEEDED`, report the affected tier, scope, allowance, and reset time.
- Failed uploads do not consume daily quota; successful uploads continue to count even after expiration or cleanup.
- Do not repeatedly retry or silently select a shorter expiry.

---

<a name="中文"></a>

## 中文

### 功能

`dropage-deploy` 用于将单个 HTML 文件或包含网页入口的压缩包上传到 [dropage.online](https://dropage.online)，并返回公开链接、过期时间和可选的访问次数限制。公开链接默认有效 1 小时。

> 本仓库只包含技能文档，不包含 Dropage 服务端、客户端或自动化测试；下方限制和响应描述的是当前 `SKILL.md` 契约。

### 触发条件

当用户要求以下操作时使用此技能：

- 部署或发布页面
- 在线托管页面
- 分享 HTML 页面
- 将网站上传到网络
- 获取公开链接

### 支持的文件

- 单个 `.html` 文件
- `.zip`、`.tar`、`.tar.gz`、`.tgz` 压缩包
- 最大上传大小：10 MB
- 压缩包最多 50 个文件

压缩包根目录必须包含 `index.html`。如果所有条目都位于同一个顶层目录中，Dropage 会在检查根目录前自动移除该目录。`.7z`、`.rar`、`.tar.bz2` 和 `.tar.xz` 不支持；压缩包文件名应避免使用中文或其他特殊字符，以免发生编码问题。

### 上传选项

两个表单字段均可省略：

| 字段 | 可用值 | 默认值 |
| --- | --- | --- |
| `expiry` | `1h`、`6h`、`24h`、`7d`、`14d` | `1h` |
| `max_visits` | `0`（不限次数）、`1`、`10` | `0` |

设置为 `1` 或 `10` 后，页面在达到访问次数时会被禁用。新上传只能使用上述五种有效期；旧客户端仍可发送 `10m`、`3h` 或 `12h`。成功响应中的 `warning` 对象包含 `code: "LEGACY_EXPIRY"`、`message` 和 `skill_url: "https://dropage.online/dropage-deploy.md"`；应报告上传成功，要求用户通过 `skill_url` 更新已安装技能，且不要重试成功请求。

### 上传命令

先判断平台：Windows 使用 `curl.exe`，macOS/Linux 使用 `curl`。

#### Windows（PowerShell / CMD）

```bash
curl.exe -s -F "file=@<filepath>" -F "expiry=1h" -F "max_visits=0" https://dropage.online/api/upload
```

#### macOS / Linux（Bash / Zsh）

```bash
curl -s -F "file=@<filepath>" -F "expiry=1h" -F "max_visits=0" https://dropage.online/api/upload
```

省略 `expiry` 和 `max_visits` 即使用默认值。用户指定了有效期时必须保持该请求，不得为了绕过配额而静默缩短有效期。

### 响应处理

成功响应为 HTTP 201，并包含 `url`、`expires_at`、`max_visits` 和 `id`。成功时报告公开链接、过期时间，以及设置过的访问次数限制。失败响应为 HTTP 4xx，并包含错误原因。

遇到 HTTP 429 时同时检查 JSON 的 `code` 和 `Retry-After`。通用限流可能在每日配额检查前返回普通 429，因此不能把每个 429 都当作每日配额错误。API 的 `error` 文本目前无论页面语言如何都可能是中文，应依据结构化字段向用户解释。

### 长有效期的每日配额

`7d` 和 `14d` 两档分别有每日配额，均在次日 00:00（UTC+8）重置：

| 有效期 | 全站每日成功上传数 | 单 IP 每日成功上传数 |
| --- | ---: | ---: |
| `7d` | 50 | 5 |
| `14d` | 10 | 1 |

`1h`、`6h`、`24h` 没有额外的每日配额，但所有档位仍受 IP 限流和全站限流约束。只有成功上传计入每日配额，包括后来过期或清理的页面；删除页面不会返还配额。

进行 `7d` 或 `14d` 上传前，应告知用户该档位的全站/单 IP 限额和下一次重置时间。若返回 `DAILY_QUOTA_EXCEEDED`，报告 `expiry`、`scope`（`ip` 或 `global`）、`limit` 和 `reset_at`。被拒绝的上传不会生成公开链接或上传记录，也不会额外消耗每日配额；应等待重置或让用户选择其他档位，不要循环重试或静默改短有效期。如果单 IP 和全站每日配额同时耗尽，API 会优先返回 `scope: "ip"`。

### 限制与保留

| 项目 | 限制或行为 |
| --- | --- |
| 文件类型 | 仅 `.html`、`.zip`、`.tar`、`.tar.gz`、`.tgz` |
| 最大大小 | 10 MB |
| 压缩包 | 自动移除公共顶层目录后，根目录必须有 `index.html`；最多 50 个文件 |
| 有效期 | `1h` / `6h` / `24h` / `7d` / `14d` |
| 访问次数 | `0`（不限）/ `1` / `10` |
| 单 IP 限流 | 每分钟 3 次、每 30 分钟 30 次、每小时 45 次 |
| 全站限流 | 每小时 1,000 次上传 |
| 上传文件保留 | 上传 30 天后删除实体文件，保留元数据 |
| 访问日志保留 | 30 天 |

文件保留时间与公开链接有效期相互独立；有效期或访问次数限制可能更早关闭公开访问。已有上传不会改变其原有过期时间。

### 示例

```bash
# Windows：使用默认值上传 HTML
curl.exe -s -F "file=@index.html" https://dropage.online/api/upload

# macOS/Linux：上传 tar.gz
curl -s -F "file=@site.tar.gz" https://dropage.online/api/upload

# Windows：1 小时有效，访问 1 次后禁用
curl.exe -s -F "file=@index.html" -F "expiry=1h" -F "max_visits=1" https://dropage.online/api/upload

# macOS/Linux：14 天有效（受每日配额约束）
curl -s -F "file=@index.html" -F "expiry=14d" https://dropage.online/api/upload
```

### 排错

- 文件不存在：确认路径存在；Windows 可使用反斜杠或绝对路径，macOS/Linux 使用正斜杠；路径包含空格时请引用整个表单值。
- 文件为空或格式不支持：文件必须非空，扩展名必须是支持的格式（不区分大小写）；以点开头的隐藏文件不支持。
- HTTP 400：确认扩展名和压缩包根目录的 `index.html`；压缩包不得超过 50 个文件，且不能损坏或截断。
- HTTP 413：文件超过 10 MB，请压缩/减少资源或拆分部署。
- HTTP 429：遵守 `Retry-After`。普通限流为每 IP 每分钟 3 次、每 30 分钟 30 次、每小时 45 次；若是 `DAILY_QUOTA_EXCEEDED`，报告档位、范围、限额和重置时间，不要循环重试或静默改短有效期。

---

<a name="日本語"></a>

## 日本語

### 機能

`dropage-deploy` は、単一の HTML ファイルまたは Web ページを含むアーカイブを [dropage.online](https://dropage.online) にアップロードし、公開 URL、有効期限、任意のアクセス回数制限を返すスキルです。公開リンクのデフォルト有効期間は 1 時間です。

> このリポジトリにはスキル文書だけが含まれ、Dropage のサーバーコード、クライアントコード、自動テストは含まれません。以下の制限とレスポンスは現在の `SKILL.md` 契約を示します。

### 使用する場面

次のような依頼で使用します：

- ページをデプロイまたは公開する
- ページをオンラインでホストする
- HTML ページを共有する
- Web にサイトをアップロードする
- 公開リンクを取得する

### 対応ファイル

- 単一の `.html` ファイル
- `.zip`、`.tar`、`.tar.gz`、`.tgz` アーカイブ
- 最大アップロードサイズ：10 MB
- アーカイブ内の最大ファイル数：50

アーカイブのルートには `index.html` が必要です。すべてのエントリが同じトップレベルディレクトリに入っている場合、Dropage はルート確認前にそのディレクトリを自動的に除去します。`.7z`、`.rar`、`.tar.bz2`、`.tar.xz` は非対応です。文字コード問題を避けるため、アーカイブのファイル名には中国語や特殊文字をできるだけ使用しないでください。

### アップロードオプション

両方のフォームフィールドは省略できます：

| フィールド | 値 | デフォルト |
| --- | --- | --- |
| `expiry` | `1h`、`6h`、`24h`、`7d`、`14d` | `1h` |
| `max_visits` | `0`（無制限）、`1`、`10` | `0` |

`max_visits` を `1` または `10` にすると、その回数に達した時点で公開ページが無効になります。新しいアップロードでは現在の 5 種類の有効期間だけを使用します。旧クライアントの `10m`、`3h`、`12h` は受け付けられますが、成功レスポンスの `warning` オブジェクトには `code: "LEGACY_EXPIRY"`、`message`、`skill_url: "https://dropage.online/dropage-deploy.md"` が含まれます。アップロードが成功したことを伝え、`skill_url` からインストール済みスキルを更新するよう案内し、成功したリクエストを再試行しないでください。

### アップロードコマンド

先にプラットフォームを判定します。Windows は `curl.exe`、macOS/Linux は `curl` を使用します。

#### Windows（PowerShell / CMD）

```bash
curl.exe -s -F "file=@<filepath>" -F "expiry=1h" -F "max_visits=0" https://dropage.online/api/upload
```

#### macOS / Linux（Bash / Zsh）

```bash
curl -s -F "file=@<filepath>" -F "expiry=1h" -F "max_visits=0" https://dropage.online/api/upload
```

`expiry` と `max_visits` を省略するとデフォルト値になります。ユーザーが指定した有効期間を、クォータ回避のために黙って短縮してはいけません。

### レスポンス処理

成功時は HTTP 201 で、`url`、`expires_at`、`max_visits`、`id` が返ります。成功時には公開 URL、有効期限、設定されているアクセス回数制限を伝えます。失敗時は HTTP 4xx とエラー理由が返ります。HTTP 429 では JSON の `code` と `Retry-After` を確認してください。一般的なレート制限が日次クォータ確認より先に実行されるため、すべての 429 が日次クォータエラーとは限りません。API の `error` テキストはページの言語に関係なく現在中国語の場合があるため、構造化されたフィールドに基づいて説明してください。

### 長い有効期間の日次クォータ

`7d` と `14d` には別々の日次クォータがあり、翌日の 00:00（UTC+8）にリセットされます。

| 有効期間 | サイト全体の成功数/日 | IP ごとの成功数/日 |
| --- | ---: | ---: |
| `7d` | 50 | 5 |
| `14d` | 10 | 1 |

`1h`、`6h`、`24h` には追加の日次クォータはありませんが、すべての期間が IP ごとのレート制限とサイト全体のレート制限に従います。成功したアップロードだけがクォータを消費し、後で期限切れ・クリーンアップされたページも対象です。削除してもクォータは戻りません。

`7d` または `14d` を使う前に、選択した期間のサイト全体/IP ごとの上限と次のリセット時刻を伝えます。`DAILY_QUOTA_EXCEEDED` の場合は `expiry`、`scope`（`ip` または `global`）、`limit`、`reset_at` を伝えてください。拒否されたアップロードには公開 URL やレコードは作成されず、追加のクォータも消費しません。リセットを待つか別の期間を選んでもらい、再試行ループや有効期間の黙った変更は行わないでください。IP ごととサイト全体のクォータが同時に尽きた場合、API は先に `scope: "ip"` を返します。

### 制限と保持期間

| 項目 | 制限または動作 |
| --- | --- |
| ファイル形式 | `.html`、`.zip`、`.tar`、`.tar.gz`、`.tgz` のみ |
| 最大サイズ | 10 MB |
| アーカイブ | 共通トップレベルディレクトリ除去後のルートに `index.html`、最大 50 ファイル |
| 有効期間 | `1h` / `6h` / `24h` / `7d` / `14d` |
| アクセス回数 | `0`（無制限）/ `1` / `10` |
| IP ごとの制限 | 3/分、30/30分、45/時 |
| サイト全体の制限 | 1,000 アップロード/時 |
| アップロードファイル保持 | アップロードから 30 日後に実体を削除し、メタデータは保持 |
| アクセスログ保持 | 30 日 |

ファイル保持期間と公開リンクの有効期間は独立しています。有効期限またはアクセス回数制限により、公開アクセスが先に無効になる場合があります。既存のアップロードの有効期限は変更されません。

### 例

```bash
# Windows：HTML をデフォルト値でアップロード
curl.exe -s -F "file=@index.html" https://dropage.online/api/upload

# macOS/Linux：tar.gz をアップロード
curl -s -F "file=@site.tar.gz" https://dropage.online/api/upload

# Windows：1 時間有効、1 回のアクセス後に無効化
curl.exe -s -F "file=@index.html" -F "expiry=1h" -F "max_visits=1" https://dropage.online/api/upload

# macOS/Linux：14 日有効（日次クォータの対象）
curl -s -F "file=@index.html" -F "expiry=14d" https://dropage.online/api/upload
```

### トラブルシューティング

- ファイルが見つからない：パスが存在することを確認します。Windows はバックスラッシュまたは絶対パス、macOS/Linux はスラッシュを使用し、空白を含むパスはフォーム値全体を引用します。
- 空ファイルまたは非対応形式：空でないファイルで、対応する拡張子（大文字小文字は不問）を使用します。ドットで始まる隠しファイルは非対応です。
- HTTP 400：拡張子と、アーカイブのルートにある `index.html` を確認します。ファイル数は 50 以下、アーカイブは破損・切り詰め状態でない必要があります。
- HTTP 413：10 MB を超えています。アセットを圧縮・削減するか、複数のデプロイに分割します。
- HTTP 429：`Retry-After` に従います。通常の制限は IP ごとに 3/分、30/30分、45/時です。`DAILY_QUOTA_EXCEEDED` なら期間、範囲、上限、リセット時刻を伝え、再試行ループや有効期間の黙った短縮を行いません。
