# Java SE 实战案例

AiSpect 可以在没有任何依赖注入容器（如 Spring 或 Quarkus）的纯 Java SE 环境中使用。您可以手动初始化引擎并为标准的 Java 接口创建代理。

本案例（位于 `examples/example-java-se` 目录）展示了如何手动初始化框架，并涵盖了“阶段拦截（Phase）测试”和“数据类型（Data Type）测试”。

---

## 1. 案例架构

以下是本案例中包含的所有代码文件及其用途的完整列表：

- **应用入口与核心逻辑**
  - `Main.java`: 单文件中包含了完整的案例。它包括 `PhaseTestService` 和 `DataTypeTestService` 接口及其桩实现，自定义的智能体（如 `SummaryAfterAgent` 和 `CollectionClassifierAgent`），用于无 API 密钥演示的 `MockAiClient`，以及手动构建 `AiSpect` 引擎并创建代理的 `main` 编排方法。

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
     * Java SE 示例应用入口。
     * <p>
     * 演示在纯 Java SE 环境（无 Spring / Quarkus 容器）下如何使用 AiSpect 框架：
     * 通过 {@link AiSpect#builder()} 手动初始化引擎，使用 {@link AiSpect#proxy} 方法
     * 为普通接口创建代理，并自动拦截带有 {@code @AiUnitAgent} 注解的方法。
     * </p>
     *
     * <p>本示例覆盖以下功能场景：</p>
     * <ol>
     *   <li>Phase 测试：BEFORE、AFTER、EXCEPTION、ALL 四个执行阶段</li>
     *   <li>数据类型测试：String 清洗、集合分类、图像处理</li>
     * </ol>
     */
    public class Main {

        public static void main(String[] args) throws Exception {
            printBanner();

            // 获取环境变量中的 API Key
            String apiKey = System.getenv("GEMINI_API_KEY");
            if (apiKey == null || apiKey.trim().isEmpty()) {
                System.err.println("WARNING: GEMINI_API_KEY environment variable is not set.");
                System.err.println("The AI client will return mock responses for demonstration.\n");
            }

            // 根据是否配置了 API Key，选择使用真实的 OpenAI/Gemini 适配器，还是使用 Mock 客户端
            AiClient aiClient = (apiKey != null && !apiKey.trim().isEmpty())
                    ? new OpenAiAdapter()
                    : new MockAiClient();

            // 构建 AiSpect 引擎，注入 AI 客户端
            AiSpect engine = AiSpect.builder()
                    .aiClient(aiClient)
                    .build();

            // ================================================================
            // 场景 1: Phase 测试
            // ================================================================
            System.out.println("=== 场景 1: Phase 测试 ===\n");

            PhaseTestService rawPhaseService = new PhaseTestServiceImpl();
            PhaseTestService phaseService = engine.proxy(rawPhaseService, PhaseTestService.class);

            // BEFORE 阶段：内容合规审查
            System.out.println("[BEFORE Phase] 输入: \"safe data\"");
            String beforeResult = phaseService.testBeforePhase("safe data");
            System.out.println("[BEFORE Phase] 结果: " + beforeResult + "\n");

            // AFTER 阶段：AI 自动总结方法返回值
            System.out.println("[AFTER Phase] 输入: \"dummy\"");
            String afterResult = phaseService.testAfterPhase("dummy");
            System.out.println("[AFTER Phase] 结果: " + afterResult + "\n");

            // EXCEPTION 阶段：异常降级
            System.out.println("[EXCEPTION Phase] 输入: \"error\" (触发异常降级)");
            String exceptionResult = phaseService.testExceptionPhase("error");
            System.out.println("[EXCEPTION Phase] 结果: " + exceptionResult + "\n");

            // ALL 阶段：AI 完全接管，从文本中提取 JSON
            System.out.println("[ALL Phase] 输入: \"用户名: Alice, 年龄: 25, 城市: 北京\"");
            String allResult = phaseService.testAllPhase("用户名: Alice, 年龄: 25, 城市: 北京");
            System.out.println("[ALL Phase] 结果: " + allResult + "\n");

            // ================================================================
            // 场景 2: 数据类型测试
            // ================================================================
            System.out.println("=== 场景 2: 数据类型测试 ===\n");

            DataTypeTestService rawDataService = new DataTypeTestServiceImpl();
            DataTypeTestService dataService = engine.proxy(rawDataService, DataTypeTestService.class);

            // String 清洗
            System.out.println("[DataType String] 输入: \"  Hello   World!!  \"");
            String cleanedStr = dataService.processString("  Hello   World!!  ");
            System.out.println("[DataType String] 结果: " + cleanedStr + "\n");

            // 集合分类
            List<String> items = List.of("苹果", "香蕉", "小轿车", "篮球", "橙子", "卡车", "足球", "葡萄");
            System.out.println("[DataType Collection] 输入: " + items);
            Map<String, List<String>> categorized = dataService.processCollection(items);
            System.out.println("[DataType Collection] 结果: " + categorized + "\n");

            System.out.println("=== 示例演示完成 ===");
        }

        // ====================================================================
        // PhaseTestService 接口与实现
        // ====================================================================

        /**
         * 阶段测试服务接口。
         * <p>
         * 配合 {@link AiUnitAgent} 注解，演示 AiSpect 框架在不同执行阶段
         * （BEFORE, AFTER, EXCEPTION, ALL）的拦截代理能力。
         * </p>
         */
        public interface PhaseTestService {

            /**
             * 测试前置阶段（BEFORE）：内容合规审查。
             * 在本地方法执行前，由 {@link ContentModerationAgent} 对输入文本进行安全过滤。
             *
             * @param input 待审核的输入文本
             * @return 原生业务逻辑处理产生的结果
             */
            @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "执行前进行内容合规审查")
            String testBeforePhase(String input);

            /**
             * 测试后置阶段（AFTER）：AI 对返回结果进行总结。
             * 原生方法先执行，其返回值再被 {@link SummaryAfterAgent} 智能提炼。
             *
             * @param input 测试输入文本
             * @return 由大模型总结后的文本
             */
            @AiUnitAgent(value = SummaryAfterAgent.class, executePhase = AiExecutePhase.AFTER, description = "执行后对生成文本进行总结提取")
            String testAfterPhase(String input);

            /**
             * 测试异常降级阶段（EXCEPTION）：异常容灾。
             * 当原生方法抛出异常时，{@link FallbackAgent} 自动接管并返回降级结果。
             *
             * @param input 触发异常的测试输入（传 "error" 触发异常）
             * @return 降级结果文本
             */
            @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "异常时触发降级回退机制")
            String testExceptionPhase(String input);

            /**
             * 测试全权接管阶段（ALL）：AI 完全替代本地方法。
             * {@link JsonExtractorAgent} 直接从输入文本中提取并返回 JSON，本地方法被跳过。
             *
             * @param input 含有潜在 JSON 信息的输入文本
             * @return 大模型提取出的格式化 JSON 字符串
             */
            @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "在所有生命周期环节提取并解析JSON数据")
            String testAllPhase(String input);
        }

        /**
         * PhaseTestService 的本地原生实现（桩实现）。
         */
        public static class PhaseTestServiceImpl implements PhaseTestService {

            @Override
            public String testBeforePhase(String input) {
                return "Original method executed successfully with input: " + input;
            }

            @Override
            public String testAfterPhase(String input) {
                // AI 会拦截此方法的返回值并进行总结
                return "The quick brown fox jumps over the lazy dog. This is a very long and detailed sentence that needs to be summarized by the AI agent in the AFTER phase.";
            }

            @Override
            public String testExceptionPhase(String input) {
                // 模拟运行时异常
                if ("error".equalsIgnoreCase(input)) {
                    throw new RuntimeException("Simulated internal server error in testExceptionPhase");
                }
                return "No error occurred.";
            }

            @Override
            public String testAllPhase(String input) {
                // AI 的 ALL 阶段会直接接管，此处不会被执行
                return "This text should not be returned, because ALL phase agent handles it.";
            }
        }

        // ====================================================================
        // DataTypeTestService 接口与实现
        // ====================================================================

        /**
         * 数据类型测试服务接口。
         * <p>
         * 演示 AiSpect 框架对不同数据类型（字符串、集合、图像）的处理能力。
         * </p>
         */
        public interface DataTypeTestService {

            /**
             * 清洗并格式化输入文本字符串。
             * {@link DataCleanAgent} 通过 ALL 阶段接管，将原始文本交给大模型做智能清洗。
             *
             * @param text 待清洗的原始文本
             * @return 清洗、纠错或格式化后的字符串
             */
            @AiUnitAgent(value = DataCleanAgent.class, description = "清理字符串数据", executePhase = AiExecutePhase.ALL)
            String processString(String text);

            /**
             * 对集合数据（字符串列表）进行 AI 分类整理。
             * {@link CollectionClassifierAgent} 通过 ALL 阶段接管，将列表项按类别分组。
             *
             * @param items 待分类的字符串列表
             * @return 按类别分组的映射 Map
             */
            @AiUnitAgent(value = CollectionClassifierAgent.class, description = "对集合数据进行分类处理", executePhase = AiExecutePhase.ALL)
            Map<String, List<String>> processCollection(List<String> items);

            /**
             * 根据指令对图片进行多模态处理。
             * {@link ImageProcessAgent} 通过 ALL 阶段接管，直接处理图片字节数组。
             *
             * @param image       原始图片二进制字节数组
             * @param instruction 图片处理指令
             * @return 处理后的图片二进制字节数组
             */
            @AiUnitAgent(value = ImageProcessAgent.class, description = "根据指令处理图片", executePhase = AiExecutePhase.ALL)
            byte[] processImage(byte[] image, String instruction);
        }

        /**
         * DataTypeTestService 的本地原生实现（桩实现/降级逻辑）。
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
        // 自定义 Agent 实现
        // ====================================================================

        /**
         * 后置摘要智能体，在目标方法执行后调用大模型对结果进行总结。
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
         * 集合分类智能体，通过大模型对传入的列表项进行分类并返回 JSON 映射结果。
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
                String result = response.message().content().replace("
    ```

- **资源文件**
  - `simplelogger.properties`: SLF4J simple logger 的配置文件。

    ```properties
    org.slf4j.simpleLogger.defaultLogLevel=debug
    org.slf4j.simpleLogger.log.com.aispect=debug
    org.slf4j.simpleLogger.showDateTime=true
    org.slf4j.simpleLogger.dateTimeFormat=yyyy-MM-dd HH:mm:ss:SSS Z
    ```

---

## 2. 引擎初始化

在纯 Java SE 环境中，您可以使用 `AiSpect.builder()` 来构建引擎，然后使用 `proxy` 方法为您的接口创建代理。

```java
import com.aispect.api.AiClient;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import com.aispect.engine.se.AiSpect;

public class Main {
    public static void main(String[] args) throws Exception {
        // 1. 初始化 AI 客户端
        AiClient aiClient = new OpenAiAdapter();
        
        // 2. 构建 AiSpect 引擎
        AiSpect engine = AiSpect.builder()
                .aiClient(aiClient)
                .build();
                
        // 3. 为您的服务创建代理
        PhaseTestService rawService = new PhaseTestServiceImpl();
        PhaseTestService proxiedService = engine.proxy(rawService, PhaseTestService.class);
        
        // 4. 调用被代理的方法
        String result = proxiedService.testBeforePhase("safe data");
        System.out.println(result);
    }
}
```

## 2. 使用注解

就像在 Spring 或 Quarkus 中一样，您可以在接口方法上使用 `@AiUnitAgent` 注解来定义 AI 行为。

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.security.ContentModerationAgent;

public interface PhaseTestService {

    // 执行前进行内容合规审查
    @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "执行前进行内容合规审查")
    String testBeforePhase(String input);
}
```

---

## 3. 运行体验

### 步骤一：配置 API 密钥
将您的 API 密钥设置为环境变量：
```bash
export GEMINI_API_KEY="您的_API_KEY_在这里"
```

### 步骤二：运行应用
在项目根目录运行 Gradle 命令：
```bash
./gradlew :example-java-se:run
```
