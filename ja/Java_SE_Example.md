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
