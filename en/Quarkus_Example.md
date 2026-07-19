# Quarkus Example

AiSpect seamlessly integrates with Quarkus through CDI (Contexts and Dependency Injection). You can enable AI capabilities in your application simply by using `@Inject`.

This example (located in the `examples/example-quarkus` directory) demonstrates how to integrate AiSpect in a Quarkus application and exposes various REST APIs for code reviews, phase testing, and data type testing.

---

## 1. Example Architecture

Here is a complete list of all code files included in this example and their purposes:

- **Configuration**
  - `AiSpectConfig.java`: CDI configuration class that produces the `AiClient` and `AiOperations` beans.

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
     * AiSpect core CDI configuration classes.
     * <p>
     * 负责生产 {@link AiClient} 和 {@link AiOperations} 两个核心 Bean，
     * 供 {@link com.aispect.engine.quarkus.AiAgentCdiInterceptor} 通过 {@code @Inject} 使用。
     * </p>
     * <p>
     * Read the {@code GEMINI_API_KEY} environment variable first. If not set, the built-in Mock client is used for demonstrations.
     * </p>
     */
    @ApplicationScoped
    public class AiSpectConfig {

        /**
         * Produces the {@link AiClient} Bean.
         * <p>
         * If {@code GEMINI_API_KEY} is configured, use {@link OpenAiAdapter} (compatible with Gemini OpenAI-compatible interface);
         * Otherwise the built-in Mock implementation is returned for demonstration purposes.
         * </p>
         *
         * @return AiClient instance
         */
        @Produces
        @ApplicationScoped
        public AiClient aiClient() {
            String apiKey = System.getenv("GEMINI_API_KEY");
            if (apiKey != null && !apiKey.trim().isEmpty()) {
                return new OpenAiAdapter();
            }
            // When the API Key is not configured, the Mock client is used and a fixed demo response is returned.
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
         * Produce {@link AiOperations} beans.
         * <p>
         * Wrap {@link AiClient} into the standard {@link AiOperations} interface,
         * Used by interceptors to dispatch large model requests at runtime.
         * </p>
         *
         * @param aiClient AiClient instance injected by CDI
         * @return AiOperations instance
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

- **Controllers (Resources)**
  - `CodeReviewResource.java`: REST endpoint for handling code review webhooks.

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

  - `PhaseTestResource.java`: REST endpoint for testing different interceptor phases.

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

  - `DataTypeTestResource.java`: REST endpoint for testing various data types.

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

- **Services**
  - `CodeReviewService.java`: Business logic and `@AiUnitAgent` definitions for code reviews.

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

  - `PhaseTestService.java`: Business logic and `@AiUnitAgent` definitions for phase testing.

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

        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "Conduct content compliance review before execution")
        public String testBeforePhase(String input) {
            return "Original method executed successfully with input: " + input;
        }

        // In Quarkus, we reuse the basic flow agents. Note that we don't have SummaryAfterAgent in core agents,
        // so we'll just demonstrate the BEFORE, EXCEPTION, and ALL phases which use core agents.
        
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "Trigger downgrade fallback mechanism when exception occurs")
        public String testExceptionPhase(String input) {
            if ("error".equalsIgnoreCase(input)) {
                throw new RuntimeException("Simulated internal server error in testExceptionPhase");
            }
            return "No error occurred.";
        }

        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "Extract and parse JSON data at all lifecycle stages")
        public String testAllPhase(String input) {
            return "This text should not be returned, because ALL phase agent handles it.";
        }
    }
    ```

  - `DataTypeTestService.java`: Business logic and `@AiUnitAgent` definitions for data type handling.

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

        @AiUnitAgent(value = DataCleanAgent.class, description = "Clean string data", executePhase = AiExecutePhase.ALL)
        public String processString(String text) {
            return text;
        }

        @AiUnitAgent(value = ImageProcessAgent.class, description = "Process images according to instructions", executePhase = AiExecutePhase.ALL)
        public byte[] processImage(byte[] image, String instruction) {
            return image;
        }
    }
    ```

- **Resources**
  - `application.properties`: Quarkus application configuration.

    ```properties
    quarkus.http.port=8081
    quarkus.log.category."com.aispect".level=DEBUG
    ```

---

## 2. Engine Configuration

In Quarkus, you provide the `AiClient` and `AiOperations` beans via a CDI configuration class.

```java
import com.aispect.api.AiClient;
import com.aispect.api.AiOperations;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Produces;

@ApplicationScoped
public class AiSpectConfig {

    // Produce the AiClient bean
    @Produces
    @ApplicationScoped
    public AiClient aiClient() {
        return new OpenAiAdapter();
    }
    
    // Produce the AiOperations bean
    @Produces
    @ApplicationScoped
    public AiOperations aiOperations(AiClient aiClient) {
        // Wrap the AiClient into AiOperations for the interceptors
        return new AiOperationsWrapper(aiClient);
    }
}
```

## 3. Using Annotations

Simply annotate your CDI bean methods with `@AiUnitAgent`. The Quarkus interceptor will automatically handle the AI logic.

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class CodeReviewService {

    // Analyze the code diff and return a JSON summary
    @AiUnitAgent(value = DataCleanAgent.class, description = "Analyzes the code diff and returns a JSON summary of defects and suggestions.")
    public String reviewDiff(String diffContent) {
        // Fallback in case AI processing fails
        return "{ \"status\": \"fallback\" }";
    }
}
```

## 4. Injecting into Resources

You can then inject this service into your REST endpoints using standard JAX-RS.

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
        // The call is automatically intercepted by AiSpect
        return codeReviewService.reviewDiff(diff);
    }
}
```

---

## 5. Running the Experience

### Step 1: Configure API Key
Set your API key as an environment variable:
```bash
export GEMINI_API_KEY="YOUR_API_KEY_HERE"
```

### Step 2: Start Quarkus in Dev Mode
Run the Gradle command in the root directory:
```bash
./gradlew :example-quarkus:quarkusDev
```
