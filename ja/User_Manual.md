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

## 3. 高度な使い方: グラフエージェントとリソースノードのオーケストレーション

より複雑なビジネス分散のために、単一の Unit Agent を超えて、グラフエージェントを使用してクラス全体のワークフローを引き継ぐことができます。

### コアアノテーション
- **`@AiGraphAgent`**: **クラスレベル**に付与され、複雑なワークフローのグローバルなエントリポイントとして機能します。
- **`@AiNode`**: **メソッドレベル**に付与され、LLMが自発的にスケジュールできるサブ機能ノード（Tool/Skillに似ています）として機能します。

### 厳密なトラブルシューティングと検証メカニズム
プロセスオーケストレーションの厳密性を保証するために、AiSpectエンジンは起動フェーズ（Bootstrap）に**全量エラー収集**機能を導入しています。スキャナーは最初の設定エラーに遭遇してもショートサーキットして終了することはなく、現在の関数に関するすべての設定の欠陥を収集し、一度にすべてのエラーログを出力します。

**開発者は以下のトラブルシューティング制約を厳守する必要があります。そうしない場合、アプリケーションの起動はフレームワークによって安全にインターセプトされます。**
1. **強力な関連付けの制約**: `@AiNode` が付与されたメソッドのホストクラスは、`@AiGraphAgent` が付与されていなければなりません。フレームワークは、明確なエントリポイントの所属がない「孤児ノード」のロードを拒否します。
2. **フェーズ排他の制約**: 同じメソッド上で複数のエージェントアノテーション（`@AiUnitAgent` または `@AiNode`）を使用する場合、それらは異なる実行フェーズ（`executePhase`）を持たなければなりません。さらに、`ALL` フェーズと `BEFORE`/`AFTER` などの特定のフェーズを混在させることは固く禁じられています。
