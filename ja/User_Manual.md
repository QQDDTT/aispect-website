# AiSpect ユーザーマニュアル (User Manual)

AiSpect へようこそ！このマニュアルを通じて、外部プロジェクトに AiSpect のインテリジェントな機能を統合して使用する方法を素早く学ぶことができます。

## 1. 概念の概要

AiSpect は「アノテーションベース」のエージェント・オーケストレーション・フレームワークです。
複雑な LLM 通信ロジックを自分で管理する必要はありません。対象となる関数に `@AiUnitAgent` アノテーションを追加するだけで、その関数が呼び出された際に対応するエージェントによってインターセプトされ、AI を活用してデータ処理、判断、結果の強化が行われます。

## 2. クイック統合 (Spring Boot の例)

### ステップ 1: 依存関係の JAR を追加
ダウンロードした `aispect-engine-spring` とその基盤となる依存パッケージをプロジェクトのクラスパスに配置します。
*(または、プライベート Maven リポジトリ経由でインポートします)*

### ステップ 2: LLM API キーの設定
`application.yml` に設定を追加します：
```yaml
aispect:
  provider: gemini
  default-model: gemini-2.5-flash
  api-key: "${GEMINI_API_KEY:sk-xxxxxx}"
```

### ステップ 3: ビジネスロジックとアノテーションの記述
ビジネスメソッドを定義し、`@AiUnitAgent` アノテーションを付与します。
```java
@Service
public class OrderService {

    // 実際に注文を生成する前に、AI に入力パラメータの妥当性や意図をチェックさせます
    @AiUnitAgent(name = OrderValidatorAgentImpl.class, description = "注文要件を検証する")
    public Order createOrder(String userRequirement) {
        // ... 実際のデータベース保存ロジック ...
        return new Order();
    }
}
```

### ステップ 4: エージェント規約の実装
対応する `agentClass` を実装します：
```java
@Component
public class OrderValidatorAgentImpl extends AbstractUnitAgent<Object, Order> {
    
    @Override
    public Order orchestrate(AiInvocationContext<Object> ctx) throws AiSpectException {
        // このロジックは元のメソッド実行時に呼び出されます。getAiOperations(ctx) を介してLLMと通信できます。
        // 例: args 内の userRequirement に高リスクの単語が含まれているか判断し、含まれている場合は例外をスローしてプロセスを中断する、
        // またはLLMを呼び出して特定の注文情報を生成し、直接返すことができます。
        return new Order();
    }
}
```

## 3. 高度な使い方
より複雑なエージェントを実装することで、`orchestrate` 内で外部ツール呼び出し（MCP）を実行したり、`@AiUnitAgent` を持つ複数のコンポーネントに対して DAG グラフ構造のオーケストレーション（近日公開）を行うことができます。
