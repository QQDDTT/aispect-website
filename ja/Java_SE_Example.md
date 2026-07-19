# Java SE 実践例

AiSpect は、Spring や Quarkus のような依存性の注入（DI）コンテナがなくても、純粋な Java SE 環境で使用できます。エンジンを手動で初期化し、標準の Java インターフェースのプロキシを作成できます。

この例（`examples/example-java-se` ディレクトリにあります）は、フレームワークを手動で初期化する方法を示し、「フェーズテスト」と「データ型テスト」をカバーしています。

---

## 1. 構成アーキテクチャ

この例に含まれるすべてのコードファイルとその目的の完全なリストは以下の通りです：

- **アプリケーションエントリとコアロジック**
  - `Main.java`: 単一のファイルに完全な例が含まれています。これには、`PhaseTestService` と `DataTypeTestService` インターフェース、そのスタブ実装、`SummaryAfterAgent` や `CollectionClassifierAgent` などのカスタムエージェント、API キーなしのデモンストレーション用の `MockAiClient`、および `AiSpect` エンジンを手動で構築してプロキシを作成する `main` メソッドが含まれます。

- **リソース**
  - `simplelogger.properties`: SLF4J simple logger の設定ファイル。

---

## 2. エンジンの初期化

純粋な Java SE 環境では、`AiSpect.builder()` を使用してエンジンを構築し、次に `proxy` メソッドを使用してインターフェースのプロキシを作成します。

```java
import com.aispect.api.AiClient;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import com.aispect.engine.se.AiSpect;

public class Main {
    public static void main(String[] args) throws Exception {
        // 1. AI クライアントを初期化する
        AiClient aiClient = new OpenAiAdapter();
        
        // 2. AiSpect エンジンを構築する
        AiSpect engine = AiSpect.builder()
                .aiClient(aiClient)
                .build();
                
        // 3. サービスのプロキシを作成する
        PhaseTestService rawService = new PhaseTestServiceImpl();
        PhaseTestService proxiedService = engine.proxy(rawService, PhaseTestService.class);
        
        // 4. プロキシ化されたメソッドを呼び出す
        String result = proxiedService.testBeforePhase("safe data");
        System.out.println(result);
    }
}
```

## 2. アノテーションの使用

Spring や Quarkus と同様に、インターフェースのメソッドに `@AiUnitAgent` アノテーションを使用して、AI の動作を定義できます。

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.security.ContentModerationAgent;

public interface PhaseTestService {

    // 実行前にコンテンツのコンプライアンスチェックを行う
    @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "実行前にコンテンツのコンプライアンスチェックを行う")
    String testBeforePhase(String input);
}
```

---

## 3. 実行体験

### ステップ 1: API キーの設定
API キーを環境変数として設定します：
```bash
export GEMINI_API_KEY="あなたの_API_KEY_をここに"
```

### ステップ 2: アプリケーションの実行
ルートディレクトリで Gradle コマンドを実行します：
```bash
./gradlew :example-java-se:run
```


## 完全なソースコード

### `java/com/aispect/example/se/Main.java`

```java
package com.aispect.example.se;

import java.util.Collections;
import java.util.List;
import java.util.Map;

import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.data.DataCleanAgent;
import com.aispect.agents.data.ImageProcessAgent;
import com.aispect.agents.flow.FallbackAgent;
import com.aispect.agents.data.JsonExtractorAgent;
import com.aispect.agents.security.ContentModerationAgent;
import com.aispect.api.AiClient;
import com.aispect.common.exception.AiSpectException;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import com.aispect.core.client.config.AiClientConfig;
import com.aispect.core.client.model.Message;
import com.aispect.core.client.model.PromptContext;
import com.aispect.core.client.model.Response;
import com.aispect.core.client.model.Role;
import com.aispect.core.engine.bootstrap.proxy.ProxyFactory;
import com.aispect.engine.se.AiSpect;

/**
 * Java SE サンプル アプリケーションのエントリ。
 * <p>
 * 純粋な Java SE 環境 (Spring / Quarkus コンテナなし) で AiSpect フレームワークを使用する方法を示します。
 * {@link AiSpect#proxy} メソッドを使用して、{@link AiSpect#builder()} 経由でエンジンを手動で初期化します。
 * 共通インターフェースのプロキシを作成し、{@code @AiUnitAgent} のアノテーションが付けられたメソッドを自動的にインターセプトします。
 * </p>
 *
 * <p>この例では、次の機能シナリオを取り上げます。</p>
 * <ol>
 *   <li>フェーズ テスト: BEFORE、AFTER、EXCEPTION、ALL の 4 つの実行ステージ</li>
 *   <li>データ型のテスト: 文字列のクリーニング、コレクションの分類、画像処理</li>
 * </ol>
 */
