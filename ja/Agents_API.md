# Agents API ドキュメント

`aispect-agents` モジュールでは、デフォルトで複数の事前構築済み AI エージェント（Agents）が提供されています。インターフェースに `@AiUnitAgent` アノテーションを使用するだけで、通常の Java メソッドをこれらの大規模言語モデルエージェントにマッピングできます。

---

## 📝 1. データのクリーンアップと拡張 (DataCleanAgent)

**完全修飾クラス名**: `com.aispect.agents.data.DataCleanAgent`

**機能説明**: 
テキストの処理、推敲、クリーンアップ、および拡張に特化しています。ユーザーの入力テキストを受け取り、より豊富で専門的な語彙を使用して詳細を補足し、短いテキストを深みのある段落に拡張します。

**パラメータ規約**:
- **パラメータ 1 (`String`)**: ユーザーが提供する元のテキスト。

**使用例**:
```java
@AiUnitAgent(
    name = DataCleanAgent.class, 
    description = "Expands the given text with more details and richer vocabulary."
)
String processText(String originalText);
```

---

## 🖼️ 2. 画像処理と認識 (ImageProcessAgent)

**完全修飾クラス名**: `com.aispect.agents.data.ImageProcessAgent`

**機能説明**: 
画像データを受信して理解することに特化したマルチモーダルエージェントです。特定の指示を付加することで、LLM（Gemini 1.5 Flash や Imagen など）に、アップロードされた画像に基づいた識別、分析、または修正の提案を行わせることができます。

**パラメータ規約**:
- **パラメータ 1 (`byte[]`)**: 画像のバイナリデータ（フレームワークによって自動的にマルチモーダルな `inlineData` に変換され、LLMに送信されます）。
- **パラメータ 2 (`String`)**: 画像に関するユーザーからの具体的な指示（例：「写真のニックをジュディに変更してください」）。

**使用例**:
```java
@AiUnitAgent(
    name = ImageProcessAgent.class, 
    description = "Processes an image based on instructions.", 
    modelName = "imagen-4-generate"
)
byte[] processImage(byte[] image, String instruction); // ビジネスロジックに応じてStringを返すよう設定することも可能
```

---

## 📖 3. 画像からのストーリー作成 (ImageStoryAgent)

**完全修飾クラス名**: `com.aispect.agents.data.ImageStoryAgent`

**機能説明**: 
こちらもマルチモーダルエージェントですが、内部のシステムプロンプトが「非常に創造的なストーリーテラー」としてカスタマイズされています。画像内の詳細、キャラクター、シーンを観察し、ユーザーが提供するプロンプトと組み合わせて、想像力豊かなストーリーを生成します。

**パラメータ規約**:
- **パラメータ 1 (`byte[]`)**: 画像のバイナリデータ。
- **パラメータ 2 (`String`)**: 追加のプロット要件またはプロンプト（空の場合、デフォルトで「画像に基づいてストーリーを語る」ことが要求されます）。

**使用例**:
```java
@AiUnitAgent(
    name = ImageStoryAgent.class, 
    description = "Tells an imaginative story based on the image.", 
    modelName = "gemini-2.5-flash"
)
String storyFromImage(byte[] image, String instruction);
```

---

## 🗂️ 4. JSON エクストラクタ (JsonExtractorAgent)

**完全修飾クラス名**: `com.aispect.agents.data.JsonExtractorAgent`

**機能説明**: 
非構造化された乱雑なテキストから JSON 文字列を抽出するために使用されます。元のメソッドの実行をインターセプトし、入力されたテキストを AI の解析に渡し、余分な Markdown タグ（```json など）を自動的にフィルタリングして、純粋な JSON 文字列のみを返します。

**パラメータ規約**:
- **パラメータ 1 (`String` または任意の `Object`)**: 乱雑なデータを含む元のテキスト。

**使用例**:
```java
@AiUnitAgent(
    name = JsonExtractorAgent.class, 
    description = "Extract JSON data from it"
)
String extractJson(String rawText);
```
