# Nano Banana API (Gemini Image Generation) 仕様と実装ルール

## 1. モデル定義と選択
Nano Banana APIは、Geminiモデルを使用した画像生成機能の別称です。用途に応じて以下の2つのモデルを使い分けてください。

| 別称 | 正式モデル名 (`model`) | 特徴 | 推奨用途 |
| :--- | :--- | :--- | :--- |
| **Nano Banana** | `gemini-2.5-flash-image` | 高速、低レイテンシ | 通常の画像生成、速度優先のタスク |
| **Nano Banana Pro** | `gemini-3-pro-image-preview` | 高画質、高度な推論、4K対応 | プロ品質のアセット、複雑な指示、テキストレンダリング、画風変換 |

## 2. パラメータ設定ルール (厳守)

### 2.1 アスペクト比 (`aspect_ratio`)
リクエスト時の `generation_config.image_config.aspect_ratio` には、**以下の文字列のみ**指定可能です。これ以外の値（数値や空文字）は `400 INVALID_ARGUMENT` エラーとなります。

* `'1:1'` (デフォルト)
* `'16:9'`, `'9:16'`
* `'4:3'`, `'3:4'`
* `'2:3'`, `'3:2'`
* `'5:4'`, `'4:5'`
* `'21:9'`

### 2.2 解像度 (`image_size`) ※Proモデルのみ
`gemini-3-pro-image-preview` を使用する場合のみ指定可能です。**「K」は大文字**である必要があります。

* `'1K'` (デフォルト)
* `'2K'`
* `'4K'`
    * ※ `1k` (小文字) はエラーになります。

### 2.3 レスポンス形式 (`response_modalities`)
* デフォルト: `['TEXT', 'IMAGE']` (推奨)
    * 特にProモデルは「思考プロセス(TEXT)」と「生成画像(IMAGE)」の両方を返すため、この設定が必要です。
* 画像のみ: `['IMAGE']`

## 3. 入力仕様 (マルチモーダル)

### 3.1 参照画像の制限
* **Nano Banana (Flash):** 最大 **3枚** まで。
* **Nano Banana Pro:** 最大 **14枚** まで。
    * オブジェクト参照: 最大6枚
    * 人物参照（キャラクター維持）: 最大5枚

### 3.2 スタイル参照・画風変換
既存の画像のスタイルを適用する場合、プロンプトで明示的に指示する必要があります。
* **Payload構造:** `[{ text: "プロンプト..." }, { inline_data: { mime_type: "...", data: "..." } }]`
* **プロンプト例:** "Apply the artistic style, color palette, and texture of the provided reference image."

### 3.3 Google検索によるグラウンディング
Proモデルでは、最新情報（天気、株価など）に基づいた画像生成が可能です。
* 設定: `tools=[{"google_search": {}}]` をconfigに追加。
* 制限: 検索結果の画像自体は生成に使用されず、レスポンスからも除外されます。

## 4. レスポンス処理と「思考プロセス」

### 4.1 思考 (Thinking) - Proモデルのみ
Proモデルは複雑なプロンプトに対して「思考」を行い、最終画像の前に中間生成を行う場合があります。
* APIレスポンスの `part.text` に思考プロセスが含まれます。
* `part.thought = true` のフラグで識別可能です。
* **注意:** チャット（マルチターン）の場合、`thought_signature`（思考署名）を次のリクエスト履歴に含める必要があります（Google Gen AI SDKを使用していれば自動処理されます）。

### 4.2 透かし
すべての生成画像には **SynthID** の不可視透かしが自動的に埋め込まれます。

## 5. 禁止事項と制限

* **権利侵害:** 他人の権利を侵害する画像、欺瞞的、有害なコンテンツの生成は禁止です。
* **入力制限:** 動画・音声ファイルは画像生成の入力としてサポートされていません。
* **言語:** 最高のパフォーマンスを得るには、EN, JA (日本語) などの主要言語を使用してください。

## 6. 実装コード例 (JavaScript/JSON構成イメージ)

```javascript
// Proモデルで4K画像を生成する場合のConfig例
const config = {
  model: "gemini-3-pro-image-preview",
  generationConfig: {
    response_modalities: ["TEXT", "IMAGE"], // 思考テキストを受け取るため
    image_config: {
      aspect_ratio: "16:9", // 許可リスト内の文字列
      image_size: "4K"      // 大文字のK
    }
  }
};

// スタイル参照時のContents構成例
const contents = [
  {
    role: "user",
    parts: [
      { text: "Generate a futuristic city. Apply the style of this image." },
      {
        inline_data: {
          mime_type: "image/png",
          data: "BASE64_STRING..." // ヘッダーなしのBase64
        }
      }
    ]
  }
];
