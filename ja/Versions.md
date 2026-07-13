# AiSpect ダウンロードセンター (Downloads)

AiSpect のコアコードは現在クローズドソースですが、コンパイル済みの JAR パッケージと Maven/Gradle リポジトリのアドレスを公開しています。

## 最新バージョンのダウンロード (v1.0.0-Beta)

プロジェクトのアーキテクチャに応じて、対応するアダプター JAR と依存関係をダウンロードしてください。

### コアパッケージ（必須）
最下層のアノテーション定義とコアな AI 通信インターセプト機能を提供します：
- 📦 `aispect-agents-1.0.0.jar` —— [ダウンロード](../downloads/aispect-agents-1.0.0.jar) (ゼロ依存宣言パッケージ)
- 📦 `aispect-core-1.0.0.jar` —— [ダウンロード](../downloads/aispect-core-1.0.0.jar) (基盤ネットワーク、LLM 通信、プロキシエンジンを含む)

### エンジンアダプター（いずれか1つを選択）
実行環境に応じて対応するアダプターパッケージをダウンロードしてください：
- 🍃 **Spring Boot 専用**: `aispect-engine-spring-1.0.0.jar` —— [ダウンロード](../downloads/aispect-engine-spring-1.0.0.jar)
- ☕ **Pure Java SE 専用**: `aispect-engine-se-1.0.0.jar` —— [ダウンロード](../downloads/aispect-engine-se-1.0.0.jar)

---

## Maven と Gradle の設定方法

以下の設定を使用して、AiSpect をプロジェクトに直接インポートできます。

### 🍎 Maven (`pom.xml`)

Spring Boot を使用している場合は、Engine と Agents パッケージをインポートするだけです：

```xml
<dependencies>
    <!-- AiSpect Spring Boot エンジンアダプター -->
    <dependency>
        <groupId>com.aispect</groupId>
        <artifactId>aispect-engine-spring</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
    
    <!-- 公式エージェントライブラリ -->
    <dependency>
        <groupId>com.aispect</groupId>
        <artifactId>aispect-agents</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

### 🐘 Gradle (`build.gradle`)

対応する Gradle の依存関係のインポート方法：

```groovy
dependencies {
    // AiSpect Spring Boot エンジンアダプター
    implementation 'com.aispect:aispect-engine-spring:1.0.0-SNAPSHOT'
    
    // 公式エージェントライブラリ
    implementation 'com.aispect:aispect-agents:1.0.0-SNAPSHOT'
}
```

---

## 外部依存関係について

AiSpect のコアアノテーション層は、外部への依存を完全にゼロに保っています。しかし、実行時にエンジンによって取得される `aispect-core` には、大規模言語モデルとの通信用のネットワーククライアントツール（OkHttp 3.x など）がカプセル化されています。Spring Boot を使用する場合、`aispect-engine-spring` がこれらのカスケード依存関係のバージョンを自動的に管理するため、手動で介入する必要はありません。
