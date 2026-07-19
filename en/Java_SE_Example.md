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
