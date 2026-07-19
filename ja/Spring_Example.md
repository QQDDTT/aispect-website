# Spring Boot 実践例

AiSpect は Spring Boot とシームレスに統合する機能を提供します。複雑な HTTP リクエストやプロンプト結合コードを記述することなく、Java インターフェースを定義するだけで大規模言語モデル（LLM）と対話できるようになります。

この例（`examples/example-spring-boot` ディレクトリにあります）では、各種インターセプターの「フェーズ（Phase）」と「データ型（Data Type）」をテストする API ダッシュボードを構築する方法を示します。

---

## 1. 構成アーキテクチャ

この例に含まれるすべてのコードファイルとその目的の完全なリストは以下の通りです：

- **アプリケーションエントリ**
  - `Application.java`: Spring Boot の起動クラス。

    ```java
    package com.aispect.example.spring;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    import com.aispect.engine.spring.annotation.EnableAiSpect;

    /**
     * Spring Boot アプリケーション起動クラス
     * @EnableAiSpect アノテーションを含めて AiSpect エンジンを有効にします
     */
    @SpringBootApplication
    @EnableAiSpect
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```

- **コントローラー (Controllers)**
  - `TestController.java`: すべてのフェーズ (Phase)、データ型 (Data Type)、およびグラフ (Graph) のテストインターフェースを含む REST/Web エンドポイントを提供します。

    ```java
    package com.aispect.example.spring.controller;

    import java.util.List;
    import java.util.Map;

    import org.springframework.http.MediaType;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.multipart.MultipartFile;

    import com.aispect.example.spring.dto.User;
    import com.aispect.example.spring.service.DataTypeTestService;
    import com.aispect.example.spring.service.PhaseTestService;

    /**
     * さまざまな AiSpect 機能のテスト インターフェイスを提供するテスト コントローラー
     */
    @RestController
    @RequestMapping("/test")
    public class TestController {

        private final PhaseTestService phaseTestService;
        private final DataTypeTestService dataTypeTestService;

        public TestController(PhaseTestService phaseTestService, DataTypeTestService dataTypeTestService) {
            this.phaseTestService = phaseTestService;
            this.dataTypeTestService = dataTypeTestService;
        }

        // -- Phase Tests --

        /**
         * テスト前フェーズ (Before Phase)
         * 
         * @param input 入力文字列
         * @return 前処理された文字列
         */
        @GetMapping("/phase/before")
        public String phaseBefore(@RequestParam(defaultValue = "safe data") String input) {
            return phaseTestService.testBeforePhase(input);
        }

        /**
         * アフターフェーズ
         * 
         * @param input 入力文字列
         * @return ポストサマリー処理済み文字列
         */
        @GetMapping("/phase/after")
        public String phaseAfter(@RequestParam(defaultValue = "dummy") String input) {
            return phaseTestService.testAfterPhase(input);
        }

        /**
         * テスト例外フェーズ (Exception Phase)
         * 
         * @param input 入力文字列
         * @return 例外ダウングレードまたは通常処理の結果
         */
        @GetMapping("/phase/exception")
        public String phaseException(@RequestParam(defaultValue = "error") String input) {
            return phaseTestService.testExceptionPhase(input);
        }

        /**
         * ライフサイクル全段階のテスト (全フェーズ)
         * 
         * @param text 入力テキスト
         * @return ライフサイクル全体の処理結果
         */
        @PostMapping("/phase/all")
        public String phaseAll(@RequestBody String text) {
            return phaseTestService.testAllPhase(text);
        }

        // -- Data Type Tests --

        /**
         * 文字列データ型の処理をテストする
         * 
         * @param text 入力テキスト
         * @return AI でクリーンアップされた文字列
         */
        @PostMapping("/datatype/string")
        public String typeString(@RequestBody String text) {
            return dataTypeTestService.processString(text);
        }

        /**
         * POJO データ型の処理をテストする
         * 
         * @param ユーザー ユーザー オブジェクト
         * @AIによって状態が変更された後のユーザーオブジェクトを返します
         */
        @PostMapping(value = "/datatype/pojo", consumes = MediaType.APPLICATION_JSON_VALUE)
        public User typePojo(@RequestBody User user) {
            return dataTypeTestService.processPojo(user);
        }

        /**
         * テストコレクションのデータ型の処理
         * 
         * @param items 文字列コレクション
         * @return AI分類後のコレクションマッピング
         */
        @PostMapping(value = "/datatype/collection")
        public Map<String, List<String>> typeCollection(@RequestBody List<String> items) {
            return dataTypeTestService.processCollection(items);
        }

        /**
         * 画像データ型の処理をテストする
         * 
         * @paramファイル イメージファイル
         * @param命令処理命令
         * @return AI処理された画像のバイト配列
         * @throws 例外 例外
         */
        @PostMapping("/datatype/image")
        public byte[] typeImage(@RequestParam("image") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processImage(file.getBytes(), instruction);
        }

        /**
         * 音声データ型の処理をテストする
         * 
         * @param ファイル オーディオ ファイル
         * @param命令解析命令
         * @return AIが解析したテキスト記述
         * @throws 例外 例外
         */
        @PostMapping("/datatype/audio")
        public String typeAudio(@RequestParam("audio") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processAudio(file.getBytes(), file.getContentType(), instruction);
        }

        /**
         * ビデオデータ型の処理をテストする
         * 
         * @param ファイル ビデオ ファイル
         * @param命令解析命令
         * @return AIが解析したテキスト記述
         * @throws 例外 例外
         */
        @PostMapping("/datatype/video")
        public String typeVideo(@RequestParam("video") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processVideo(file.getBytes(), file.getContentType(), instruction);
        }

        /**
         * テスト画像コンテンツの詳細な分析
         * 
         * @paramファイル イメージファイル
         * @param命令の説明命令
         * @return 画像解析テキストの説明
         * @throws 例外 例外
         */
        @PostMapping("/datatype/image-analyze")
        public String analyzeImage(@RequestParam("image") MultipartFile file,
                @RequestParam("instruction") String instruction) throws Exception {
            return dataTypeTestService.analyzeImage(file.getBytes(), instruction);
        }

        /**
         * テキスト読み上げ (TTS) をテストする
         * 
         * @param text 入力文字列テキスト
         * @return 合成サウンドオーディオバイナリストリーム
         */
        @PostMapping(value = "/datatype/tts", produces = "audio/wav")
        public byte[] typeTts(@RequestBody String text) {
            return dataTypeTestService.textToSpeech(text);
        }
    }
    ```