public class Main {

    public static void main(String[] args) throws Exception {
        printBanner();

        // 環境変数でAPIキーを取得します。
        String apiKey = System.getenv("GEMINI_API_KEY");
        if (apiKey == null || apiKey.trim().isEmpty()) {
            System.err.println("WARNING: GEMINI_API_KEY environment variable is not set.");
            System.err.println("The AI client will return mock responses for demonstration.\n");
        }

        // API キーが構成されているかどうかに応じて、実際の OpenAI/Gemini アダプターを使用するか、モック クライアントを使用するかを選択します。
        AiClient aiClient = (apiKey != null && !apiKey.trim().isEmpty())
                ? new OpenAiAdapter()
                : new MockAiClient();

        // AiSpect エンジンを構築し、AI クライアントを注入する
        AiSpect engine = AiSpect.builder()
                .aiClient(aiClient)
                .build();

        // ================================================================
        // シナリオ 1: フェーズ テスト
        // ================================================================
        System.out.println("=== シナリオ 1: フェーズ テスト ===\n");

        PhaseTestService rawPhaseService = new PhaseTestServiceImpl();
        PhaseTestService phaseService = engine.proxy(rawPhaseService, PhaseTestService.class);

        // BEFORE フェーズ: コンテンツ コンプライアンス レビュー
        System.out.println("[前フェーズ]入力:\"safe data\"");
        String beforeResult = phaseService.testBeforePhase("safe data");
        System.out.println("[前フェーズ] 結果:" + beforeResult + "\n");

        // AFTER段階：AIがメソッドの戻り値を自動集計
        System.out.println("[アフターフェーズ]入力：\"dummy\"");
        String afterResult = phaseService.testAfterPhase("dummy");
        System.out.println("[アフターフェーズ] 結果:" + afterResult + "\n");

        // EXCEPTION 段階：異常劣化
        System.out.println("[例外フェーズ] 入力: \"error\" (触发异常降级)");
        String exceptionResult = phaseService.testExceptionPhase("error");
        System.out.println("[例外フェーズ] 結果:" + exceptionResult + "\n");

        // すべての段階: AI が完全に引き継ぎ、テキストから JSON を抽出します
        System.out.println("[全相] 入力：\"用户名: Alice, 年龄: 25, 城市: 北京\"");
        String allResult = phaseService.testAllPhase("ユーザー名: アリス、年齢: 25、都市: 北京");
        System.out.println("[全フェーズ] 結果:" + allResult + "\n");

        // ================================================================
        // シナリオ 2: データ型のテスト
        // ================================================================
        System.out.println("=== シナリオ 2: データ型のテスト ===\n");

        DataTypeTestService rawDataService = new DataTypeTestServiceImpl();
        DataTypeTestService dataService = engine.proxy(rawDataService, DataTypeTestService.class);

        // 弦のクリーニング
        System.out.println("[データ型文字列] 入力: \"  Hello   World!!  \"");
        String cleanedStr = dataService.processString("  Hello   World!!  ");
        System.out.println("[データ型文字列] 結果:" + cleanedStr + "\n");

        // コレクションの分類
        List<String> items = List.of("りんご", "香蕉", "小轿车", "篮球", "橙子", "卡车", "足球", "葡萄");
        System.out.println("[データ型コレクション]入力:" + items);
        Map<String, List<String>> categorized = dataService.processCollection(items);
        System.out.println("[DataType コレクション] 結果:" + categorized + "\n");

        System.out.println("=== サンプルデモ完了 ===");
    }

    // ====================================================================
    // PhaseTestService インターフェースと実装
    // ====================================================================

    /**
     * ステージテストサービスインターフェース。
     * <p>
     * {@link AiUnitAgent} アノテーションを使用して、さまざまな実行段階で AiSpect フレームワークをデモンストレーションします
     * (BEFORE、AFTER、EXCEPTION、ALL) インターセプト プロキシ機能。
     * </p>
     */
    public interface PhaseTestService {

