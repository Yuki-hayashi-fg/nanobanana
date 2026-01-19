# Database Schema Documentation

このプロジェクトで使用しているデータベース構造（パブリックテーブル）の定義書です。
AIにコード修正を依頼する際は、この内容を前提情報として渡してください。

## 1. テーブル一覧

| テーブル名 | 用途 | 備考 |
| :--- | :--- | :--- |
| **users** | ユーザー管理 | ダミーメール運用によるユーザー名管理 |
| **templates** | プロンプトテンプレート | ユーザーが保存した定型文 |
| **generations** | 生成履歴 | 生成パラメータと参照画像を保存 |
| **generated_images** | 生成画像 | 生成結果の画像データを保存 |
| **api_keys** | APIキー管理 | 外部AIサービスのキー（暗号化必須） |

---

## 2. 詳細定義

### `users`
ユーザー名とパスワードによる独自ログイン（裏側でダミーメール認証）を管理するためのテーブル。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `uuid` | **PK**, `references auth.users(id)` | Supabase AuthのUIDと紐付け |
| `username` | `text` | `unique`, `not null` | ログイン用ユーザー名 |
| `password_hash`| `text` | `not null` | (アプリの実装依存) |
| `created_at` | `timestamptz`| `default now()` | 作成日時 |

---

### `templates`
ユーザーが作成したプロンプトのテンプレートを保存するテーブル。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `uuid` | **PK**, `default gen_random_uuid()` | |
| `user_id` | `uuid` | `not null`, `references users(id)` | 所有者 |
| `title` | `text` | | テンプレート名 |
| `content` | `text` | | プロンプト本文・変数定義(JSON) |
| `created_at` | `timestamptz`| `default now()` | 作成日時 |

---

### `generations`
画像生成リクエストの履歴。生成に使用したパラメータと参照画像を保存する。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `bigint` | **PK**, `auto_increment` | |
| `prompt` | `text` | `not null` | 生成に使用したプロンプト |
| `aspect_ratio` | `varchar(10)` | `not null` | 生成比率 |
| `resolution` | `varchar(10)` | `not null` | 生成解像度 |
| `temperature` | `decimal(3,2)` | `not null` | 生成温度 |
| `image_count` | `int` | `not null` | 生成枚数 |
| `reference_image_base64` | `longtext` | `nullable` | 参照画像のBase64 |
| `reference_image_mime` | `varchar(50)` | `nullable` | 参照画像のMIME |
| `created_at` | `timestamp` | `default CURRENT_TIMESTAMP` | 作成日時 |

---

### `generated_images`
生成された画像データを保存するテーブル。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `bigint` | **PK**, `auto_increment` | |
| `generation_id` | `bigint` | `not null`, `references generations(id)` | 生成履歴との紐付け |
| `image_base64` | `longtext` | `not null` | 生成画像のBase64 |
| `mime_type` | `varchar(50)` | `not null` | MIMEタイプ |
| `created_at` | `timestamp` | `default CURRENT_TIMESTAMP` | 作成日時 |

---

### `api_keys`
GeminiなどのAPIキーを保存するテーブル。**セキュリティのため、生のキーではなく暗号化された文字列を保存する。**

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `uuid` | **PK**, `default gen_random_uuid()` | |
| `user_id` | `uuid` | `not null`, `references users(id)` | 所有者 |
| `name` | `text` | | キーの名称（例: "Gemini Pro"） |
| `key` | `text` | `not null` | **暗号化済み**のAPIキー文字列 |
| `created_at` | `timestamptz`| `default now()` | 作成日時 |

---

## 3. 実装上の注意点 (AIへの指示用)

1.  **RLS (Row Level Security):**
    * 基本的に `user_id = auth.uid()` のデータのみ読み書きできるようにポリシーを設定すること。
2.  **APIキーの扱い:**
    * `api_keys.key` に保存する際は、必ずクライアントサイドで `CryptoJS` 等を用いて暗号化すること。
    * `key` カラムの値をそのままAPIリクエストに使用しないこと（復号プロセスを経ること）。
3.  **参照画像の保存:**
    * `generations.reference_image_base64` に保存する際、Base64本文のみを保存し、`data:` プレフィックスは含めないこと。
