# Quarkus Example

AiSpect seamlessly integrates with Quarkus through CDI (Contexts and Dependency Injection). You can enable AI capabilities in your application simply by using `@Inject`.

This example (located in the `examples/example-quarkus` directory) demonstrates how to integrate AiSpect in a Quarkus application and exposes various REST APIs for code reviews, phase testing, and data type testing.

---

## 1. Example Architecture

Here is a complete list of all code files included in this example and their purposes:

- **Configuration**
  - `AiSpectConfig.java`: CDI configuration class that produces the `AiClient` and `AiOperations` beans.

- **Controllers (Resources)**
  - `CodeReviewResource.java`: REST endpoint for handling code review webhooks.
  - `PhaseTestResource.java`: REST endpoint for testing different interceptor phases.
  - `DataTypeTestResource.java`: REST endpoint for testing various data types.

- **Services**
  - `CodeReviewService.java`: Business logic and `@AiUnitAgent` definitions for code reviews.
  - `PhaseTestService.java`: Business logic and `@AiUnitAgent` definitions for phase testing.
  - `DataTypeTestService.java`: Business logic and `@AiUnitAgent` definitions for data type handling.

- **Resources**
  - `application.properties`: Quarkus application configuration.

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