- **契約インターフェース (Services)**
  - `PhaseTestService.java`: 異なるインターセプターフェーズ (Before, After, Exception, All) をテストするためのインターフェース。

    ```java
    package com.aispect.example.spring.service;

    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agents.security.ContentModerationAgent;
    import com.aispect.agents.flow.FallbackAgent;
    import com.aispect.agents.data.JsonExtractorAgent;
    import com.aispect.example.spring.agents.SummaryAfterAgent;

    /**
     * ステージテストサービスインターフェース。
     * <p>
     * {@link AiUnitAgent} アノテーションを使用して、さまざまな実行側面の段階 (BEFORE、AFTER、EXCEPTION、ALL) で AiSpect フレームワークをデモンストレーションおよびテストします。
     * インターセプトプロキシ、コンテキスト転送、大規模モデル処理アクセス機能。
     * </p>
     */
    public interface PhaseTestService {

        /**
         * フロントアスペクト（BEFOREステージ）の処理インターセプトをテストします。
         * <p>
         * {@link ContentModerationAgent} をバインドして、ローカルのネイティブ メソッドが実行される前に AI フィルタリングをトリガーします。
         * 大規模なモデルでは、入力テキストの事前準拠とセキュリティのレビューが実行されます。レビューが失敗した場合は、事前に例外をインターセプトしてスローできます。
         * </p>
         *
         * @param input レビュー対象の入力テキストの内容
         * @return 事前レビュー後、ネイティブ ビジネス ロジック処理によって生成されたテキスト結果
         */
        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "実行前にコンテンツコンプライアンスレビューを実施する")
        String testBeforePhase(String input);

        /**
         * ポストアスペクト（AFTERステージ）のテスト処理インターセプト。
         * <p>
         * {@link SummaryAfterAgent} をバインドします。結果を取得するためにネイティブ メソッド (スタブ実装) が最初に実行されます。
         * 返された結果は入力パラメーターとして自動的に AI に渡され、大規模モデルはその後のインテリジェントな要約、改良、および書き換えを実行します。
         * </p>
         *
         * @param input テスト入力の元のテキスト文字列
         * @return 大規模モデルによって要約され洗練された最終的なテキスト結果
         */
        @AiUnitAgent(value = SummaryAfterAgent.class, executePhase = AiExecutePhase.AFTER, description = "実行後、生成されたテキストを要約して抽出します。")
        String testAfterPhase(String input);

        /**
         * 例外の劣化側面のキャプチャとロールバックをテストします (EXCEPTION フェーズ)。
         * <p>
         * {@link FallbackAgent} をバインドします。ネイティブ ビジネス メソッドが動作中に発生し、キャッチされない例外をスローした場合、
         * インターセプターは例外を自動的にインターセプトし、コンテキストと例外情報を大規模モデルに渡して、インテリジェントな災害復旧、復帰および劣化への対応を実現します。
         * </p>
         *
         * @param input 例外またはテストのロールバックをトリガーするテスト入力テキスト
         * @return 例外が発生した後、大規模モデルの災害復旧メカニズムによって返される劣化結果のテキスト
         */
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "例外が発生したときにダウングレードフォールバックメカニズムをトリガーする")
        String testExceptionPhase(String input);

        /**
         * 完全なテイクオーバーの側面 (ALL フェーズ) の処理を​​テストします。
         * <p>
         * {@link JsonExtractorAgent} をバインドします。大規模なモデルは、ネイティブ メソッドの実行ロジックを直接置き換えて引き継ぎます。
         * ネイティブのローカル メソッドは直接スキップされ、AI が入力テキストからフォーマットされた JSON 文字列の結果を直接分析、抽出、組み立てます。
         * </p>
         *
         * @param input 基礎となる JSON 情報または抽出要件を含む入力テキスト
         * @return 大規模なモデルは、抽出された書式設定された JSON 文字列を直接引き継いで解析します。
         */
        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "ライフサイクルのすべての段階で JSON データを抽出および解析する")
        String testAllPhase(String input);
    }
    ```

  - `DataTypeTestService.java`: 異なるデータ型 (String, POJO, Collection, Audio, Video) をテストするためのインターフェース。

    ```java
    package com.aispect.example.spring.service;

    import java.util.List;
    import java.util.Map;

    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agents.data.DataCleanAgent;
    import com.aispect.agents.data.ImageProcessAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.example.spring.agents.AudioProcessAgent;
    import com.aispect.example.spring.agents.CollectionProcessAgent;
    import com.aispect.example.spring.agents.PojoProcessAgent;
    import com.aispect.example.spring.agents.TextToSpeechAgent;
    import com.aispect.example.spring.agents.VideoProcessAgent;
    import com.aispect.example.spring.agents.VideoStoryAgent;
    import com.aispect.example.spring.dto.User;

    /**
     * データ型テスト サービス インターフェイス
     * <p>
     * AiSpect フレームワークによって処理およびサポートされるさまざまなデータ型を示します (Pojos、リスト コレクション、マルチメディア イメージ バイナリ、
     * オーディオ バイナリ、ビデオ バイナリ、音声合成など)、マルチモダリティ処理機能とエージェント オーケストレーション メカニズムを示します。
     * </p>
     */
    public interface DataTypeTestService {

        /**
         * 入力テキスト文字列データをクリーンアップしてフォーマットします。
         * <p>
         * {@link DataCleanAgent} をバインドし、すべてのステージ (ALL ステージ) を引き継ぎ、インテリジェント クリーニングのために元のテキストを大規模モデルに渡します。
         * </p>
         *
         * @param text クリーンアップする元のテキスト文字列
         * @return クリーニング、エラー修正、またはフォーマット後の文字列結果
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "クリーンな文字列データ", executePhase = AiExecutePhase.ALL)
        String processString(String text);

        /**
         * ユーザー オブジェクト データを処理し、その状態プロパティを更新します。
         * <p>
         * {@link PojoProcessAgent} をバインドして、すべてのフェーズ (ALL フェーズ) を引き継ぎます。
         * このメソッドには AOP の「proceed」メカニズムが含まれています。まず、ローカルの `DataTypeTestServiceImpl` で名前とプレフィックスおよびステータスの初期化ビジネス ロジックを実行します。
         * 処理された「ユーザー」は自動的に JSON に変換され、年齢の増加とステータス マークのアップグレードのために AI に渡されます。
         * </p>
         *
         * @param user 名前、年齢、初期ステータスなどを含むユーザー送信オブジェクトエンティティ。
         * @return ローカル処理と大規模モデルのインテリジェントな注釈後の完全な User オブジェクト
         */
        @AiUnitAgent(value = PojoProcessAgent.class, description = "ユーザーオブジェクトデータの処理とステータスの更新", executePhase = AiExecutePhase.ALL)
        User processPojo(User user);

        /**
         * コレクション データ (文字列リスト) を分類し、Key-Value マップのマッピング構造で返します。
         * <p>
         * {@link CollectionProcessAgent} をバインドして、すべてのフェーズ (ALL フェーズ) を引き継ぎます。
         * 非構造化文字列のコレクションを抽出し、トピックまたは属性に従って構造化されたカテゴリに整理する大規模モデルの機能を示します。
         * </p>
         *
         * @param items 分類する文字列リストのコレクション
         * @return 分類結果マッピング カテゴリ別グループ化後のマップ
         */
        @AiUnitAgent(value = CollectionProcessAgent.class, description = "収集データの分類", executePhase = AiExecutePhase.ALL)
        Map<String, List<String>> processCollection(List<String> items);

        /**
         * 指定された編集命令に従って、入力画像バイナリ データに対してマルチモーダルなインテリジェントな変更を実行します。
         * <p>
         * {@link ImageProcessAgent} をバインドして、すべてのフェーズ (ALL フェーズ) を引き継ぎます。
         * 大規模モデルは、入力イメージのバイト配列の処理をサポートし、生​​成されたイメージのバイト配列を直接返します。
         * </p>
         *
         * @param image 元の画像のバイナリ バイト配列
         * @param 命令 画像の処理または変更方法を大規模モデルに指示するテキスト命令
         * @return 変更または処理された画像のバイナリ バイト配列
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "指示に従って画像を処理する", executePhase = AiExecutePhase.ALL)
        byte[] processImage(byte[] image, String instruction);

        /**
         * 音声コンテンツをマルチモーダルに分析し、指示に従って回答します。
         * <p>
         * {@link AudioProcessAgent} をバインドし、使用モデルを {@code gemini-3.5-flash} として指定します。
         * オーディオ バイナリ ストリームを直接読み取り、その音声、メロディー、または環境コンテキストを理解する大規模モデルの機能を実証します。
         * </p>
         *
         * @param audio オーディオのバイナリバイト配列（PCM / WAV 形式データなど）
         * @param mimeType オーディオの MIME メディア タイプ（{@code audio/wav}、{@code audio/mp3} など）
         * @param命令 オーディオを分析するために大規模なモデルを必要とするプロンプトワード命令
         * @return 大規模なモデル分析から抽出されたテキスト回答または文字起こし結果
         */
        @AiUnitAgent(value = AudioProcessAgent.class, description = "音声コンテンツを分析する", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String processAudio(byte[] audio, String mimeType, String instruction);

        /**
         * ビデオコンテンツをマルチモーダルに分析し、指示に従って回答します。
         * <p>
         * {@link VideoProcessAgent} をバインドし、使用モデルを {@code gemini-3.5-flash} として指定します。
         * ビデオ ファイルのクロスモーダル フレームとサウンド ストリームを直接関連付けて理解する大規模モデルの機能を示します。
         * </p>
         *
         * @param video 動画ファイル（MP4形式データなど）のバイナリバイト配列
         * @param mimeType ビデオの MIME メディア タイプ (例: {@code video/mp4})
         * @param命令 ビデオを分析するために大規模なモデルを必要とするプロンプトワード命令
         * @return 大規模なモデル分析から抽出されたテキストの回答または要約
         */
        @AiUnitAgent(value = VideoProcessAgent.class, description = "ビデオコンテンツを分析する", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String processVideo(byte[] video, String mimeType, String instruction);

        /**
         * 複数のモダリティでビデオを自動的に解釈し、構造化されたシーンの説明と物語の要約を生成します。
         * <p>
         * {@link VideoStoryAgent} をバインドし、使用モデルを {@code gemini-3.5-flash} として指定します。
         * 大きなモデルは、指示を与えることなく、ビデオ全体のテーマ、重要なシーン、効果音やセリフを時系列に沿って自動的に記述します。
         * マルチモーダルビデオの理解とコンテンツ作成における包括的な機能を実証します。
         * </p>
         *
         * @param video 動画ファイル（MP4形式データなど）のバイナリバイト配列
         * @param mimeType ビデオの MIME メディア タイプ (例: {@code video/mp4})
         * @return 大きなモデルによって出力されるビデオ シーンの説明と物語の概要テキスト
         */
        @AiUnitAgent(value = VideoStoryAgent.class, description = "ビデオシーンの説明と物語の概要を自動的に生成", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String describeVideo(byte[] video, String mimeType);

        /**
         * 画像コンテンツをマルチモーダルに分析し、分析指示をテキスト形式で直接返します。
         * <p>
         * {@link ImageProcessAgent} をバインドし、使用モデルを {@code gemini-3.5-flash} として指定します。
         * 画像内のシーン、テキスト (OCR)、およびエンティティを抽出し、説明を生成する大規模モデルの機能を示します。
         * </p>
         *
         * @param image 元の画像のバイナリ バイト配列
         * @param命令プロンプトワード命令を抽出して説明する
         * @return 画像の説明文または解析結果
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "画像コンテンツを分析し、テキストの説明を返します", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String analyzeImage(byte[] image, String instruction);

        /**
         * 大規模なモデルを通じて、入力テキストを高忠実度サウンドのオーディオに変換します。
         * <p>
         * {@link TextToSpeechAgent} をバインドし、ネイティブ音声合成大規模モデル {@code gemini-3.1-flash-tts-preview} を使用するように指定します。
         * 大規模モデルのマルチモーダル TTS インターフェイスを通じてテキストをオーディオ バイナリ ストリームに生成し、それを直接出力する機能を示します。
         * </p>
         *
         * @param text 音声に変換する入力文字列テキスト
         * @return 生成された音声オーディオファイル（MP3形式のオーディオバイトストリームなど）のバイナリバイト配列
         */
        @AiUnitAgent(value = TextToSpeechAgent.class, description = "テキストを音声に変換する", modelName = "gemini-3.1-flash-tts-preview", executePhase = AiExecutePhase.ALL)
        byte[] textToSpeech(String text);

        /**
         * フレームワークのプロキシ アノテーションをバイパスし、元のユーザー処理インターフェイスのリフレクションまたは直接呼び出しをサポートします。
         *
         * @param user ユーザー転送オブジェクトエンティティ
         * @return 処理結果オブジェクト
         */
        Object processPojo(Object user);
    }
    ```

  - `GraphTestService.java`: エージェントグラフ (Agent Graph) オーケストレーションをテストするためのインターフェース。

    ```java
    package com.aispect.example.spring.service;

    import java.util.Map;

    import com.aispect.agent.annotation.AiGraphAgent;
    import com.aispect.agent.annotation.AiNode;
    import com.aispect.agents.graph.DynamicWorkflowGraphAgent;

    /**
     * Graph Agent のワークフロー オーケストレーション サービス インターフェイスをテストします。
     * <p>
     * このインターフェースは {@link AiGraphAgent} と連携し、{@link DynamicWorkflowGraphAgent} トポロジ グラフ エージェントをバインドします。
     * 一連のデータ依存のマルチステップ (複数の {@link AiNode} によって識別されるステップ) を管理およびスケジュールするための AI 協調生成ワークフロー。
     * ストーリーのアウトラインを作成する -> アウトラインに基づいてストーリーを書く -> 書かれたストーリーを要約する というプロセスが含まれます。
     * </p>
     */
    @AiGraphAgent(name = "workflowGraphAgent", value = DynamicWorkflowGraphAgent.class, description = "A generic workflow agent that orchestrates a series of tasks")
    public interface GraphTestService {

        /**
         * テーマの背景に基づいてストーリーの概要を作成します (ステップ 1)。
         * <p>
         * ワークフローの「generateOutline」ノードに対応し、グローバル コンテキスト (globalContext) の入力パラメータ {@code "initialTopic"} に依存します。
         * 生成されたストーリー アウトラインは、次のステップの入力としてグローバル コンテキストに再度保存されます。
         * </p>
         *
         * @param globalContext コンテキスト マッピング プロセスによってグローバルに共有されるマップ。少なくともキー {@code "initialTopic"} が含まれている必要があります。
         * @return 生成された概要テキストのコンテンツ
         */
        @AiNode(name = "generateOutline", description = "Given the global context which contains 'initialTopic', generate a story outline and return it.")
        String generateOutline(Map<String, Object> globalContext);

        /**
         * 生成されたストーリー アウトライン フレームワークに基づいて、短くて完全なストーリーを作成します (ステップ 2)。
         * <p>
         * ワークフローの「writeStory」ノードに対応する入力ソースは、前のステップで生成された {@code "generateOutline_result"} です。
         * 結果として得られるストーリーはグローバル コンテキストに保存されます。
         * </p>
         *
         * @param globalContext コンテキスト マッピング プロセスによってグローバルに共有されるマップ。少なくともキー {@code "generateOutline_result"} が含まれている必要があります。
         * @return 短編小説のテキスト コンテンツが作成されました
         */
        @AiNode(name = "writeStory", description = "Given the global context which contains 'generateOutline_result', write a short story based on the outline and return it.")
        String writeStory(Map<String, Object> globalContext);

        /**
         * 前の段階 (ステップ 3) で生成されたストーリーを賢く要約します。
         * <p>
         * ワークフローの `summarizeStory` ノードに対応し、その入力ソースは 2 番目のステップで生成された {@code "writeStory_result"} です。
         * ストーリーの最終的な要約は、ワークフロー全体の最終出力として返されるか、グローバル コンテキストに保存されます。
         * </p>
         *
         * @param globalContext コンテキスト マッピング プロセスによってグローバルに共有されるマップ。少なくともキー {@code "writeStory_result"} が含まれている必要があります。
         * @return ストーリーの一文の要約テキスト
         */
        @AiNode(name = "summarizeStory", description = "Given the global context which contains 'writeStory_result', summarize the story in one sentence and return it.")
        String summarizeStory(Map<String, Object> globalContext);
    }
    ```

  - `AiSpectErrorTestService.java`: エラー処理とフォールバックをテストするためのインターフェース。

    ```java
    package com.aispect.example.spring.service;

    import com.aispect.agent.annotation.AiNode;
    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agents.data.DataCleanAgent;

    /**
     * AiSpect コンポーネントのスキャン期間中に異常情報をテストするために使用されるサービス インターフェイス。
     * <p>
     * このインターフェイスには、具体的には、{@link com.aispect.common.Exception.AiScanException} エラーを引き起こす可能性があるエージェントとノードの一連の構成使用法が含まれています。
     * これは、ブートストラップ (スタートアップ スキャン検証) フェーズ中に、アノテーションの競合や不正な構成に対するフレームワークの堅牢なチェック機能とフレンドリーなエラー プロンプト機能を検証およびテストするように設計されています。
     * </p>
     */
    public interface AiSpectErrorTestService {

        /**
         * シナリオ 1: 同じメソッドで同じ実行フェーズ (デフォルトは ALL フェーズ) を繰り返しバインドするユニット エージェントの競合をテストします。
         * <p>
         * 複数の {@link AiUnitAgent} が同じメソッドにアノテーションを付けられている場合は、異なる実行フェーズ (executePhase) を指定する必要があります。
         * 指定されない場合 (暗黙的にデフォルトで ALL ステージに設定される)、または明示的に同じステージに設定される場合、同じステージの競合検証エラーがトリガーされます。
         * </p>
         *
         * @param input テスト入力の元のテキスト文字列
         * @return エージェントによって処理された応答テキスト。通常のスキャンを開始するときにエラーが報告された場合、それまでのスキャンは実行できません。
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "エージェント 1: デフォルト ステージ")
        @AiUnitAgent(value = DataCleanAgent.class, description = "エージェント 2: これもデフォルトのステージです")
        String testSamePhaseConflict(String input);

        /**
         * シナリオ 2: 同じメソッド上で、フルフェーズ テイクオーバー (ALL フェーズ) とローカルの特定のフェーズ (BEFORE フェーズなど) を混合した競合がないかテストします。
         * <p>
         * {@link AiExecutePhase#ALL} は、ターゲット メソッドの実行ロジックを完全に引き継ぐことを意味します。これは、他のフェーズ (BEFORE または AFTER など) を意味します。
         * それと一緒に暮らすことはできません。 ALL フェーズと BEFORE/AFTER フェーズの両方のエージェントが同時にマークされると、フレームワークはスケジュールに失敗し、スキャン期間の競合例外をトリガーします。
         * </p>
         *
         * @param input テスト入力の元のテキスト文字列
         * @return 処理された応答結果
         */
        @AiUnitAgent(value = DataCleanAgent.class, executePhase = AiExecutePhase.ALL, description = "フルステージ乗っ取り")
        @AiUnitAgent(value = DataCleanAgent.class, executePhase = AiExecutePhase.BEFORE, description = "前段処理")
        String testAllPhaseMixedConflict(String input);

        /**
         * シナリオ 3: 同じメソッド上の同じ実行フェーズ (デフォルト フェーズ) を繰り返しバインドすることによる {@link AiNode} の競合をテストします。
         * <p>
         * メソッドがグラフ ノードに属し、複数の {@link AiNode} アノテーションにバインドされている場合、これらのノードが同じ実行フェーズを使用する場合、
         * AiNode の同相競合検証エラーは、フレームワークがスキャンを開始するとトリガーされます。
         * </p>
         *
         * @param input テスト入力の元のテキスト文字列
         * @ワークフローノードによる処理後の結果を返す
         */
        @AiNode(name = "node1", description = "ノード 1: デフォルト ステージ")
        @AiNode(name = "node2", description = "ノード 2: これもデフォルトのステージです")
        String testNodeSamePhaseConflict(String input);

        /**
         * シナリオ 4: {@link AiNode} の ALL フルフェーズ テイクオーバーと特定のフェーズ (BEFORE フェーズ) を混在させるテストの競合。
         * <p>
         * ユニット エージェントと同様に、AiNode の ALL フェーズを他のローカル実行フェーズと混合したり、同じメソッドで構成したりすることはできません。
         * そうしないと、グラフ ワークフローの単一ノード スケジューリング ルートが破棄され、コンポーネント スキャン期間の競合エラーがトリガーされます。
         * </p>
         *
         * @param input テスト入力の元のテキスト文字列
         * @ワークフローノードによる処理後の結果を返す
         */
        @AiNode(name = "nodeAll", executePhase = AiExecutePhase.ALL, description = "フルステージ乗っ取り")
        @AiNode(name = "nodeBefore", executePhase = AiExecutePhase.BEFORE, description = "前段処理")
        String testNodeAllPhaseMixedConflict(String input);

    }
    ```

