# Database Schema Documentation (Supabase)

このプロジェクトで使用しているSupabaseのデータベース構造（パブリックテーブル）の定義書です。
AIにコード修正を依頼する際は、この内容を前提情報として渡してください。

## 1. テーブル一覧

| テーブル名 | 用途 | 備考 |
| :--- | :--- | :--- |
| **users** | ユーザー管理 | ダミーメール運用によるユーザー名管理 |
| **templates** | プロンプトテンプレート | ユーザーが保存した定型文 |
| **image_history** | 生成履歴 | 生成結果のURLと参照画像を保存 |
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
| `content` | `text` | | プロンプト本文 |
| `created_at` | `timestamptz`| `default now()` | 作成日時 |

---

### `image_history`
画像生成の履歴ログ。生成された画像のURLと、**生成に使用した参照画像（スタイル画）**を保存する。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `uuid` | **PK**, `default gen_random_uuid()` | |
| `user_id` | `uuid` | `not null`, `references users(id)` | 所有者 |
| `prompt` | `text` | | 生成に使用したプロンプト |
| `image_url` | `text` | `not null` | 生成された画像のURL (期限付き) |
| `reference_image`| `text` | **(重要)**, `nullable` | スタイル参照元の画像データ (Base64) |
| `created_at` | `timestamptz`| `default now()` | 生成日時 |

> **注意:** `reference_image` はBase64テキストデータのため容量が大きいです。不要な場合はNULLを許容しています。

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
    * `image_history` に保存する際、`image_url` は生成結果（Googleサーバー）、`reference_image` は入力画像（ローカルアップロード）であることを混同しないこと。
