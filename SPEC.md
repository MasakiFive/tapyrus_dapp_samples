# 仕様書 — workshop202207

## 1. プロジェクト概要

Tapyrus ブロックチェーン API (TapyrusAPI) を Ruby on Rails から操作するための学習・実装用 Web アプリケーション。
CLI (Rake タスク) および Web UI からブロックチェーンのアドレス・トークン・タイムスタンプを管理する。

---

## 2. 技術スタック

| 項目 | 内容 |
|---|---|
| 言語 | Ruby 3.1.2 |
| フレームワーク | Rails 7.0.3 |
| DB | PostgreSQL 14 |
| 実行環境 | Docker / Docker Compose |
| HTTP クライアント | Faraday + faraday_middleware |
| フロントエンド | Bootstrap 5 + Importmap + Hotwire (Turbo/Stimulus) |
| ページネーション | kaminari |
| Web サーバ | Puma |

---

## 3. 認証・接続設定

### 3.1 TapyrusAPI 認証方式

| 種類 | 内容 |
|---|---|
| Bearer トークン | `ACCESS_TOKEN` 定数。リクエストヘッダ `Authorization: Bearer <token>` で送信 |
| クライアント証明書 | PKCS12 形式 (`.p12`)。SSL クライアント認証に使用 |

### 3.2 設定箇所

`lib/utils/tapyrus_api.rb` の先頭定数。

```ruby
ACCESS_TOKEN            = '<アクセストークン>'
TAPYRUS_API_ENDPOINT_URL = '<エンドポイント URL>'
```

証明書ファイルはプロジェクトルートに配置。

```
workshop202207/
└── tapyrus_api_client_cert_2022-07-26.p12
```

---

## 4. TapyrusAPI ラッパー仕様

クラス: `TapyrusApi` (`lib/utils/tapyrus_api.rb`)
パターン: Singleton

### 4.1 クラスメソッド一覧

#### アドレス

| メソッド | HTTP | エンドポイント | 説明 |
|---|---|---|---|
| `get_addresses(per:, page:, purpose:)` | GET | `/api/v1/addresses` | アドレス一覧取得 |
| `post_addresses(purpose:)` | POST | `/api/v1/addresses` | アドレス新規作成 |

**パラメータ**

`get_addresses`
- `per` (default: 25) — 1 ページあたりの件数
- `page` (default: 1) — ページ番号
- `purpose` (default: `"general"`) — アドレス用途

`post_addresses`
- `purpose` (default: `"general"`) — アドレス用途

#### トークン

| メソッド | HTTP | エンドポイント | 説明 |
|---|---|---|---|
| `get_tokens(confirmation_only)` | GET | `/api/v1/tokens` | 保有トークン一覧取得 |
| `post_tokens_issue(amount:, token_type:, split:)` | POST | `/api/v1/tokens/issue` | トークン新規発行 |
| `put_tokens_transfer(token_id, address:, amount:)` | PUT | `/api/v1/tokens/:token_id/transfer` | トークン送付 |
| `post_tokens_reissue(token_id, amount:, split:)` | POST | `/api/v1/tokens/:token_id/reissue` | トークン追加発行 |

**トークン種別 (`token_type`)**

| 値 | 種別 | 特徴 |
|---|---|---|
| 1 | 再発行可能トークン | 追加発行可能、総量可変 |
| 2 | 再発行不可能トークン | 追加発行不可、総量固定 |
| 3 | NFT | 発行数常に 1 |

**パラメータ**

`post_tokens_issue`
- `amount` (必須) — 発行量
- `token_type` (default: 1) — トークン種別
- `split` (default: 1) — UTXO 分割数

`put_tokens_transfer`
- `token_id` (必須) — 送付するトークンの ID
- `address` (必須) — 送付先アドレス
- `amount` (必須) — 送付量

`post_tokens_reissue`
- `token_id` (必須) — 追加発行するトークンの ID
- `amount` (必須) — 追加発行量
- `split` (default: 1) — UTXO 分割数

#### タイムスタンプ

| メソッド | HTTP | エンドポイント | 説明 |
|---|---|---|---|
| `get_timestamps` | GET | `/api/v1/timestamps` | タイムスタンプ一覧取得 |
| `get_timestamp(id)` | GET | `/api/v1/timestamps/:id` | タイムスタンプ詳細取得 |
| `post_timestamp(content:, digest:, prefix:, type:)` | POST | `/api/v1/timestamps` | タイムスタンプ作成 |

