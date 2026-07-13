# Spring Boot 実践例

AiSpect は Spring Boot とシームレスに統合する機能を提供します。複雑な HTTP リクエストやプロンプト結合コードを記述することなく、Java インターフェースを定義するだけで大規模言語モデル（LLM）と対話できるようになります。

この例（`examples/example-spring-boot` ディレクトリにあります）では、各種インターセプターの「フェーズ（Phase）」と「データ型（Data Type）」をテストする API ダッシュボードを構築する方法を示します。

---

## 1. アプリケーションアーキテクチャ

- **コントローラー (`TestController`)**: 各フェーズおよびデータ型テストの REST/Web エンドポイントを提供します。
- **規約インターフェース (`PhaseTestService` & `DataTypeTestService`)**: ビジネスロジックのコアです。`@AiUnitAgent` アノテーションを使用することで、リクエストが各エージェントへ自動的にプロキシされます。
- **フロントエンドページ (`index.html`)**: モダンなダークテーマの API ダッシュボードを提供します。左側のサイドバーでテストを選択し、右側でリクエストの設定と結果のプレビュー（JSON フォーマットや SSE ストリーミング対応）を行えます。

---

AiSpect を使用する Spring Boot アプリケーションでは、特定のアノテーションを付与するだけで `Service` インターフェースに AI 機能を持たせることができます。例えば、POJO やコレクションを処理する場合：

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.example.spring.agents.PojoProcessAgent;
import com.aispect.example.spring.dto.User;
import java.util.List;
import java.util.Map;

public interface DataTypeTestService {

    // POJO の処理：PojoProcessAgent へプロキシ
    @AiUnitAgent(name = PojoProcessAgent.class, description = "Updates the status of a user object")
    User processPojo(User user);
    
    // コレクションの処理
    @AiUnitAgent(description = "Categorizes a list of items into fruits and vegetables")
    Map<String, List<String>> processCollection(List<String> items);
    
    // ... その他のデータ型およびフェーズテスト
}
```

フレームワークの AOP インターセプターは、`@AiUnitAgent` の付いたメソッドの実行を自動的にインターセプトします。呼び出し時にパラメータを自動的に抽出し、LLM との通信を実行し、指定された厳密な型（POJO、List、Map、さらには Stream や byte[]）のオブジェクトを返します。

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
ブラウザを開き `http://localhost:8080` にアクセスします。新しい **AiSpect API Dashboard** が表示されます。サイドバーを使用してテストケースを切り替えます：
- **フェーズテスト (Phase Tests)**: Before、After、Exception の各フェーズにおけるカスタムロジックのインターセプトと介入を体験できます。
- **データ型テスト (Data Type Tests)**: 文字列、POJO (JSON)、コレクション、画像 (マルチモーダル)、ストリーム (SSE) など、さまざまなデータ型の LLM 処理と自動バインディング機能を確認できます。