- **サービス実装クラス**
  - `PhaseTestServiceImpl.java`, `DataTypeTestServiceImpl.java`, `GraphTestServiceImpl.java`, `AiSpectErrorTestServiceImpl.java`: 各サービスのネイティブ実装 (フォールバック/スタブロジック)。

    ```java
    package com.aispect.example.spring.service;

    import org.springframework.stereotype.Service;

    /**
     * ステージテストサービス実装クラス
     * さまざまなライフサイクル段階に応じたテスト方法ロジックを実装
     */
    @Service
    public class PhaseTestServiceImpl implements PhaseTestService {

        @Override
        public String testBeforePhase(String input) {
            return "Original method executed successfully with input: " + input;
        }

        @Override
        public String testAfterPhase(String input) {
            // AI will intercept this returned string and summarize it
            return "The quick brown fox jumps over the lazy dog. This is a very long and detailed sentence that needs to be summarized by the AI agent in the AFTER phase.";
        }

        @Override
        public String testExceptionPhase(String input) {
            // Simulate a runtime exception
            if ("error".equalsIgnoreCase(input)) {
                throw new RuntimeException("Simulated internal server error in testExceptionPhase");
            }
            return "No error occurred.";
        }

        @Override
        public String testAllPhase(String input) {
            // This should be short-circuited by JsonExtractorAgent and never run.
            return "This text should not be returned, because ALL phase agent handles it.";
        }
    }
    ```

    ```java
    package com.aispect.example.spring.service;

    import java.util.Collections;
    import java.util.List;
    import java.util.Map;

    import org.springframework.stereotype.Service;

    import com.aispect.example.spring.dto.User;

    /**
     * データ型テストサービス実装クラス
     * さまざまなデータ型 (文字列、POJO、コレクション、イメージ、ストリームなど) のメソッドのデフォルト実装を提供します。
     */
    @Service
    public class DataTypeTestServiceImpl implements DataTypeTestService {

        @Override
        public String processString(String text) {
            return text;
        }

        @Override
        public User processPojo(User user) {
            if (user == null) {
                user = new User();
            }
            if (user.getName() != null && !user.getName().startsWith("[Local]")) {
                user.setName("[Local] " + user.getName());
            }
            if (user.getStatus() == null || user.getStatus().isEmpty()) {
                user.setStatus("PENDING");
            }
            return user;
        }

        @Override
        public Map<String, List<String>> processCollection(List<String> items) {
            return Collections.emptyMap();
        }

        @Override
        public byte[] processImage(byte[] image, String instruction) {
            return image;
        }

        @Override
        public String processAudio(byte[] audio, String mimeType, String instruction) {
            return "";
        }

        @Override
        public String processVideo(byte[] video, String mimeType, String instruction) {
            return "";
        }

        @Override
        public String describeVideo(byte[] video, String mimeType) {
            return "";
        }

        @Override
        public String analyzeImage(byte[] image, String instruction) {
            return "";
        }

        @Override
        public byte[] textToSpeech(String text) {
            return new byte[0];
        }

        @Override
        public Object processPojo(Object user) {
            throw new UnsupportedOperationException("Unimplemented method 'processPojo'");
        }
    }
    ```

    ```java
    package com.aispect.example.spring.service;

    import java.util.Map;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Service;

    /**
     * Graph Agentのサービス実装をテストする
     * DynamicWorkflowGraphAgent によってスケジュールできる複数の @AiNode が含まれています
     */
    @Service
    public class GraphTestServiceImpl implements GraphTestService {

        private static final Logger logger = LoggerFactory.getLogger(GraphTestServiceImpl.class);

        @Override
        public String generateOutline(Map<String, Object> globalContext) {
            Object topic = globalContext.get("initialTopic");
            if (topic == null) {
                topic = "Default Topic";
            }
            logger.info("Generating outline for topic: {}", topic);
            return "Outline for: " + topic + " - 1. Introduction, 2. Climax, 3. Resolution";
        }

        @Override
        public String writeStory(Map<String, Object> globalContext) {
            Object outline = globalContext.get("generateOutline_result");
            logger.info("Writing story based on outline: {}", outline);
            return "Once upon a time, there was a great adventure. " + outline + ". The End.";
        }

        @Override
        public String summarizeStory(Map<String, Object> globalContext) {
            Object story = globalContext.get("writeStory_result");
            logger.info("Summarizing story: {}", story);
            return "Summary: A thrilling adventure happened.";
        }
    }
    ```

    ```java
    package com.aispect.example.spring.service;

    import org.springframework.stereotype.Service;

    @Service
    public class AiSpectErrorTestServiceImpl implements AiSpectErrorTestService {

        @Override
        public String testSamePhaseConflict(String input) {
            return "testSamePhaseConflict";
        }

        @Override
        public String testAllPhaseMixedConflict(String input) {
            return "testAllPhaseMixedConflict";
        }

        @Override
        public String testNodeSamePhaseConflict(String input) {
            return "testNodeSamePhaseConflict";
        }

        @Override
        public String testNodeAllPhaseMixedConflict(String input) {
            return "testNodeAllPhaseMixedConflict";
        }

    }
    ```

