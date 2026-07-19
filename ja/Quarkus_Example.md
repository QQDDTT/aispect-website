# Quarkus 実践例

AiSpect は CDI（コンテキストと依存性の注入）を通じて Quarkus とシームレスに統合されます。`@Inject` を使用するだけで、アプリケーションに AI 機能を有効にすることができます。

この例（`examples/example-quarkus` ディレクトリにあります）は、Quarkus アプリケーションに AiSpect を統合する方法を示し、コードレビュー、フェーズテスト、およびデータ型テスト用のさまざまな REST API を公開しています。

---

## 1. 構成アーキテクチャ

この例に含まれるすべてのコードファイルとその目的の完全なリストは以下の通りです：

- **設定**
  - `AiSpectConfig.java`: `AiClient` と `AiOperations` Bean を生成する CDI 設定クラス。

    <details><summary>View <code>AiSpectConfig.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import com.aispect.api.AiClient;
    import com.aispect.api.AiOperations;
    import com.aispect.core.client.adapter.openai.OpenAiAdapter;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;
    import com.aispect.core.client.config.AiClientConfig;
    import com.aispect.common.exception.AiSpectException;
    import jakarta.enterprise.context.ApplicationScoped;
    import jakarta.enterprise.inject.Produces;

    import java.util.stream.Stream;

    /**
     * AiSpect コア CDI 構成クラス。
     * <p>
     * 2 つのコア Bean {@link AiClient} と {@link AiOperations} の生成を担当します。
     * {@code @Inject} 経由で {@link com.aispect.engine.quarkus.AiAgentCdiInterceptor} によって使用されます。
     * </p>
     * <p>
     * まず、{@code GEMINI_API_KEY} 環境変数を読み取ります。設定されていない場合、組み込みの Mock クライアントがデモンストレーションに使用されます。
     * </p>
     */
    @ApplicationScoped
    public class AiSpectConfig {

        /**
         * {@link AiClient} Bean を生成します。
         * <p>
         * {@code GEMINI_API_KEY} が設定されている場合は、{@link OpenAiAdapter} (Gemini OpenAI 互換インターフェイスと互換性のある) を使用します。
         * それ以外の場合は、組み込みの Mock 実装がデモンストレーションの目的で返されます。
         * </p>
         *
         * @return AiClient インスタンス
         */
        @Produces
        @ApplicationScoped
        public AiClient aiClient() {
            String apiKey = System.getenv("GEMINI_API_KEY");
            if (apiKey != null && !apiKey.trim().isEmpty()) {
                return new OpenAiAdapter();
            }
            // API キーが設定されていない場合は、モック クライアントが使用され、固定のデモ応答が返されます。
            return new AiClient() {
                @Override
                public Response generate(PromptContext ctx, AiClientConfig config) throws AiSpectException {
                    String content = "[Mock] AiSpect Quarkus Example — set GEMINI_API_KEY for real AI responses.";
                    return new Response(new Message(Role.ASSISTANT, content), new Response.Usage(0, 0, 0));
                }

                @Override
                public Stream<Response> streamGenerate(PromptContext ctx, AiClientConfig config) throws AiSpectException {
                    return Stream.of(generate(ctx, config));
                }
            };
        }

        /**
         * {@link AiOperations} Bean を生成します。
         * <p>
         * {@link AiClient} を標準の {@link AiOperations} インターフェースにラップします。
         * 実行時に大規模なモデルリクエストをディスパッチするためにインターセプタによって使用されます。
         * </p>
         *
         * @param aiClient CDI によって挿入された AiClient インスタンス
         * @return AiOperations インスタンス
         */
        @Produces
        @ApplicationScoped
        public AiOperations aiOperations(AiClient aiClient) {
            return new AiOperations() {
                @Override
                public AiClient getClient() {
                    return aiClient;
                }

                @Override
                public Response prompt(PromptContext ctx) throws AiSpectException {
                    return aiClient.generate(ctx, null);
                }

                @Override
                public Stream<Response> promptStream(PromptContext ctx) throws AiSpectException {
                    return aiClient.streamGenerate(ctx, null);
                }

                @Override
                public Response prompt(String modelName, String agentName, PromptContext ctx) throws AiSpectException {
                    AiClientConfig config = (modelName != null && !modelName.trim().isEmpty())
                            ? new AiClientConfig(null, null, modelName, null, null)
                            : null;
                    return aiClient.generate(ctx, config);
                }

                @Override
                public Stream<Response> promptStream(String modelName, String agentName, PromptContext ctx) throws AiSpectException {
                    AiClientConfig config = (modelName != null && !modelName.trim().isEmpty())
                            ? new AiClientConfig(null, null, modelName, null, null)
                            : null;
                    return aiClient.streamGenerate(ctx, config);
                }
            };
        }
    }
    ```

    </details>


- **コントローラー (Resources)**
  - `CodeReviewResource.java`: コードレビュー Webhook を処理するための REST エンドポイント。

    <details><summary>View <code>CodeReviewResource.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import jakarta.inject.Inject;
    import jakarta.ws.rs.Consumes;
    import jakarta.ws.rs.POST;
    import jakarta.ws.rs.Path;
    import jakarta.ws.rs.Produces;
    import jakarta.ws.rs.core.MediaType;
    import jakarta.ws.rs.core.Response;
    import java.util.Map;

    @Path("/webhook/code-review")
    public class CodeReviewResource {

        @Inject
        CodeReviewService codeReviewService;

        @POST
        @Consumes(MediaType.APPLICATION_JSON)
        @Produces(MediaType.APPLICATION_JSON)
        public Response handleWebhook(Map<String, Object> payload) {
            // In a real scenario, extract diff from GitHub/GitLab webhook payload
            String diff = (String) payload.getOrDefault("diff", "");
            if (diff.isEmpty()) {
                return Response.status(400).entity("{\"error\": \"No diff provided\"}").build();
            }

            try {
                String reviewResult = codeReviewService.reviewDiff(diff);
                return Response.ok(reviewResult).build();
            } catch (Exception e) {
                return Response.serverError().entity("{\"error\": \"" + e.getMessage() + "\"}").build();
            }
        }
    }
    ```

    </details>

  - `PhaseTestResource.java`: 異なるインターセプターフェーズをテストするための REST エンドポイント。

    <details><summary>View <code>PhaseTestResource.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import jakarta.inject.Inject;
    import jakarta.ws.rs.Consumes;
    import jakarta.ws.rs.GET;
    import jakarta.ws.rs.POST;
    import jakarta.ws.rs.Path;
    import jakarta.ws.rs.Produces;
    import jakarta.ws.rs.QueryParam;
    import jakarta.ws.rs.core.MediaType;

    @Path("/test/phase")
    public class PhaseTestResource {

        @Inject
        PhaseTestService phaseTestService;

        @GET
        @Path("/before")
        @Produces(MediaType.TEXT_PLAIN)
        public String phaseBefore(@QueryParam("input") String input) {
            if (input == null || input.isEmpty()) {
                input = "safe data";
            }
            return phaseTestService.testBeforePhase(input);
        }

        @GET
        @Path("/exception")
        @Produces(MediaType.TEXT_PLAIN)
        public String phaseException(@QueryParam("input") String input) {
            if (input == null || input.isEmpty()) {
                input = "error";
            }
            return phaseTestService.testExceptionPhase(input);
        }

        @POST
        @Path("/all")
        @Consumes(MediaType.TEXT_PLAIN)
        @Produces(MediaType.TEXT_PLAIN)
        public String phaseAll(String text) {
            return phaseTestService.testAllPhase(text);
        }
    }
    ```

    </details>

  - `DataTypeTestResource.java`: さまざまなデータ型をテストするための REST エンドポイント。

    <details><summary>View <code>DataTypeTestResource.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import jakarta.inject.Inject;
    import jakarta.ws.rs.Consumes;
    import jakarta.ws.rs.POST;
    import jakarta.ws.rs.Path;
    import jakarta.ws.rs.Produces;
    import jakarta.ws.rs.core.MediaType;

    @Path("/test/datatype")
    public class DataTypeTestResource {

        @Inject
        DataTypeTestService dataTypeTestService;

        @POST
        @Path("/string")
        @Consumes(MediaType.TEXT_PLAIN)
        @Produces(MediaType.TEXT_PLAIN)
        public String typeString(String text) {
            return dataTypeTestService.processString(text);
        }
    }
    ```

    </details>


