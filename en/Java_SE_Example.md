# Java SE Example

AiSpect can be used in a pure Java SE environment without any dependency injection containers like Spring or Quarkus. You can manually initialize the engine and create proxies for your standard Java interfaces.

This example (located in the `examples/example-java-se` directory) demonstrates how to initialize the framework manually and covers "Phase Testing" and "Data Type Testing".

---

## 1. Example Architecture

Here is a complete list of all code files included in this example and their purposes:

- **Application Entry & Core Logic**
  - `Main.java`: Contains the complete example in a single file. It includes the `PhaseTestService` and `DataTypeTestService` interfaces, their stub implementations, custom agents like `SummaryAfterAgent` and `CollectionClassifierAgent`, a `MockAiClient` for demonstration without API keys, and the `main` method that orchestrates everything by manually building the `AiSpect` engine and creating proxies.

- **Resources**
  - `simplelogger.properties`: Configuration file for the SLF4J simple logger.

---

## 2. Engine Initialization

In a pure Java SE environment, you use `AiSpect.builder()` to build the engine, and then use the `proxy` method to create proxies for your interfaces.

```java
import com.aispect.api.AiClient;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import com.aispect.engine.se.AiSpect;

public class Main {
    public static void main(String[] args) throws Exception {
        // 1. Initialize AI Client
        AiClient aiClient = new OpenAiAdapter();
        
        // 2. Build AiSpect engine
        AiSpect engine = AiSpect.builder()
                .aiClient(aiClient)
                .build();
                
        // 3. Create proxy for your service
        PhaseTestService rawService = new PhaseTestServiceImpl();
        PhaseTestService proxiedService = engine.proxy(rawService, PhaseTestService.class);
        
        // 4. Call the proxied method
        String result = proxiedService.testBeforePhase("safe data");
        System.out.println(result);
    }
}
```

## 2. Using Annotations

Just like in Spring or Quarkus, you can use the `@AiUnitAgent` annotation on your interface methods to define AI behaviors.

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.security.ContentModerationAgent;

public interface PhaseTestService {

