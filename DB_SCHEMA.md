# Database Schema Documentation

このドキュメントは、Gemini画像生成サービスで使用するデータベース定義をまとめたものです。
プロンプトテンプレート機能を追加したため、テンプレート用テーブルも含めています。

---

## 1. テーブル一覧

| テーブル名 | 用途 | 備考 |
| :--- | :--- | :--- |
| **generations** | 画像生成リクエスト | 生成パラメータと参照画像の保存 |
| **generated_images** | 生成画像 | generationsと紐付く生成結果 |
| **prompt_templates** | プロンプトテンプレート | 定型プロンプトの保存 |

---

## 2. DDL

```sql
CREATE TABLE generations (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    prompt TEXT NOT NULL,
    aspect_ratio VARCHAR(10) NOT NULL,
    resolution VARCHAR(10) NOT NULL,
    temperature DECIMAL(3,2) NOT NULL,
    image_count INT NOT NULL,
    reference_image_base64 LONGTEXT,
    reference_image_mime VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE generated_images (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    generation_id BIGINT UNSIGNED NOT NULL,
    image_base64 LONGTEXT NOT NULL,
    mime_type VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (generation_id) REFERENCES generations(id)
) ENGINE=InnoDB;

CREATE TABLE prompt_templates (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

---

## 3. テーブル詳細

### `generations`
画像生成リクエストの履歴を保存するテーブルです。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT UNSIGNED` | **PK**, auto increment | 生成リクエストID |
| `prompt` | `TEXT` | `not null` | 生成プロンプト |
| `aspect_ratio` | `VARCHAR(10)` | `not null` | アスペクト比 |
| `resolution` | `VARCHAR(10)` | `not null` | 解像度 |
| `temperature` | `DECIMAL(3,2)` | `not null` | 温度パラメータ |
| `image_count` | `INT` | `not null` | 生成枚数 |
| `reference_image_base64` | `LONGTEXT` | `nullable` | 参照画像（Base64） |
| `reference_image_mime` | `VARCHAR(50)` | `nullable` | 参照画像MIMEタイプ |
| `created_at` | `TIMESTAMP` | `default CURRENT_TIMESTAMP` | 作成日時 |

---

### `generated_images`
生成された画像データを保存するテーブルです。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT UNSIGNED` | **PK**, auto increment | 生成画像ID |
| `generation_id` | `BIGINT UNSIGNED` | `not null` | generationsへの外部キー |
| `image_base64` | `LONGTEXT` | `not null` | 画像データ（Base64） |
| `mime_type` | `VARCHAR(50)` | `not null` | MIMEタイプ |
| `created_at` | `TIMESTAMP` | `default CURRENT_TIMESTAMP` | 作成日時 |

---

### `prompt_templates`
プロンプトテンプレートを保存するテーブルです。

| カラム名 | データ型 | 制約 / デフォルト | 説明 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT UNSIGNED` | **PK**, auto increment | テンプレートID |
| `title` | `VARCHAR(200)` | `not null` | テンプレート名 |
| `content` | `TEXT` | `not null` | テンプレート本文 |
| `created_at` | `TIMESTAMP` | `default CURRENT_TIMESTAMP` | 作成日時 |
| `updated_at` | `TIMESTAMP` | `default CURRENT_TIMESTAMP` | 更新日時 |
