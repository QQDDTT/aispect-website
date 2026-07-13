# Spring Boot Example

AiSpect provides the ability to seamlessly integrate with Spring Boot, allowing you to interact with large language models simply by defining Java interfaces, without writing complex HTTP requests and Prompt concatenation code.

This example (located in the `examples/example-spring-boot` directory) demonstrates how to quickly build a Web application with capabilities for "text polishing", "image processing", and "image storytelling".

---

## 1. Example Architecture

- **Controller (`DocumentController`)**: Provides REST/Web endpoints to handle text and file upload requests from the frontend.
- **Contract Interface (`AiEditorService`)**: The core of the business logic. No implementation class is needed; all methods are annotated with `@AiUnitAgent`, and the AiSpect engine dynamically proxies and calls the corresponding Agent at runtime.
- **Frontend Page (`index.html`)**: Provides a simple two-pane interactive interface, with editing/uploading on the left and a preview of AI results on the right.

---

In a Spring Boot application using AiSpect, your `Service` interface can gain AI capabilities simply by using specific annotations. Furthermore, you can write an implementation class for it to provide Fallback or localized processing logic:

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import com.aispect.agents.data.ImageProcessAgent;
import com.aispect.agents.data.ImageStoryAgent;

public interface AiEditorService {

    // Text processing: Proxied to DataCleanAgent
    @AiUnitAgent(name = DataCleanAgent.class, description = "Expands the given text with more details and richer vocabulary.")
    String processText(String originalText);
    
    // Image analysis: Proxied to ImageProcessAgent
    @AiUnitAgent(name = ImageProcessAgent.class, description = "Processes an image based on instructions.", modelName="imagen-4-generate")
    byte[] processImage(byte[] image, String instruction);

    // Image story: Proxied to ImageStoryAgent
    @AiUnitAgent(name = ImageStoryAgent.class, description = "Tells an imaginative story based on the image.", modelName="gemini-2.5-flash")
    String storyFromImage(byte[] image, String instruction);
}
```

The framework's AOP interceptor (e.g., `AgentInterceptor`) automatically intercepts the execution of methods with `@AiUnitAgent`. Upon invocation, it automatically extracts parameters (like converting `byte[]` to multi-modal input for the LLM) and executes communication with the LLM. If an exception occurs or the interception is explicitly rejected, it gracefully degrades to executing your local `AiEditorServiceImpl`.

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
Open a browser and navigate to `http://localhost:8080`. You can enter text in the left text box and click **Process** to experience text polishing, or upload an image and click **Tell Story** to experience multi-modal text and image generation capabilities.