#### ユーザー情報

| メソッド | HTTP | エンドポイント | 説明 |
|---|---|---|---|
| `get_userinfo(confirmation_only)` | GET | `/api/v1/userinfo` | ウォレット残高・情報取得 |

---

## 5. Rake タスク仕様

名前空間: `api`

### 5.1 実装済みタスク

| タスク | 引数 | 説明 |
|---|---|---|
| `api:get_addresses[per,page,purpose]` | 省略可 | アドレス一覧表示 |
| `api:post_addresses[purpose]` | 省略可 | アドレス新規作成 |
| `api:get_tokens[confirmation_only]` | 省略可 | 保有トークン一覧表示 |
| `api:post_tokens_issue[amount,token_type,split]` | `amount` 必須 | トークン発行 |
| `api:put_tokens_transfer[token_id,address,amount]` | 全て必須 | トークン送付 |

### 5.2 未実装タスク (今後追加予定)

| タスク | 説明 |
|---|---|
| `api:get_timestamps` | タイムスタンプ一覧表示 |
| `api:post_timestamp[content,digest,prefix,type]` | タイムスタンプ作成 |
| `api:post_tokens_reissue[token_id,amount,split]` | トークン追加発行 |
| `api:get_userinfo` | ウォレット情報表示 |

---

## 6. Web UI 仕様 (未実装 → 今後追加)

現状 `config/routes.rb` は空。以下の画面を実装する。

### 6.1 画面一覧

| 画面 | URL | 機能 |
|---|---|---|
| ダッシュボード | `GET /` | ウォレット残高・概要表示 |
| アドレス一覧 | `GET /addresses` | アドレス一覧 (ページネーション) |
| アドレス作成 | `POST /addresses` | アドレス新規作成 |
| トークン一覧 | `GET /tokens` | 保有トークン一覧 |
| トークン発行 | `GET /tokens/new` / `POST /tokens/issue` | トークン発行フォーム |
| トークン送付 | `GET /tokens/:id/transfer` / `PUT /tokens/:id/transfer` | トークン送付フォーム |
| トークン追加発行 | `POST /tokens/:id/reissue` | 再発行フォーム |
| タイムスタンプ一覧 | `GET /timestamps` | タイムスタンプ一覧 |
| タイムスタンプ作成 | `GET /timestamps/new` / `POST /timestamps` | タイムスタンプ作成フォーム |

### 6.2 コントローラ構成 (予定)

```
app/controllers/
├── application_controller.rb
├── dashboard_controller.rb
├── addresses_controller.rb
├── tokens_controller.rb
└── timestamps_controller.rb
```

---

## 7. 既知の課題・改善点

### 7.1 TapyrusAPI ラッパー

| # | 問題 | 対応方針 |
|---|---|---|
| 1 | `client_cert` と `client_key` が PKCS12 を二重に読み込んでいる | `initialize` 内で一度だけ読み込むよう修正 |
| 2 | インデント崩れ (`post_addresses`, `post_tokens_issue` 等) | コード整形 |
| 3 | アクセストークンとエンドポイント URL がハードコードされている | 環境変数 (`ENV`) またはクレデンシャルへ移行 |
| 4 | エラーハンドリングが `FileNotFound` / `UrlNotFound` のみ | API 応答エラー (4xx/5xx) のハンドリング追加 |

### 7.2 Rake タスク

| # | 問題 | 対応方針 |
|---|---|---|
| 1 | タイムスタンプ・再発行・ユーザー情報タスクが未実装 | `api.rake` に追加 |
| 2 | `pp res` のみで出力が見づらい | 整形した出力に改善 |

### 7.3 Web UI

| # | 問題 | 対応方針 |
|---|---|---|
| 1 | ルーティング・コントローラ・ビューが未実装 | 6. の設計に従い実装 |

---

## 8. 起動手順

```bash
# DB 作成 (初回のみ)
docker compose run --rm web bin/rails db:create

# 起動
docker compose up --build

# 停止・削除
docker compose down -v --remove-orphans
```

---

## 9. TapyrusAPI 参考ドキュメント

- アドレス作成: `/address/operation/createAddress`
- トークン発行: `/token/operation/issueToken`
- トークン送付: `/token/operation/transferToken`
- トークン再発行: `/token/operation/reissueToken`
- トークン一覧: `/token/operation/getTokens`
