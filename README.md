# nanobanana
# AI開発アシスタントへの指示書（プロジェクトルール）

あなたは現在、サンドボックス環境（CodePen風の環境）で動作する、Supabase連携のWebアプリケーションを開発しています。
以下の**「鉄の掟」**を常に遵守し、絶対に以前のバグを再発させないでください。

## 🚫 禁止事項（厳守）
1.  **localStorage / sessionStorage の使用禁止**
    * **理由:** サンドボックス環境のセキュリティ制限で即クラッシュします。
    * **代替策:** 変数による一時保存（MemoryStorage）のみを使用してください。
2.  **window.alert / window.confirm の使用禁止**
    * **理由:** ブラウザによりブロックされ、無視されます。
    * **代替策:** 自作の「トースト通知（showToast）」や「モーダルウィンドウ」を使用してください。
3.  **formの標準送信の禁止**
    * **理由:** 画面遷移がブロックされエラーになります。
    * **対策:** すべてのボタンクリックイベントで必ず `e.preventDefault()` を実行してください。
4.  **Supabase `service_role` キーの使用禁止**
    * **対策:** 必ず `anon` (public) キーを使用してください。
5.  **不適切な `upsert` の使用禁止**
    * **理由:** ID重複時の挙動制御が不安定なため。
    * **対策:** IDがある場合は `update`、ない場合は `insert` と明示的に分けて記述してください。

## 🛠 実装の前提情報

### 1. データベース構成
以下のテーブル構成を前提にコードを書いてください。
* **generations**: 生成リクエスト本体
* **generated_images**: 生成画像（generationsと紐付け）
* **prompt_templates**: プロンプトテンプレート

### 2. 技術スタック
* HTML / Tailwind CSS (CDN)
* JavaScript (Vanilla JS)
* Supabase JS Client (v2)
* CryptoJS (APIキー暗号化用)

### 3. 特別なロジック
* **Supabase初期化:** `persistSession: false`, `storage: MemoryStorage` オプションが必須。
* **画像生成 (4K対応):** 通常生成と4K生成は、同じキューシステム (`generationQueue`) で管理し、並列処理による競合を防ぐこと。
* **スタイル参照:** 画像生成時はマルチモーダル入力（テキスト＋画像）に対応すること。

## ⚠️ コード生成時のチェックリスト
コードを出力する前に、以下を自問自答してください。
* ボタンのイベントリスナー（`.addEventListener`）は消えていないか？
* `localStorage` という単語が含まれていないか？
* 4K生成中や保存処理中に、ユーザーへのフィードバック（Loading表示など）があるか？
