# Quarkus 実践例

AiSpect は CDI（コンテキストと依存性の注入）を通じて Quarkus とシームレスに統合されます。`@Inject` を使用するだけで、アプリケーションに AI 機能を有効にすることができます。

この例（`examples/example-quarkus` ディレクトリにあります）は、Quarkus アプリケーションに AiSpect を統合する方法を示し、コードレビュー、フェーズテスト、およびデータ型テスト用のさまざまな REST API を公開しています。

---

## 1. 構成アーキテクチャ

この例に含まれるすべてのコードファイルとその目的の完全なリストは以下の通りです：

- **設定**
  - `AiSpectConfig.java`: `AiClient` と `AiOperations` Bean を生成する CDI 設定クラス。

- **コントローラー (Resources)**
  - `CodeReviewResource.java`: コードレビュー Webhook を処理するための REST エンドポイント。
  - `PhaseTestResource.java`: 異なるインターセプターフェーズをテストするための REST エンドポイント。
  - `DataTypeTestResource.java`: さまざまなデータ型をテストするための REST エンドポイント。

- **サービス (Services)**
  - `CodeReviewService.java`: コードレビューのためのビジネスロジックと `@AiUnitAgent` 定義。
  - `PhaseTestService.java`: フェーズテストのためのビジネスロジックと `@AiUnitAgent` 定義。
  - `DataTypeTestService.java`: データ型処理のためのビジネスロジックと `@AiUnitAgent` 定義。

- **リソース**
  - `application.properties`: Quarkus アプリケーションの設定ファイル。

---

## 2. エンジンの設定

Quarkus では、CDI 構成クラスを介して `AiClient` および `AiOperations` Bean を提供します。

```java
import com.aispect.api.AiClient;
import com.aispect.api.AiOperations;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Produces;

@ApplicationScoped
public class AiSpectConfig {

    // AiClient Bean を生成する
    @Produces
    @ApplicationScoped
    public AiClient aiClient() {
        return new OpenAiAdapter();
    }
    
    // AiOperations Bean を生成する
    @Produces
    @ApplicationScoped
    public AiOperations aiOperations(AiClient aiClient) {
        // インターセプターのために AiClient を AiOperations にラップする
        return new AiOperationsWrapper(aiClient);
    }
}
```

## 3. アノテーションの使用

CDI Bean のメソッドに `@AiUnitAgent` をアノテーションするだけです。Quarkus インターセプターが自動的に AI ロジックを処理します。

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class CodeReviewService {

    // コードの差分を分析し、JSON 形式の要約を返す
    @AiUnitAgent(value = DataCleanAgent.class, description = "コードの差分を分析し、欠陥と提案の JSON 要約を返します。")
    public String reviewDiff(String diffContent) {
        // AI 処理が失敗した場合のフォールバックロジック
        return "{ \"status\": \"fallback\" }";
    }
}
```

## 4. リソースへの注入

その後、標準の JAX-RS を使用してこのサービスを REST エンドポイントに注入できます。

```java
import jakarta.inject.Inject;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;

@Path("/webhook/code-review")
public class CodeReviewResource {

    @Inject
    CodeReviewService codeReviewService;

    @POST
    public String handleWebhook(String diff) {
        // 呼び出しは自動的に AiSpect によってインターセプトされます
        return codeReviewService.reviewDiff(diff);
    }
}
```

---

## 5. 実行体験

### ステップ 1: API キーの設定
API キーを環境変数として設定します：
```bash
export GEMINI_API_KEY="あなたの_API_KEY_をここに"
```

### ステップ 2: Quarkus を開発モードで起動
ルートディレクトリで Gradle コマンドを実行します：
```bash
./gradlew :example-quarkus:quarkusDev
```
