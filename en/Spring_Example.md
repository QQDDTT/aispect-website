# Spring Boot Example

AiSpect provides the ability to seamlessly integrate with Spring Boot, allowing you to interact with large language models simply by defining Java interfaces, without writing complex HTTP requests and Prompt concatenation code.

This example (located in the `examples/example-spring-boot` directory) demonstrates how to quickly build an API Dashboard with capabilities for testing different interceptor "Phases" and various "Data Types".

---

## 1. Example Architecture

Here is a complete list of all code files included in this example and their purposes:

- **Application Entry**
  - `Application.java`: The Spring Boot startup class.

- **Controllers**
  - `TestController.java`: Provides REST/Web endpoints, including all Phase, Data Type, and Graph testing interfaces.

- **Contract Interfaces (Services)**
  - `PhaseTestService.java`: Interfaces for testing different interceptor phases (Before, After, Exception, All).
  - `DataTypeTestService.java`: Interfaces for testing different data types (String, POJO, Collection, Audio, Video).
  - `GraphTestService.java`: Interfaces for testing Agent Graph orchestrations.
  - `AiSpectErrorTestService.java`: Interfaces for testing error handling and fallbacks.

- **Service Implementations**
  - `PhaseTestServiceImpl.java`, `DataTypeTestServiceImpl.java`, `GraphTestServiceImpl.java`, `AiSpectErrorTestServiceImpl.java`: The native implementations (fallback/stub logic) for the respective services.

- **Agents**
  - `SummaryAfterAgent.java`: An agent to summarize text after a method execution.
  - `CollectionProcessAgent.java`: An agent to categorize items in a collection.
  - `PojoProcessAgent.java`: An agent to process complex POJO structures.
  - `AudioProcessAgent.java`, `VideoProcessAgent.java`, `TextToSpeechAgent.java`, `VideoStoryAgent.java`: Agents for handling multi-modal tasks like audio and video processing.

- **Configuration & DTOs**
  - `AiGraphConfig.java`: Configuration class for defining Agent Graph workflows.
  - `User.java`: A DTO used to demonstrate POJO binding.

- **Resources & Frontend**
  - `application.properties`: Spring Boot application configuration.
  - `index.html`: Provides a modern, dark-themed interactive API dashboard.
  - `app.js`: Frontend logic for configuring requests and previewing results (JSON formatting, SSE streaming).
  - `style.css`: Styles for the frontend dashboard.

---

In a Spring Boot application using AiSpect, your `Service` interface can gain AI capabilities simply by using specific annotations. For instance, processing POJOs or complex collections:

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.example.spring.agents.PojoProcessAgent;
import com.aispect.example.spring.dto.User;
import java.util.List;
import java.util.Map;

public interface DataTypeTestService {

    // POJO processing: Proxied to PojoProcessAgent
    @AiUnitAgent(name = PojoProcessAgent.class, description = "Updates the status of a user object")
    User processPojo(User user);
    
    // Collection processing
    @AiUnitAgent(description = "Categorizes a list of items into fruits and vegetables")
    Map<String, List<String>> processCollection(List<String> items);
    
    // ... other data types and phase testing
}
```

The framework's AOP interceptor automatically intercepts the execution of methods with `@AiUnitAgent`. Upon invocation, it automatically extracts parameters, executes communication with the LLM, and returns your desired strongly-typed objects (like POJOs, Lists, Maps, or even Streams and byte[]).

---

## 3. Running the Experience

### Step 1: Configure API Key
Inject your model API key via environment variables or startup parameters:
```bash
export GEMINI_API_KEY="YOUR_API_KEY_HERE"
```

### Step 2: Start the Project
Run the Gradle command in the root directory to start Spring Boot:
```bash
./gradlew :example-spring-boot:bootRun
```

### Step 3: Access the Page
Open a browser and navigate to `http://localhost:8080`. You will see the new **AiSpect API Dashboard**. Use the sidebar to switch between different test cases:
- **Phase Tests**: Experience intercepting and intervening in Before, After, and Exception phases.
- **Data Type Tests**: Experience the framework's ability to handle Strings, POJOs (JSON), Collections, Images (Multi-modal), and Streams (SSE streaming) with automatic type binding.