- **エージェント (Agents)**
  - `SummaryAfterAgent.java`: メソッド実行後にテキストを要約するエージェント。

    ```java
    package com.aispect.example.spring.agents;

    import java.util.Collections;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    /**
     * 要約後のエージェント
     * AbstractUnitAgent を継承して大規模モデルを呼び出し、ターゲット メソッドの実行後に結果を要約します。
     */
    public class SummaryAfterAgent extends AbstractUnitAgent<Object, String> {
        @Override
        public String postExecute(AiInvocationContext<Object> ctx, String result) throws AiSpectException {
            String input = result != null ? result : "";

            String systemPrompt = "You are a summarizing assistant. Summarize the user's text into a concise, professional sentence.";
            Message userMsg = new Message(Role.USER, input);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

  - `CollectionProcessAgent.java`: コレクション内のアイテムを分類するエージェント。

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;
    import com.fasterxml.jackson.core.type.TypeReference;
    import com.fasterxml.jackson.databind.ObjectMapper;

    import java.util.Collections;
    import java.util.List;
    import java.util.Map;

    import com.fasterxml.jackson.core.JsonProcessingException;

    /**
     * 回収処理業者
     * 大規模なモデルを通じて受信リスト項目をインテリジェントに分類し、JSON マッピング結果を返します。
     */
    public class CollectionProcessAgent extends AbstractUnitAgent<Object, Map<String, List<String>>> {
        private static final ObjectMapper mapper = new ObjectMapper();

        @Override
        public Map<String, List<String>> execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            String input = (args != null && args.length > 0) ? args[0].toString() : "[]";
            
            String systemPrompt = "You are a categorization assistant. Group the following list of items into categories. Output ONLY valid JSON mapping category name (String) to a list of items (List<String>), no markdown.";
            Message userMsg = new Message(Role.USER, input);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            String result = response.message().content().replace("
    ```

  - `PojoProcessAgent.java`: 複雑な POJO 構造を処理するエージェント。

    ```java
    package com.aispect.example.spring.agents;

    import java.util.Collections;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;
    import com.aispect.example.spring.dto.User;
    import com.fasterxml.jackson.core.JsonProcessingException;
    import com.fasterxml.jackson.databind.ObjectMapper;

    /**
     * POJO処理エージェント
     * 大きなモデルを呼び出してユーザー オブジェクト (User) のプロパティを変更し、更新されたオブジェクトを返します。
     */
    public class PojoProcessAgent extends AbstractUnitAgent<Object, User> {
        private static final ObjectMapper mapper = new ObjectMapper();

        @Override
        public User execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            // 1. ローカル独自メソッドのビジネスロジック（AOP進行機構）を明示的に呼び出す
            User localUser;
            try {
                localUser = (User) ctx.proceed();
            } catch (Throwable e) {
                throw new AiSpectException("Local invocation failed", e);
            }

            if (localUser == null) {
                localUser = new User();
            }
            
            String inputJson;
            try {
                inputJson = mapper.writeValueAsString(localUser);
            } catch(JsonProcessingException e) {
                inputJson = "{}";
            }

            String systemPrompt = "You are an AI post-processor. Read the User JSON (which has been locally processed). Update the 'status' to 'VERIFIED' and add 10 to 'age'. Return ONLY valid JSON, no markdown.";
            Message userMsg = new Message(Role.USER, inputJson);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            String aiResult = response.message().content().replace("
    ```

  - `AudioProcessAgent.java`, `VideoProcessAgent.java`, `TextToSpeechAgent.java`, `VideoStoryAgent.java`: オーディオやビデオなどのマルチモーダルタスクを処理するためのエージェント。

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * オーディオ処理エージェントはメソッドをインターセプトし、オーディオ byte[] とその mimeType を inlineData にカプセル化して、大規模モデルに送信します。
     */
    public class AudioProcessAgent extends AbstractUnitAgent<Object, String> {

        @Override
        public String execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            byte[] audioBytes = (args != null && args.length > 0 && args[0] instanceof byte[]) ? (byte[]) args[0] : new byte[0];
            String mimeType = (args != null && args.length > 1 && args[1] instanceof String) ? (String) args[1] : "audio/mp3";
            String instruction = (args != null && args.length > 2 && args[2] instanceof String) ? (String) args[2] : "Describe this audio";

            String systemPrompt = "You are an audio analysis assistant. Please process the provided audio according to the user instruction.";
            Message userMsg = new Message(Role.USER, instruction, null, null, audioBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * ビデオ処理エージェントはメソッドをインターセプトし、ビデオ byte[] とその mimeType を inlineData にカプセル化し、大規模モデルに送信します。
     */
    public class VideoProcessAgent extends AbstractUnitAgent<Object, String> {

        @Override
        public String execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            byte[] videoBytes = (args != null && args.length > 0 && args[0] instanceof byte[]) ? (byte[]) args[0] : new byte[0];
            String mimeType = (args != null && args.length > 1 && args[1] instanceof String) ? (String) args[1] : "video/mp4";
            String instruction = (args != null && args.length > 2 && args[2] instanceof String) ? (String) args[2] : "Describe this video";

            String systemPrompt = "You are a video analysis assistant. Please process the provided video according to the user instruction.";
            Message userMsg = new Message(Role.USER, instruction, null, null, videoBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * 音声合成エージェントは、テキスト プロンプトの単語を音声に変換します。
     */
    public class TextToSpeechAgent extends AbstractUnitAgent<Object, byte[]> {

        @Override
        public byte[] execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            String text = (args != null && args.length > 0 && args[0] != null) ? args[0].toString() : "";

            String systemPrompt = "You are a text-to-speech assistant. Please convert the user's text into speech audio data.";
            Message userMsg = new Message(Role.USER, text);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            
            // response.message() が inlineData を返す場合、それはオーディオ バイトとして直接返されます。
            if (response.message().inlineData() != null) {
                return response.message().inlineData();
            }
            
            // 最後の行: テキストの内容に対応するバイト配列を返します。
            return response.message().content() != null ? response.message().content().getBytes() : new byte[0];
        }
    }
    ```

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * ビデオ ストーリーテリング エージェントを視聴します。
     * <p>
     * エージェントはビデオ バイナリ データを受け取り、大規模なマルチモーダル モデルを使用してビデオ画像と音声を自動的に分析します。
     * ユーザーからの追加の指示を必要とせずに、詳細なシーンの説明と物語の概要を生成します。
     * </p>
     */
    public class VideoStoryAgent extends AbstractUnitAgent<Object, String> {

        @Override
        public String execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            byte[] videoBytes = (args != null && args.length > 0 && args[0] instanceof byte[]) ? (byte[]) args[0] : new byte[0];
            String mimeType = (args != null && args.length > 1 && args[1] instanceof String) ? (String) args[1] : "video/mp4";

            String systemPrompt = "You are a professional video narrator. "
                    + "Watch the provided video carefully and generate a vivid, structured description: "
                    + "1) Summarize the overall theme or story of the video in one sentence. "
                    + "2) Describe the key scenes or moments chronologically. "
                    + "3) Note any significant audio, dialogue, or sound effects. "
                    + "Answer in the same language as the user's message (default: Chinese).";

            Message userMsg = new Message(Role.USER, "このビデオの内容とシーンを詳しく説明してください。", null, null, videoBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

- **設定と DTO**
  - `AiGraphConfig.java`: Agent Graph ワークフローを定義するための設定クラス。

    ```java
    package com.aispect.example.spring.config;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.function.RouterFunction;
    import org.springframework.web.servlet.function.RouterFunctions;
    import org.springframework.web.servlet.function.ServerResponse;

    import com.aispect.api.AiOperations;
    import com.aispect.engine.spring.launcher.AiGraphAgentSpringLauncher;

    /**
     * サンプル アプリケーションで Graph Agent 関連 Bean を手動で構成するための構成クラス。
     */
    @Configuration
    public class AiGraphConfig {

        /**
         * AiGraphAgentSpringLauncherを登録します。
         * Spring の開始時にすべての @AiGraphAgent を収集し、ライフサイクルを管理する責任を負います。
         *
         * @param aiOperations 基礎となる自動アセンブリによって提供される、コア AI クライアント操作インターフェイス
         * @return ランチャーインスタンス
         */
        @Bean
        public AiGraphAgentSpringLauncher aiGraphAgentSpringLauncher(AiOperations aiOperations) {
            return new AiGraphAgentSpringLauncher(aiOperations);
        }

        /**
         * Graph AgentのHTTPルートを登録します。
         * ここでは、ルーティングパスやリクエストメソッドを自由にカスタマイズし、実行タスクをLauncherに振り分けるかどうかはビジネス層に任されています。
         *
         * @param ランチャー グラフランチャー
         * @return 動的に登録されたエンドポイント ルート
         */
        @Bean
        public RouterFunction<ServerResponse> graphAgentRouterFunction(AiGraphAgentSpringLauncher launcher) {
            return RouterFunctions.route()
                    .POST("/ai/graph/{agentName}", request -> {
                        String agentName = request.pathVariable("agentName");
                        try {
                            String body = request.body(String.class);
                            Object result = launcher.executeAgent(agentName, body);
                            return ServerResponse.ok().body(result != null ? result : "Graph execution completed with null result.");
                        } catch (IllegalArgumentException e) {
                            return ServerResponse.notFound().build();
                        } catch (Exception e) {
                            return ServerResponse.status(500).body("Error executing graph agent: " + e.getMessage());
                        }
                    })
                    .build();
        }
    }
    ```

  - `User.java`: POJO バインディングを示すために使用される DTO。

    ```java
    package com.aispect.example.spring.dto;

    /**
     * ユーザー データ転送オブジェクト (DTO)
     * AiSpect でエンティティ クラスの配信と処理をテストするために使用されます。
     */
    public class User {
        private String name;
        private int age;
        private String status;

        public User() {}

        public User(String name, int age, String status) {
            this.name = name;
            this.age = age;
            this.status = status;
        }

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
        public String getStatus() { return status; }
        public void setStatus(String status) { this.status = status; }
    }
    ```

- **リソースとフロントエンド**
  - `application.properties`: Spring Boot アプリケーション設定ファイル。

    ```properties
    logging.level.com.aispect=DEBUG

    # AiSpect AI Framework Configuration
    aispect.provider=gemini
    aispect.default-model=gemini-2.5-flash
    # You can set the API key directly here, or inject it via environment variables (e.g., from .env)
    aispect.api-key=${GEMINI_API_KEY:}
    ```

  - `index.html`: モダンでダークテーマのインタラクティブな API テストダッシュボードを提供します。

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>AiSpect API Dashboard</title>
        <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
        <link href="/css/style.css" rel="stylesheet">
    </head>
    <body>

        <aside class="sidebar">
            <div class="logo-container">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L2 7l10 5 10-5-10-5z"></path><path d="M2 17l10 5 10-5"></path><path d="M2 12l10 5 10-5"></path></svg>
                AiSpect
            </div>

            <div class="nav-menu-container">
                <div class="nav-section">
                    <div class="nav-title" onclick="toggleSection(this)">
                        <span>Phase Tests</span>
                        <svg class="chevron-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
                    </div>
                    <div id="nav-phase" class="nav-items-wrapper"></div>
                </div>

                <div class="nav-section">
                    <div class="nav-title" onclick="toggleSection(this)">
                        <span>Data Type Tests</span>
                        <svg class="chevron-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
                    </div>
                    <div id="nav-datatype" class="nav-items-wrapper"></div>
                </div>

                <div class="nav-section">
                    <div class="nav-title" onclick="toggleSection(this)">
                        <span>Graph Tests</span>
                        <svg class="chevron-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
                    </div>
                    <div id="nav-graph" class="nav-items-wrapper"></div>
                </div>
            </div>
        </aside>

        <main class="main-wrapper">
            <header class="header">
                <h1 id="page-title">Select a Test</h1>
                <p id="page-desc">Choose an API endpoint from the sidebar to begin testing.</p>
            </header>

            <div class="content-grid" id="main-content" style="display: none;">
                <!-- Request Card -->
                <div class="card fade-in">
                    <h3>
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--primary)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
                        Configure Request
                    </h3>
                    <div id="endpoint-info" class="endpoint-badge"></div>
                    <form id="test-form" onsubmit="event.preventDefault(); runTest();">
                        <div id="form-fields"></div>
                        <button type="submit" class="btn btn-primary" id="run-btn">
                            <span>Send Request</span>
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>
                        </button>
                    </form>
                </div>

                <!-- Response Card -->
                <div class="card fade-in" style="animation-delay: 0.1s;">
                    <h3 style="display: flex; justify-content: space-between;">
                        <span style="display: flex; align-items: center; gap: 8px;">
                            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--secondary)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="21 8 21 21 3 21 3 8"></polyline><rect x="1" y="3" width="22" height="5"></rect><line x1="10" y1="12" x2="14" y2="12"></line></svg>
                            Response
                        </span>
                        <span id="response-status" style="font-size: 13px; font-weight: normal; color: var(--text-muted);"></span>
                    </h3>
                    <div class="result-container" id="result-box">
                        <div class="empty-state" id="empty-state">
                            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" style="opacity: 0.5; margin-bottom: 16px;"><circle cx="12" cy="12" r="10"></circle><polyline points="12 16 16 12 12 8"></polyline><line x1="8" y1="12" x2="16" y2="12"></line></svg>
                            Waiting for execution...
                        </div>
                        <div class="empty-state" id="loading-state" style="display: none;">
                            <div class="spinner"></div>
                            <p style="margin-top: 16px; color: white;">Processing request...</p>
                        </div>
                        <pre class="result-output" id="result-text" style="display: none;"></pre>
                        <img class="result-image" id="result-image" />
                        <audio class="result-audio" id="result-audio" controls style="display: none; width: 100%; margin-top: 16px; outline: none;"></audio>
                    </div>
                </div>
            </div>
        </main>
        <script src="/js/app.js"></script>
    </body>
    </html>
    ```

  - `app.js`: リクエストの設定と結果のプレビュー (JSON フォーマット、SSE ストリーミング) のためのフロントエンドロジック。

    ```javascript
    /**
     * AiSpect デモ コントロール パネルのコア JavaScript スクリプト
     * 左側のテスト ナビゲーション バーのレンダリング、リクエスト構成フォームの動的構築、AOP インタラクション リクエストの開始、さまざまなマルチモーダル AI 応答の解析とレンダリングを担当します。
     */

    // すべての API テスト シナリオの構成レジストリ
    const testConfigs = [
        // --- フェーズテスト (アスペクトライフサイクルフェーズテスト) ---
        { 
            id: 'phase-before', 
            group: 'phase', 
            title: 'Phase: Before', 
            type: 'GET', 
            url: '/test/phase/before', 
            inputs: [{name: 'input', label: 'Input Text', type: 'text', default: 'safe data'}] 
        },
        { 
            id: 'phase-after', 
            group: 'phase', 
            title: 'Phase: After', 
            type: 'GET', 
            url: '/test/phase/after', 
            inputs: [{name: 'input', label: 'Input Text', type: 'text', default: 'dummy'}] 
        },
        { 
            id: 'phase-exception', 
            group: 'phase', 
            title: 'Phase: Exception', 
            type: 'GET', 
            url: '/test/phase/exception', 
            inputs: [{name: 'input', label: 'Input Text', type: 'text', default: 'error'}] 
        },
        { 
            id: 'phase-all', 
            group: 'phase', 
            title: 'Phase: All', 
            type: 'POST', 
            url: '/test/phase/all', 
            isBody: true, 
            inputs: [{name: 'body', label: 'Raw Text', type: 'textarea', default: 'Hello AiSpect'}] 
        },
        
        // --- データ型テスト (マルチモーダルおよび異なるデータ型テスト) ---
        { 
            id: 'type-string', 
            group: 'datatype', 
            title: 'Type: String', 
            type: 'POST', 
            url: '/test/datatype/string', 
            isBody: true, 
            inputs: [{name: 'body', label: 'String Body', type: 'textarea', default: 'Testing raw string injection'}] 
        },
        { 
            id: 'type-pojo', 
            group: 'datatype', 
            title: 'Type: POJO', 
            type: 'POST', 
            url: '/test/datatype/pojo', 
            isJson: true, 
            inputs: [{name: 'body', label: 'JSON User Object', type: 'textarea', default: '{\n  "name": "Alice",\n  "age": 25,\n  "status": "NEW"\n}'}] 
        },
        { 
            id: 'type-collection', 
            group: 'datatype', 
            title: 'Type: Collection', 
            type: 'POST', 
            url: '/test/datatype/collection', 
            isJson: true, 
            inputs: [{name: 'body', label: 'JSON Array', type: 'textarea', default: '[\n  "apple",\n  "banana",\n  "carrot"\n]'}] 
        },
        { 
            id: 'type-image', 
            group: 'datatype', 
            title: 'Type: Image', 
            type: 'POST', 
            url: '/test/datatype/image', 
            isFormData: true, 
            inputs: [
                {name: 'image', label: 'Upload Image', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Describe the contents of this image'}
            ], 
            responseType: 'blob' 
        },
        { 
            id: 'type-image-analyze', 
            group: 'datatype', 
            title: 'Type: Image Analyze (Text)', 
            type: 'POST', 
            url: '/test/datatype/image-analyze', 
            isFormData: true, 
            inputs: [
                {name: 'image', label: 'Upload Image', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Perform a detailed scene analysis of this image.'}
            ] 
        },
        { 
            id: 'type-audio', 
            group: 'datatype', 
            title: 'Type: Audio Analysis', 
            type: 'POST', 
            url: '/test/datatype/audio', 
            isFormData: true, 
            inputs: [
                {name: 'audio', label: 'Upload Audio', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Transcribe this audio and explain its background environment.'}
            ] 
        },
        { 
            id: 'type-video', 
            group: 'datatype', 
            title: 'Type: Video Analysis', 
            type: 'POST', 
            url: '/test/datatype/video', 
            isFormData: true, 
            inputs: [
                {name: 'video', label: 'Upload Video', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Summarize the visual and audio actions in this video.'}
            ] 
        },
        { 
            id: 'type-tts', 
            group: 'datatype', 
            title: 'Type: Text to Speech (TTS)', 
            type: 'POST', 
            url: '/test/datatype/tts', 
            isBody: true, 
            inputs: [
                {name: 'body', label: 'Text to Speak', type: 'textarea', default: 'Welcome to AiSpect, the next generation framework for agentic development!'}
            ], 
            responseType: 'audio' 
        },
        { 
            id: 'type-stream', 
            group: 'datatype', 
            title: 'Type: Stream', 
            type: 'GET', 
            url: '/test/datatype/stream', 
            inputs: [{name: 'topic', label: 'Topic', type: 'text', default: 'Cyberpunk City'}], 
            isStream: true 
        },
        
        // --- グラフ テスト (トポロジ グラフ ワークフロー オーケストレーション テスト) ---
        { 
            id: 'graph-execute', 
            group: 'graph', 
            title: 'Graph: Workflow', 
            type: 'POST', 
            url: '/ai/graph/workflowGraphAgent', 
            isBody: true, 
            inputs: [{name: 'body', label: 'Topic (String)', type: 'textarea', default: 'A lone spaceman exploring a ruined planet.'}] 
        }
    ];

    // 現在選択されているテスト構成アイテム
    let currentTest = null;
    // ユーザーが繰り返し送信したり同時実行の競合を防ぐためにリクエストをキャンセルするためのコントローラー
    let currentAbortController = null;

    /**
     * 左側のサイドバーのナビゲーション項目を初期化する
     * 構成レジストリを走査し、ボタン要素を動的に作成してクリック選択イベントをバインドし、グループを対応する DOM コンテナにマウントします。
     */
    function initSidebar() {
        const phaseNav = document.getElementById('nav-phase');
        const datatypeNav = document.getElementById('nav-datatype');
        const graphNav = document.getElementById('nav-graph');

        testConfigs.forEach(test => {
            const el = document.createElement('div');
            el.className = 'nav-item';
            // SVG シェブロン右矢印アイコンの使用
            el.innerHTML = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"></polyline></svg> ${test.title}`;
            el.onclick = () => selectTest(test.id);
            el.id = `nav-${test.id}`;

            // グループごとに該当セクションにマウント
            if (test.group === 'phase') phaseNav.appendChild(el);
            else if (test.group === 'datatype') datatypeNav.appendChild(el);
            else if (test.group === 'graph') graphNav.appendChild(el);
        });
    }

    /**
     * テスト項目のイベントハンドラーを選択します
     * 現在のナビゲーション状態をアクティブにし、右側のコンテンツ パネル、フォーム フィールド、および以前に生成された結果領域をリセットします。
     * 
     * @param {string} id テスト項目の一意の ID
     */
    function selectTest(id) {
        // すべてのナビゲーション項目からアクティブなスタイルを削除し、新しく選択した項目にアクティブを割り当てます
        document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
        document.getElementById(`nav-${id}`).classList.add('active');

        // 現在のテスト項目構成を取得して記録します
        currentTest = testConfigs.find(t => t.id === id);
        
        // ページのタイトルと説明情報を更新します
        document.getElementById('page-title').textContent = currentTest.title;
        document.getElementById('page-desc').textContent = `Testing endpoint: ${currentTest.url}`;
        
        // レンダリング API タイプのフラグ バッジ
        const badge = document.getElementById('endpoint-info');
        badge.textContent = `${currentTest.type} ${currentTest.url}`;
        badge.className = `endpoint-badge ${currentTest.type.toLowerCase()}`;

        // リクエストパラメータ設定フォームを動的にレンダリングする
        renderForm();
        // 最後のリクエストによって生成された応答データをクリアします
        resetResult();
        // 右側にメイングリッドコンテナを表示します
        document.getElementById('main-content').style.display = 'grid';
    }

    /**
     * 現在のテスト構成に基づいてリクエスト入力フォームを動的に構築してレンダリングします。
     * テキスト ボックス (テキスト)、長いテキスト ボックス (テキストエリア)、およびファイル アップロード ボタン (ファイル) のレンダリングをサポートします。
     */
    function renderForm() {
        const container = document.getElementById('form-fields');
        container.innerHTML = ''; // 古いフォームフィールドをクリアする

        currentTest.inputs.forEach(input => {
            const group = document.createElement('div');
            group.className = 'form-group';
            
            const label = document.createElement('label');
            label.className = 'form-label';
            label.textContent = input.label;
            group.appendChild(label);

            // 入力タイプに応じて、対応するフォーム コントロールを選択してレンダリングします。
            if (input.type === 'textarea') {
                const el = document.createElement('textarea');
                el.className = 'form-control';
                el.id = `input-${input.name}`;
                el.value = input.default || '';
                group.appendChild(el);
            } else if (input.type === 'file') {
                // カスタマイズされたモダンで美しいファイルアップロードボタン
                const wrapper = document.createElement('div');
                wrapper.className = 'file-input-wrapper';
                wrapper.innerHTML = `
                    <button type="button" class="btn btn-outline" style="pointer-events: none;">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
                        <span id="file-name-${input.name}">Choose File...</span>
                    </button>
                    <input type="file" id="input-${input.name}" onchange="document.getElementById('file-name-${input.name}').textContent = this.files[0] ? this.files[0].name : 'Choose File...'" />
                `;
                group.appendChild(wrapper);
            } else {
                const el = document.createElement('input');
                el.type = input.type;
                el.className = 'form-control';
                el.id = `input-${input.name}`;
                el.value = input.default || '';
                group.appendChild(el);
            }
            
            container.appendChild(group);
        });
    }

    /**
     * 応答応答パネルの状態をリセットします。
     * 画像とオーディオの再生コンポーネントを非表示にし、メモリ リークを防ぐために BLOB キャッシュを完全に解放し、空の状態のプレースホルダーを復元します。
     */
    function resetResult() {
        document.getElementById('empty-state').style.display = 'flex';
        document.getElementById('loading-state').style.display = 'none';
        document.getElementById('result-text').style.display = 'none';
        document.getElementById('result-image').style.display = 'none';
        document.getElementById('result-audio').style.display = 'none';
        document.getElementById('result-audio').src = ''; // オーディオストリームをリリースする
        document.getElementById('result-text').textContent = '';
        document.getElementById('result-text').style.color = '';
        document.getElementById('response-status').textContent = '';
    }

    /**
     * 送信ボタンとリザルトパネルの読み込み待ち状態を切り替える
     * 
     * @param {boolean} isLoading リクエストが送信/処理されているかどうか
     */
    function setLoading(isLoading) {
        const btn = document.getElementById('run-btn');
        if (isLoading) {
            // ボタンが無効になり、読み込み中 アニメーションを読み込み中
            btn.disabled = true;
            btn.innerHTML = `<div class="spinner" style="width: 16px; height: 16px; border-width: 2px;"></div> Processing...`;
            document.getElementById('empty-state').style.display = 'none';
            document.getElementById('loading-state').style.display = 'flex';
            document.getElementById('result-text').style.display = 'none';
            document.getElementById('result-image').style.display = 'none';
            document.getElementById('result-audio').style.display = 'none';
            document.getElementById('result-audio').src = '';
            document.getElementById('response-status').textContent = 'Fetching...';
        } else {
            // [復元] ボタンの利用可能な状態と元の SVG 送信アイコン
            btn.disabled = false;
            btn.innerHTML = `<span>Send Request</span><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>`;
            document.getElementById('loading-state').style.display = 'none';
        }
    }

    /**
     * テスト API リクエストを実行するコアの非同期メソッド
     * フォーム入力を抽出し、それを GET Query、Multipart FormData、JSON、または Plain Text 形式に自動的にカプセル化して送信します。
     * 大規模言語モデルによって返されるデータは、テキスト ストリーミング レンダリング (isStream)、画像レンダリング (blob)、オーディオ メディア ストリーム (audio) の自動生成と再生、およびその他の機能をサポートします。
     */
    async function runTest() {
        if (!currentTest) return;
        
        // 実行中のリクエストがある場合は、強制的に中止します (複数回のクリックでリクエストが繰り返されるのを防ぐため)
        if (currentAbortController) {
            currentAbortController.abort();
        }
        // 新しい割り込み信号コントローラーを作成する
        currentAbortController = new AbortController();

        setLoading(true);

        let url = currentTest.url;
        let options = {
            method: currentTest.type,
            signal: currentAbortController.signal
        };

        try {
            // --- 1. ビルドリクエストパラメータの組み立て ---
            if (currentTest.type === 'GET') {
                // GET リクエスト: URLSearchParams を使用したスプライシング
                const params = new URLSearchParams();
                currentTest.inputs.forEach(inp => {
                    const val = document.getElementById(`input-${inp.name}`).value;
                    if (val) params.append(inp.name, val);
                });
                if ([...params].length > 0) url += `?${params.toString()}`;
            } else if (currentTest.isFormData) {
                // FormData リクエスト: 写真、オーディオ、ビデオなどのアップロードされたマルチメディア ファイルと付随するパラメータを処理します。
                const formData = new FormData();
                currentTest.inputs.forEach(inp => {
                    const el = document.getElementById(`input-${inp.name}`);
                    if (inp.type === 'file') {
                        if (el.files[0]) formData.append(inp.name, el.files[0]);
                    } else {
                        formData.append(inp.name, el.value);
                    }
                });
                options.body = formData;
            } else if (currentTest.isJson) {
                // JSON オブジェクト リクエスト: Content-Type ヘッダーを設定し、JSON 検証とシリアル化を実行します。
                options.headers = { 'Content-Type': 'application/json' };
                const rawVal = document.getElementById(`input-body`).value;
                try {
                    JSON.parse(rawVal); // フォーマットチェック
                    options.body = rawVal;
                } catch (e) {
                    throw new Error('Invalid JSON input format');
                }
            } else if (currentTest.isBody) {
                // プレーンテキスト本文リクエスト
                options.headers = { 'Content-Type': 'text/plain' };
                options.body = document.getElementById(`input-body`).value;
            }

            // 非同期ネットワークリクエストを送信する
            const response = await fetch(url, options);
            // インターフェースから返されたHTTP応答ステータスコードとステータス情報を表示します。
            document.getElementById('response-status').textContent = `${response.status} ${response.statusText}`;

            // --- 2. レンダリング応答結果を受信して​​解析します ---
            if (currentTest.isStream) {
                // [ストリーミング応答レンダリング]: 読み込みを非表示にし、ストリーミング テキスト ストリームを動的に読み取り、リアルタイムでテキスト ボックスを押し、スクロール バーをロールバックします。
                document.getElementById('loading-state').style.display = 'none';
                const textEl = document.getElementById('result-text');
                textEl.style.display = 'block';
                textEl.textContent = '';
                
                const reader = response.body.getReader();
                const decoder = new TextDecoder();
                
                while (true) {
                    const { value, done } = await reader.read();
                    if (done) break;
                    // 復号化された UTF-8 文字ブロックの追加レンダリング
                    textEl.textContent += decoder.decode(value, { stream: true });
                    // コンテナは自動的に一番下までスクロールして戻り、最新の出力が表示されます。
                    textEl.parentElement.scrollTop = textEl.parentElement.scrollHeight;
                }
            } else if (currentTest.responseType === 'blob') {
                // [マルチモーダル画像レンダリング]: 応答を直接バイナリ BLOB に変換し、画像コンポーネントをポイントしてレンダリングするオブジェクト URL を生成します。
                const blob = await response.blob();
                const imgUrl = URL.createObjectURL(blob);
                document.getElementById('result-image').src = imgUrl;
                document.getElementById('result-image').style.display = 'block';
            } else if (currentTest.responseType === 'audio') {
                // [マルチモーダル TTS 音声再生適応]: AI 合成サウンド バイト ストリームをオーディオ BLOB にカプセル化し、HTML5 プレーヤーに与え、自動的に再生を試みます
                const blob = await response.blob();
                const audioUrl = URL.createObjectURL(blob);
                const audioEl = document.getElementById('result-audio');
                audioEl.src = audioUrl;
                audioEl.style.display = 'block';
                // 最新のブラウザーのセキュリティ ルールによって制限されている場合は、自動再生を実行し、警告ログを印刷します。
                audioEl.play().catch(e => console.log('Audio autoplay blocked or failed', e));
            } else {
                // [テキストと JSON 応答]: 返された文字列を JSON 表示用に整形してフォーマットします。それ以外の場合は、テキストを直接表示します。
                const text = await response.text();
                const textEl = document.getElementById('result-text');
                textEl.style.display = 'block';
                
                try {
                    const jsonObj = JSON.parse(text);
                    textEl.textContent = JSON.stringify(jsonObj, null, 2);
                } catch {
                    textEl.textContent = text;
                }
            }
        } catch (err) {
            // ユーザーが積極的に中断した AbortError をキャプチャし、エラーを報告せずに直接返します。そうしないと、ユーザーに赤色のエラーが表示されます。
            if (err.name === 'AbortError') return;
            document.getElementById('result-text').style.display = 'block';
            document.getElementById('result-text').textContent = `Error: ${err.message}`;
            document.getElementById('result-text').style.color = '#f87171'; // 赤のエラーメッセージ
        } finally {
            setLoading(false);
        }
    }

    // Initialize
    document.addEventListener('DOMContentLoaded', initSidebar);

    /**
     * ナビゲーション バー グループの折りたたみ/展開状態を切り替えます。
     * @param {HTMLElement} titleEl がクリックした nav-title 要素
     */
    function toggleSection(titleEl) {
        const sectionEl = titleEl.parentElement;
        sectionEl.classList.toggle('collapsed');
    }
    ```

  - `style.css`: フロントエンドダッシュボードのスタイルシート。

    ```css
    :root {
        --primary: #6366f1;
        --primary-hover: #4f46e5;
        --secondary: #ec4899;
        --bg-color: #f1f5f9;
        --surface: #ffffff;
        --surface-glass: rgba(255, 255, 255, 0.7);
        --text-main: #0f172a;
        --text-muted: #64748b;
        --sidebar-bg: rgba(15, 23, 42, 0.85);
        --border: rgba(226, 232, 240, 0.8);
        --radius-lg: 20px;
        --radius-md: 12px;
        --shadow-sm: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        --shadow-lg: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
    }

    body {
        margin: 0;
        font-family: 'Inter', sans-serif;
        background: var(--bg-color);
        background-image: 
            radial-gradient(at 0% 0%, hsla(253,16%,7%,1) 0, transparent 50%), 
            radial-gradient(at 50% 0%, hsla(225,39%,30%,1) 0, transparent 50%), 
            radial-gradient(at 100% 0%, hsla(339,49%,30%,1) 0, transparent 50%);
        color: var(--text-main);
        display: flex;
        height: 100vh;
        overflow: hidden;
    }

    /* Sidebar Styling */
    .sidebar {
        width: 280px;
        background: var(--sidebar-bg);
        backdrop-filter: blur(24px);
        -webkit-backdrop-filter: blur(24px);
        color: white;
        display: flex;
        flex-direction: column;
        border-right: 1px solid rgba(255, 255, 255, 0.1);
        z-index: 10;
    }

    .logo-container {
        padding: 32px 24px;
        font-size: 24px;
        font-weight: 700;
        letter-spacing: -0.5px;
        display: flex;
        align-items: center;
        gap: 12px;
        background: linear-gradient(135deg, #818cf8 0%, #e879f9 100%);
        -webkit-background-clip: text;
        background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .nav-menu-container {
        flex: 1;
        overflow-y: auto;
        padding-bottom: 24px;
    }

    /* Custom Scrollbar for Nav Menu */
    .nav-menu-container::-webkit-scrollbar {
        width: 6px;
    }
    .nav-menu-container::-webkit-scrollbar-track {
        background: transparent;
    }
    .nav-menu-container::-webkit-scrollbar-thumb {
        background: rgba(255, 255, 255, 0.12);
        border-radius: 3px;
        transition: background 0.2s ease;
    }
    .nav-menu-container::-webkit-scrollbar-thumb:hover {
        background: rgba(255, 255, 255, 0.25);
    }

    .nav-section {
        padding: 0 16px;
        margin-bottom: 20px;
    }

    .nav-title {
        font-size: 12px;
        text-transform: uppercase;
        letter-spacing: 1px;
        color: #94a3b8;
        margin-bottom: 12px;
        padding: 6px 12px;
        font-weight: 600;
        cursor: pointer;
        display: flex;
        justify-content: space-between;
        align-items: center;
        user-select: none;
        border-radius: 6px;
        transition: background 0.2s ease, color 0.2s ease;
    }

    .nav-title:hover {
        background: rgba(255, 255, 255, 0.05);
        color: #f8fafc;
    }

    /* Accordion Fold Logic with CSS transitions */
    .nav-items-wrapper {
        max-height: 800px;
        overflow: hidden;
        transition: max-height 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    }

    .nav-section.collapsed .nav-items-wrapper {
        max-height: 0;
    }

    /* Chevron indicator animation */
    .chevron-icon {
        color: #64748b;
        transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1), color 0.3s ease;
    }

    .nav-title:hover .chevron-icon {
        color: #cbd5e1;
    }

    .nav-section.collapsed .chevron-icon {
        transform: rotate(-90deg);
    }

    .nav-item {
        padding: 12px 16px;
        margin-bottom: 4px;
        border-radius: 8px;
        cursor: pointer;
        transition: all 0.2s ease;
        font-size: 14px;
        font-weight: 500;
        color: #cbd5e1;
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .nav-item:hover {
        background: rgba(255, 255, 255, 0.1);
        color: white;
    }

    .nav-item.active {
        background: var(--primary);
        color: white;
        box-shadow: 0 4px 12px rgba(99, 102, 241, 0.4);
    }

    /* Main Content */
    .main-wrapper {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: 32px;
        overflow-y: auto;
        position: relative;
    }

    .header {
        margin-bottom: 32px;
    }

    .header h1 {
        margin: 0;
        font-size: 28px;
        font-weight: 700;
        color: white;
        text-shadow: 0 2px 4px rgba(0,0,0,0.5);
    }

    .header p {
        margin: 8px 0 0 0;
        color: rgba(255, 255, 255, 0.7);
        font-size: 15px;
    }

    .content-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 24px;
        align-items: start;
    }

    @media (max-width: 1024px) {
        .content-grid {
            grid-template-columns: 1fr;
        }
    }

    /* Cards */
    .card {
        background: var(--surface-glass);
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);
        border: 1px solid var(--border);
        border-radius: var(--radius-lg);
        padding: 24px;
        box-shadow: var(--shadow-lg);
    }

    .card h3 {
        margin: 0 0 20px 0;
        font-size: 18px;
        color: var(--text-main);
        display: flex;
        align-items: center;
        gap: 8px;
    }

    /* Form Elements */
    .form-group {
        margin-bottom: 20px;
    }

    .form-label {
        display: block;
        margin-bottom: 8px;
        font-size: 14px;
        font-weight: 500;
        color: var(--text-muted);
    }

    .form-control {
        width: 100%;
        padding: 12px 16px;
        border: 1px solid var(--border);
        border-radius: var(--radius-md);
        background: rgba(255, 255, 255, 0.9);
        font-family: inherit;
        font-size: 15px;
        color: var(--text-main);
        transition: all 0.2s ease;
        box-sizing: border-box;
    }

    .form-control:focus {
        outline: none;
        border-color: var(--primary);
        box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2);
    }

    textarea.form-control {
        min-height: 120px;
        resize: vertical;
        line-height: 1.5;
        font-family: 'JetBrains Mono', monospace;
        font-size: 13px;
    }

    .file-input-wrapper {
        position: relative;
        overflow: hidden;
        display: inline-block;
        width: 100%;
    }

    .file-input-wrapper input[type=file] {
        font-size: 100px;
        position: absolute;
        left: 0;
        top: 0;
        opacity: 0;
        cursor: pointer;
    }

    .btn {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        gap: 8px;
        padding: 12px 24px;
        border: none;
        border-radius: var(--radius-md);
        font-size: 15px;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        font-family: inherit;
        width: 100%;
    }

    .btn-primary {
        background: linear-gradient(135deg, var(--primary) 0%, #818cf8 100%);
        color: white;
        box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
    }

    .btn-primary:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 16px rgba(99, 102, 241, 0.4);
    }

    .btn-primary:active {
        transform: translateY(0);
    }

    .btn-primary:disabled {
        opacity: 0.7;
        cursor: not-allowed;
        transform: none;
    }

    .btn-outline {
        background: transparent;
        border: 1px solid var(--border);
        color: var(--text-main);
    }

    .btn-outline:hover {
        background: rgba(0,0,0,0.02);
    }

    /* Result Area */
    .result-container {
        position: relative;
        min-height: 200px;
        background: #1e293b;
        border-radius: var(--radius-md);
        padding: 20px;
        overflow-x: auto;
        color: #f8fafc;
        box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);
    }

    .result-output {
        margin: 0;
        font-family: 'JetBrains Mono', Consolas, monospace;
        font-size: 14px;
        line-height: 1.6;
        white-space: pre-wrap;
        word-wrap: break-word;
    }

    .result-image {
        max-width: 100%;
        border-radius: 8px;
        display: none;
        box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    }

    .result-audio {
        width: 100%;
        border-radius: 8px;
        display: none;
        box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    }

    .empty-state {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100%;
        color: #64748b;
        text-align: center;
        padding: 40px 0;
    }

    .spinner {
        width: 24px;
        height: 24px;
        border: 3px solid rgba(255,255,255,0.1);
        border-radius: 50%;
        border-top-color: white;
        animation: spin 1s linear infinite;
    }

    .endpoint-badge {
        display: inline-block;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 11px;
        font-weight: 700;
        font-family: monospace;
        background: #e2e8f0;
        color: #475569;
        margin-bottom: 16px;
    }

    .endpoint-badge.get { background: #dbeafe; color: #1d4ed8; }
    .endpoint-badge.post { background: #dcfce3; color: #15803d; }

    @keyframes spin {
        to { transform: rotate(360deg); }
    }

    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(10px); }
        to { opacity: 1; transform: translateY(0); }
    }

    .fade-in {
        animation: fadeIn 0.4s cubic-bezier(0.4, 0, 0.2, 1) forwards;
    }
    ```

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