        /**
         * テスト前段階 (BEFORE): コンテンツのコンプライアンス レビュー。
         * 入力テキストは、ローカル メソッドが実行される前に、{@link ContentModerationAgent} によって安全にフィルタリングされます。
         *
         * @param input レビュー対象の入力テキスト
         * @return ネイティブビジネスロジック処理によって生成された結果
         */
        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "実行前にコンテンツコンプライアンスレビューを実施する")
        String testBeforePhase(String input);

        /**
         * テスト後フェーズ (AFTER): AI は返された結果を要約します。
         * ネイティブ メソッドが最初に実行され、その戻り値は {@link SummaryAfterAgent} によってインテリジェントに調整されます。
         *
         * @param input テスト入力テキスト
         * @return 大きなモデルによって要約されたテキスト
         */
        @AiUnitAgent(value = SummaryAfterAgent.class, executePhase = AiExecutePhase.AFTER, description = "実行後、生成されたテキストを要約して抽出します。")
        String testAfterPhase(String input);

        /**
         * テスト例外劣化ステージ (EXCEPTION): 例外災害復旧。
         * ネイティブ メソッドが例外をスローすると、{@link FallbackAgent} が自動的に引き継ぎ、フォールバック結果を返します。
         *
         * @param input 例外をトリガーするテスト入力 (例外をトリガーするには「error」を渡します)
         * @return ダウングレード結果テキスト
         */
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "例外が発生したときにダウングレードフォールバックメカニズムをトリガーする")
        String testExceptionPhase(String input);

        /**
         * 完全なテイクオーバー フェーズ (ALL) のテスト: AI はローカル メソッドを完全に置き換えます。
         * {@link JsonExtractorAgent} は、入力テキストから直接 JSON を抽出して返します。ネイティブ メソッドはスキップされます。
         *
         * @param input 基礎となる JSON 情報を含む入力テキスト
         * @return 大規模モデルから抽出されたフォーマットされた JSON 文字列
         */
        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "ライフサイクルのすべての段階で JSON データを抽出および解析する")
        String testAllPhase(String input);
    }

    /**
     * PhaseTestService のローカル ネイティブ実装 (スタブ実装)。
     */
    public static class PhaseTestServiceImpl implements PhaseTestService {

        @Override
        public String testBeforePhase(String input) {
            return "Original method executed successfully with input: " + input;
        }

        @Override
        public String testAfterPhase(String input) {
            // AIがこのメソッドの戻り値をインターセプトして要約します
            return "The quick brown fox jumps over the lazy dog. This is a very long and detailed sentence that needs to be summarized by the AI agent in the AFTER phase.";
        }

        @Override
        public String testExceptionPhase(String input) {
            // シミュレーション実行時例外
            if ("error".equalsIgnoreCase(input)) {
                throw new RuntimeException("Simulated internal server error in testExceptionPhase");
            }
            return "No error occurred.";
        }

        @Override
        public String testAllPhase(String input) {
            // AI の ALL フェーズが直接引き継ぎ、ここでは実行されません。
            return "This text should not be returned, because ALL phase agent handles it.";
        }
    }

    // ====================================================================
    // DataTypeTestService インターフェイスと実装
    // ====================================================================

    /**
     * データ型テスト サービス インターフェイス。
     * <p>
     * AiSpect フレームワークがさまざまなデータ型 (文字列、コレクション、画像) を処理できることを示します。
     * </p>
     */
    public interface DataTypeTestService {

        /**
         * 入力テキスト文字列をクリーンアップしてフォーマットします。
         * {@link DataCleanAgent} はすべてのステージを引き継ぎ、インテリジェントなクリーニングのために元のテキストを大規模モデルに渡します。
         *
         * @param text クリーンアップする元のテキスト
         * @return クリーンアップ、エラー修正、またはフォーマットされた文字列
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "クリーンな文字列データ", executePhase = AiExecutePhase.ALL)
        String processString(String text);

        /**
         * 収集データ（文字列のリスト）をAI分類します。
         * {@link CollectionClassifierAgent} は、ALL ステージを介してリスト項目をカテゴリにグループ化します。
         *
         * @param items 分類する文字列のリスト
         * @return カテゴリ別にグループ化されたマップ
         */
        @AiUnitAgent(value = CollectionClassifierAgent.class, description = "収集データの分類", executePhase = AiExecutePhase.ALL)
        Map<String, List<String>> processCollection(List<String> items);

        /**
         * 指示に従って画像のマルチモーダル処理を実行します。
         * {@link ImageProcessAgent} は ALL ステージを引き継ぎ、画像バイト配列を直接処理します。
         *
         * @param 画像の元の画像のバイナリ バイト配列
         * @param命令 画像処理命令
         * @return 処理された画像のバイナリ バイト配列
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "指示に従って画像を処理する", executePhase = AiExecutePhase.ALL)
        byte[] processImage(byte[] image, String instruction);
    }

    /**
     * DataTypeTestService のローカル ネイティブ実装 (スタブ実装/ダウングレード ロジック)。
     */
    public static class DataTypeTestServiceImpl implements DataTypeTestService {

        @Override
        public String processString(String text) {
            return text;
        }

        @Override
        public Map<String, List<String>> processCollection(List<String> items) {
            return Collections.emptyMap();
        }

        @Override
        public byte[] processImage(byte[] image, String instruction) {
            return image;
        }
    }

    // ====================================================================
    // カスタムエージェントの実装
    // ====================================================================

    /**
     * ポストサマリー エージェントは、ターゲット メソッドの実行後に大規模モデルを呼び出して結果を要約します。
     */
    public static class SummaryAfterAgent extends com.aispect.agent.core.AbstractUnitAgent<Object, String> {

        @Override
        public String postExecute(com.aispect.api.AiInvocationContext<Object> ctx, String result) throws AiSpectException {
            String input = result != null ? result : "";
            String systemPrompt = "You are a summarizing assistant. Summarize the user's text into a concise, professional sentence.";
            Message userMsg = new Message(Role.USER, input);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));
            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }

    /**
     * 大規模なモデルを通じて受信したリスト項目を分類し、JSON マッピング結果を返す分類エージェントのコレクション。
     */
    public static class CollectionClassifierAgent extends com.aispect.agent.core.AbstractUnitAgent<Object, Map<String, List<String>>> {

        private static final com.fasterxml.jackson.databind.ObjectMapper mapper = new com.fasterxml.jackson.databind.ObjectMapper();

        @Override
        public Map<String, List<String>> execute(com.aispect.api.AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            String input = (args != null && args.length > 0) ? args[0].toString() : "[]";

            String systemPrompt = "You are a categorization assistant. Group the following list of items into categories. Output ONLY valid JSON mapping category name (String) to a list of items (List<String>), no markdown.";
            Message userMsg = new Message(Role.USER, input);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            String result = response.message().content().replace("```json", "").replace("```", "").trim();

            try {
                return mapper.readValue(result, new com.fasterxml.jackson.core.type.TypeReference<Map<String, List<String>>>() {});
            } catch (Exception e) {
                return Collections.emptyMap();
            }
        }
    }

    // ====================================================================
    // モック AI クライアント (API キーがない場合に使用)
    // ====================================================================

    /**
     * シミュレートされた AI クライアント。API キーが提供されていない場合のデモンストレーションとダウングレードに使用されます。
     */
    public static class MockAiClient implements AiClient {

        @Override
        public Response generate(PromptContext ctx, AiClientConfig config) throws AiSpectException {
            String lastUserMsg = "";
            if (ctx != null && ctx.messages() != null && !ctx.messages().isEmpty()) {
                lastUserMsg = ctx.messages().get(ctx.messages().size() - 1).content();
            }
            String content = "[Mock Response] Received: " + (lastUserMsg != null ? lastUserMsg.substring(0, Math.min(50, lastUserMsg.length())) : "");
            return new Response(new Message(Role.ASSISTANT, content), new Response.Usage(0, 0, 0));
        }

        @Override
        public java.util.stream.Stream<Response> streamGenerate(PromptContext ctx, AiClientConfig config) throws AiSpectException {
            return java.util.stream.Stream.of(generate(ctx, config));
        }
    }

    // ====================================================================
    // ツール方式
    // ====================================================================

    private static void printBanner() {
        System.out.println("=====================================================");
        System.out.println("  AiSpect Java SE Example — Phase & DataType Demo  ");
        System.out.println("=====================================================\n");
    }
}

```

### `resources/simplelogger.properties`

```properties
org.slf4j.simpleLogger.defaultLogLevel=debug
org.slf4j.simpleLogger.log.com.aispect=debug
org.slf4j.simpleLogger.showDateTime=true
org.slf4j.simpleLogger.dateTimeFormat=yyyy-MM-dd HH:mm:ss:SSS Z

```