    // Content compliance check before execution
    @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "Perform content compliance check before execution")
    String testBeforePhase(String input);
}
```

---

## 3. Running the Experience

### Step 1: Configure API Key
Set your API key as an environment variable:
```bash
export GEMINI_API_KEY="YOUR_API_KEY_HERE"
```

### Step 2: Run the Application
Run the Gradle command in the root directory:
```bash
./gradlew :example-java-se:run
```


## Complete Source Code

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
 * Java SE sample application entry.
 * <p>
 * Demonstrates how to use the AiSpect framework in a pure Java SE environment (without Spring / Quarkus container):
 * Manually initialize the engine via {@link AiSpect#builder()}, using the {@link AiSpect#proxy} method
 * Create a proxy for a common interface and automatically intercept methods annotated with {@code @AiUnitAgent}.
 * </p>
 *
 * <p>This example covers the following functional scenarios:</p>
 * <ol>
 *   <li>Phase test: four execution stages of BEFORE, AFTER, EXCEPTION and ALL</li>
 *   <li>Data type testing: String cleaning, collection classification, image processing</li>
 * </ol>
 */
public class Main {

    public static void main(String[] args) throws Exception {
        printBanner();

        // Get the API Key in the environment variable
        String apiKey = System.getenv("GEMINI_API_KEY");
        if (apiKey == null || apiKey.trim().isEmpty()) {
            System.err.println("WARNING: GEMINI_API_KEY environment variable is not set.");
            System.err.println("The AI client will return mock responses for demonstration.\n");
        }

        // Depending on whether the API Key is configured, choose to use the real OpenAI/Gemini adapter or use the Mock client
        AiClient aiClient = (apiKey != null && !apiKey.trim().isEmpty())
                ? new OpenAiAdapter()
                : new MockAiClient();

        // Build the AiSpect engine and inject the AI ​​client
        AiSpect engine = AiSpect.builder()
                .aiClient(aiClient)
                .build();

        // ================================================================
        // Scenario 1: Phase test
        // ================================================================
        System.out.println("=== Scenario 1: Phase Test ===\n");

        PhaseTestService rawPhaseService = new PhaseTestServiceImpl();
        PhaseTestService phaseService = engine.proxy(rawPhaseService, PhaseTestService.class);

        // BEFORE Phase: Content Compliance Review
        System.out.println("[BEFORE Phase] input: \"safe data\"");
        String beforeResult = phaseService.testBeforePhase("safe data");
        System.out.println("[BEFORE Phase] Result:" + beforeResult + "\n");

        // AFTER stage: AI automatically summarizes method return values
        System.out.println("[AFTER Phase] input: \"dummy\"");
        String afterResult = phaseService.testAfterPhase("dummy");
        System.out.println("[AFTER Phase] Result:" + afterResult + "\n");

        // EXCEPTION stage: abnormal degradation
        System.out.println("[EXCEPTION Phase] Input: \"error\" (触发异常降级)");
        String exceptionResult = phaseService.testExceptionPhase("error");
        System.out.println("[EXCEPTION Phase] Result:" + exceptionResult + "\n");

        // ALL stage: AI takes over completely, extracting JSON from text
        System.out.println("[ALL Phase] Input: \"用户名: Alice, 年龄: 25, 城市: 北京\"");
        String allResult = phaseService.testAllPhase("Username: Alice, Age: 25, City: Beijing");
        System.out.println("[ALL Phase] Result:" + allResult + "\n");

        // ================================================================
        // Scenario 2: Data type testing
        // ================================================================
        System.out.println("=== Scenario 2: Data type testing ===\n");

        DataTypeTestService rawDataService = new DataTypeTestServiceImpl();
        DataTypeTestService dataService = engine.proxy(rawDataService, DataTypeTestService.class);

        // String cleaning
        System.out.println("[DataType String] Input: \"  Hello   World!!  \"");
        String cleanedStr = dataService.processString("  Hello   World!!  ");
        System.out.println("[DataType String] Result:" + cleanedStr + "\n");

        // Collection classification
        List<String> items = List.of("apple", "香蕉", "小轿车", "篮球", "橙子", "卡车", "足球", "葡萄");
        System.out.println("[DataType Collection] input:" + items);
        Map<String, List<String>> categorized = dataService.processCollection(items);
        System.out.println("[DataType Collection] Result:" + categorized + "\n");

        System.out.println("=== Sample demonstration completed ===");
    }

    // ====================================================================
    // PhaseTestService interface and implementation
    // ====================================================================

    /**
     * Stage test service interface.
     * <p>
     * With the {@link AiUnitAgent} annotation, demonstrate the AiSpect framework in different execution stages
     * (BEFORE, AFTER, EXCEPTION, ALL) interception proxy capabilities.
     * </p>
     */
    public interface PhaseTestService {

        /**
         * Testing pre-phase (BEFORE): content compliance review.
         * The input text is securely filtered by {@link ContentModerationAgent} before the local method is executed.
         *
         * @param input input text to be reviewed
         * @return The result generated by native business logic processing
         */
        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "Conduct content compliance review before execution")
        String testBeforePhase(String input);

        /**
         * Post-testing phase (AFTER): AI summarizes the returned results.
         * The native method is executed first, and its return value is intelligently refined by {@link SummaryAfterAgent}.
         *
         * @param input test input text
         * @return text summarized by the large model
         */
        @AiUnitAgent(value = SummaryAfterAgent.class, executePhase = AiExecutePhase.AFTER, description = "After execution, summarize and extract the generated text.")
        String testAfterPhase(String input);

        /**
         * Test exception degradation stage (EXCEPTION): exception disaster recovery.
         * When a native method throws an exception, {@link FallbackAgent} automatically takes over and returns the fallback result.
         *
         * @param input test input that triggers the exception (pass "error" to trigger the exception)
         * @return downgrade result text
         */
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "Trigger downgrade fallback mechanism when exception occurs")
        String testExceptionPhase(String input);

        /**
         * Test full takeover phase (ALL): AI completely replaces local methods.
         * {@link JsonExtractorAgent} extracts and returns JSON directly from the input text, native methods are skipped.
         *
         * @param input Input text containing underlying JSON information
         * @return Formatted JSON string extracted from the large model
         */
        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "Extract and parse JSON data at all lifecycle stages")
        String testAllPhase(String input);
    }

    /**
     * Local native implementation (stub implementation) of PhaseTestService.
     */
    public static class PhaseTestServiceImpl implements PhaseTestService {

        @Override
        public String testBeforePhase(String input) {
            return "Original method executed successfully with input: " + input;
        }

        @Override
        public String testAfterPhase(String input) {
            // AI will intercept the return value of this method and summarize it
            return "The quick brown fox jumps over the lazy dog. This is a very long and detailed sentence that needs to be summarized by the AI agent in the AFTER phase.";
        }

        @Override
        public String testExceptionPhase(String input) {
            // Simulation runtime exception
            if ("error".equalsIgnoreCase(input)) {
                throw new RuntimeException("Simulated internal server error in testExceptionPhase");
            }
            return "No error occurred.";
        }

        @Override
        public String testAllPhase(String input) {
            // The ALL phase of AI will take over directly and will not be executed here.
            return "This text should not be returned, because ALL phase agent handles it.";
        }
    }

    // ====================================================================
    // DataTypeTestService interface and implementation
    // ====================================================================

    /**
     * Data type testing service interface.
     * <p>
     * Demonstrates the AiSpect framework's ability to handle different data types (strings, collections, images).
     * </p>
     */
    public interface DataTypeTestService {

        /**
         * Cleans and formats input text strings.
         * {@link DataCleanAgent} takes over the ALL stage and hands the original text to the large model for intelligent cleaning.
         *
         * @param text The original text to be cleaned
         * @return Cleaned, error-corrected or formatted string
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "Clean string data", executePhase = AiExecutePhase.ALL)
        String processString(String text);

        /**
         * Perform AI classification on collection data (list of strings).
         * {@link CollectionClassifierAgent} takes over via the ALL stage to group list items into categories.
         *
         * @param items List of strings to be classified
         * @return Map grouped by category
         */
        @AiUnitAgent(value = CollectionClassifierAgent.class, description = "Classify collection data", executePhase = AiExecutePhase.ALL)
        Map<String, List<String>> processCollection(List<String> items);

        /**
         * Perform multimodal processing of images according to instructions.
         * {@link ImageProcessAgent} takes over through the ALL stage and directly processes the image byte array.
         *
         * @param image original image binary byte array
         * @param instruction image processing instruction
         * @return processed image binary byte array
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "Process images according to instructions", executePhase = AiExecutePhase.ALL)
        byte[] processImage(byte[] image, String instruction);
    }

    /**
     * Local native implementation of DataTypeTestService (stub implementation/downgrade logic).
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
    // Custom Agent implementation
    // ====================================================================

    /**
     * The post-summary agent calls the large model to summarize the results after the target method is executed.
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
     * A collection of classification agents that classifies incoming list items through a large model and returns JSON mapping results.
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
    // Mock AI client (used when there is no API Key)
    // ====================================================================

    /**
     * Simulated AI client, used for demonstration and downgrade when no API Key is provided.
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
    // Tool method
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

