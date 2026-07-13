# Spring Boot 実践例

AiSpect は Spring Boot とシームレスに統合する機能を提供します。複雑な HTTP リクエストやプロンプト結合コードを記述することなく、Java インターフェースを定義するだけで大規模言語モデル（LLM）と対話できるようになります。

この例（`examples/example-spring-boot` ディレクトリにあります）では、「テキストの推敲」、「画像処理」、「画像からのストーリー作成」機能を備えた Web アプリケーションを迅速に構築する方法を示します。

---

## 1. アプリケーションアーキテクチャ

- **コントローラー (`DocumentController`)**: フロントエンドからのテキストやファイルアップロードのリクエストを処理する REST/Web エンドポイントを提供します。
- **規約インターフェース (`AiEditorService`)**: ビジネスロジックのコアです。実装クラスは不要で、すべてのメソッドに `@AiUnitAgent` アノテーションが付与され、実行時に AiSpect エンジンが動的にプロキシを作成し、対応するエージェントを呼び出します。
- **フロントエンドページ (`index.html`)**: シンプルな左右分割のインタラクティブ画面を提供します。左側で編集やアップロードを行い、右側で AI の結果をプレビューします。

---

AiSpect を使用する Spring Boot アプリケーションでは、特定のアノテーションを付与するだけで `Service` インターフェースに AI 機能を持たせることができます。さらに、フォールバック（Fallback）やローカル処理ロジックを提供するための実装クラスを記述することも可能です。

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import com.aispect.agents.data.ImageProcessAgent;
import com.aispect.agents.data.ImageStoryAgent;

public interface AiEditorService {

    // テキスト処理：DataCleanAgent へプロキシ
    @AiUnitAgent(name = DataCleanAgent.class, description = "Expands the given text with more details and richer vocabulary.")
    String processText(String originalText);
    
    // 画像分析：ImageProcessAgent へプロキシ
    @AiUnitAgent(name = ImageProcessAgent.class, description = "Processes an image based on instructions.", modelName="imagen-4-generate")
    byte[] processImage(byte[] image, String instruction);

    // 画像ストーリー：ImageStoryAgent へプロキシ
    @AiUnitAgent(name = ImageStoryAgent.class, description = "Tells an imaginative story based on the image.", modelName="gemini-2.5-flash")
    String storyFromImage(byte[] image, String instruction);
}
```

フレームワークの AOP インターセプター（`AgentInterceptor` など）は、`@AiUnitAgent` の付いたメソッドの実行を自動的にインターセプトします。呼び出し時にパラメータを自動的に抽出（例えば `byte[]` を LLM 用のマルチモーダル入力に変換）し、LLM との通信を実行します。例外が発生した場合やインターセプトが明示的に拒否された場合は、ローカルの `AiEditorServiceImpl` の実行へとグレースフルにダウングレード（フォールバック）します。

---

## 3. 実行と体験

### ステップ 1: API キーの設定
環境変数または起動パラメータを通じて、モデルの API キーを注入します：
```bash
export GEMINI_API_KEY="あなたの_API_KEY_をここに入力"
```

### ステップ 2: プロジェクトの起動
プロジェクトのルートディレクトリで Gradle コマンドを実行し、Spring Boot を起動します：
```bash
./gradlew :example-spring-boot:bootRun
```

### ステップ 3: ページへのアクセス
ブラウザを開き `http://localhost:8080` にアクセスします。左側のテキストボックスに文字を入力して **Process** をクリックするとテキストの推敲を体験でき、画像をアップロードして **Tell Story** をクリックするとマルチモーダルな画像・テキスト生成能力を体験できます。