- **サービス (Services)**
  - `CodeReviewService.java`: コードレビューのためのビジネスロジックと `@AiUnitAgent` 定義。

    <details><summary>View <code>CodeReviewService.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agents.data.DataCleanAgent;

    import jakarta.enterprise.context.ApplicationScoped;

    @ApplicationScoped
    public class CodeReviewService {

        @AiUnitAgent(value = DataCleanAgent.class, description = "Analyzes the code diff and returns a JSON summary of defects and suggestions.")
        public String reviewDiff(String diffContent) {
            // Fallback in case CDI interception is not set up correctly or no API key is provided
            return """
                   {
                     "status": "fallback",
                     "message": "No review performed. Received diff length: %d"
                   }""".formatted(diffContent.length());
        }
    }
    ```

    </details>

  - `PhaseTestService.java`: フェーズテストのためのビジネスロジックと `@AiUnitAgent` 定義。

    <details><summary>View <code>PhaseTestService.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agents.data.JsonExtractorAgent;
    import com.aispect.agents.flow.FallbackAgent;
    import com.aispect.agents.security.ContentModerationAgent;
    import com.aispect.engine.quarkus.AiAgentInterceptorBinding;

    import jakarta.enterprise.context.ApplicationScoped;

    @ApplicationScoped
    @AiAgentInterceptorBinding
    public class PhaseTestService {

        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "実行前にコンテンツコンプライアンスレビューを実施する")
        public String testBeforePhase(String input) {
            return "Original method executed successfully with input: " + input;
        }

        // In Quarkus, we reuse the basic flow agents. Note that we don't have SummaryAfterAgent in core agents,
        // so we'll just demonstrate the BEFORE, EXCEPTION, and ALL phases which use core agents.
        
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "例外が発生したときにダウングレードフォールバックメカニズムをトリガーする")
        public String testExceptionPhase(String input) {
            if ("error".equalsIgnoreCase(input)) {
                throw new RuntimeException("Simulated internal server error in testExceptionPhase");
            }
            return "No error occurred.";
        }

        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "ライフサイクルのすべての段階で JSON データを抽出および解析する")
        public String testAllPhase(String input) {
            return "This text should not be returned, because ALL phase agent handles it.";
        }
    }
    ```

    </details>

  - `DataTypeTestService.java`: データ型処理のためのビジネスロジックと `@AiUnitAgent` 定義。

    <details><summary>View <code>DataTypeTestService.java</code></summary>

    ```java
    package com.aispect.example.quarkus;

    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agents.data.DataCleanAgent;
    import com.aispect.agents.data.ImageProcessAgent;
    import com.aispect.engine.quarkus.AiAgentInterceptorBinding;

    import jakarta.enterprise.context.ApplicationScoped;

    @ApplicationScoped
    @AiAgentInterceptorBinding
    public class DataTypeTestService {

        @AiUnitAgent(value = DataCleanAgent.class, description = "クリーンな文字列データ", executePhase = AiExecutePhase.ALL)
        public String processString(String text) {
            return text;
        }

        @AiUnitAgent(value = ImageProcessAgent.class, description = "指示に従って画像を処理する", executePhase = AiExecutePhase.ALL)
        public byte[] processImage(byte[] image, String instruction) {
            return image;
        }
    }
    ```

    </details>


- **リソース**
  - `application.properties`: Quarkus アプリケーションの設定ファイル。

    <details><summary>View <code>application.properties</code></summary>

    ```properties
    quarkus.http.port=8081
    quarkus.log.category."com.aispect".level=DEBUG
    ```

    </details>


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
