# Spring Boot Example

AiSpect provides the ability to seamlessly integrate with Spring Boot, allowing you to interact with large language models simply by defining Java interfaces, without writing complex HTTP requests and Prompt concatenation code.

This example (located in the `examples/example-spring-boot` directory) demonstrates how to quickly build an API Dashboard with capabilities for testing different interceptor "Phases" and various "Data Types".

---

## 1. Example Architecture

Here is a complete list of all code files included in this example and their purposes:

- **Application Entry**
  - `Application.java`: The Spring Boot startup class.

    ```java
    package com.aispect.example.spring;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    import com.aispect.engine.spring.annotation.EnableAiSpect;

    /**
     * Spring Boot application startup class
     * Include the @EnableAiSpect annotation to enable the AiSpect engine
     */
    @SpringBootApplication
    @EnableAiSpect
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```

- **Controllers**
  - `TestController.java`: Provides REST/Web endpoints, including all Phase, Data Type, and Graph testing interfaces.

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
     * Test controller, providing test interfaces for various AiSpect functions
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
         * Testing pre-phase (Before Phase)
         * 
         * @param input input string
         * @return pre-processed string
         */
        @GetMapping("/phase/before")
        public String phaseBefore(@RequestParam(defaultValue = "safe data") String input) {
            return phaseTestService.testBeforePhase(input);
        }

        /**
         * After Phase
         * 
         * @param input input string
         * @return post-summary processed string
         */
        @GetMapping("/phase/after")
        public String phaseAfter(@RequestParam(defaultValue = "dummy") String input) {
            return phaseTestService.testAfterPhase(input);
        }

        /**
         * Test exception phase (Exception Phase)
         * 
         * @param input input string
         * @return The result of exception downgrade or normal processing
         */
        @GetMapping("/phase/exception")
        public String phaseException(@RequestParam(defaultValue = "error") String input) {
            return phaseTestService.testExceptionPhase(input);
        }

        /**
         * Testing the full life cycle stage (All Phase)
         * 
         * @param text input text
         * @return Full life cycle processing results
         */
        @PostMapping("/phase/all")
        public String phaseAll(@RequestBody String text) {
            return phaseTestService.testAllPhase(text);
        }

        // -- Data Type Tests --

        /**
         * Test string data type handling
         * 
         * @param text input text
         * @return AI-cleaned string
         */
        @PostMapping("/datatype/string")
        public String typeString(@RequestBody String text) {
            return dataTypeTestService.processString(text);
        }

        /**
         * Test POJO data type handling
         * 
         * @param user user object
         * @return the user object after the state has been modified by AI
         */
        @PostMapping(value = "/datatype/pojo", consumes = MediaType.APPLICATION_JSON_VALUE)
        public User typePojo(@RequestBody User user) {
            return dataTypeTestService.processPojo(user);
        }

        /**
         * Test collection data type handling
         * 
         * @param items string collection
         * @return Collection mapping after AI classification
         */
        @PostMapping(value = "/datatype/collection")
        public Map<String, List<String>> typeCollection(@RequestBody List<String> items) {
            return dataTypeTestService.processCollection(items);
        }

        /**
         * Test image data type handling
         * 
         * @param file image file
         * @param instruction processing instruction
         * @return AI processed image byte array
         * @throws Exception Exception
         */
        @PostMapping("/datatype/image")
        public byte[] typeImage(@RequestParam("image") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processImage(file.getBytes(), instruction);
        }

        /**
         * Test audio data type handling
         * 
         * @param file audio file
         * @param instruction analysis instruction
         * @return Text description analyzed by AI
         * @throws Exception Exception
         */
        @PostMapping("/datatype/audio")
        public String typeAudio(@RequestParam("audio") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processAudio(file.getBytes(), file.getContentType(), instruction);
        }

        /**
         * Test video data type handling
         * 
         * @param file video file
         * @param instruction analysis instruction
         * @return Text description analyzed by AI
         * @throws Exception Exception
         */
        @PostMapping("/datatype/video")
        public String typeVideo(@RequestParam("video") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processVideo(file.getBytes(), file.getContentType(), instruction);
        }

        /**
         * In-depth analysis of test image content
         * 
         * @param file image file
         * @param instruction description instruction
         * @return Image analysis text description
         * @throws Exception Exception
         */
        @PostMapping("/datatype/image-analyze")
        public String analyzeImage(@RequestParam("image") MultipartFile file,
                @RequestParam("instruction") String instruction) throws Exception {
            return dataTypeTestService.analyzeImage(file.getBytes(), instruction);
        }

        /**
         * Test text-to-speech (TTS)
         * 
         * @param text input string text
         * @return synthesized sound audio binary stream
         */
        @PostMapping(value = "/datatype/tts", produces = "audio/wav")
        public byte[] typeTts(@RequestBody String text) {
            return dataTypeTestService.textToSpeech(text);
        }
    }
    ```

- **Contract Interfaces (Services)**
  - `PhaseTestService.java`: Interfaces for testing different interceptor phases (Before, After, Exception, All).

    ```java
    package com.aispect.example.spring.service;

    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agents.security.ContentModerationAgent;
    import com.aispect.agents.flow.FallbackAgent;
    import com.aispect.agents.data.JsonExtractorAgent;
    import com.aispect.example.spring.agents.SummaryAfterAgent;

    /**
     * Stage test service interface.
     * <p>
     * With the {@link AiUnitAgent} annotation, demonstrate and test the AiSpect framework in different execution aspects stages (BEFORE, AFTER, EXCEPTION, ALL)
     * interception proxy, context transfer and large model processing access capabilities.
     * </p>
     */
    public interface PhaseTestService {

        /**
         * Test the processing interception of the front aspect (BEFORE stage).
         * <p>
         * Bind {@link ContentModerationAgent} to trigger AI filtering before local native methods are executed.
         * Large models will perform pre-compliance and security review of input text. If the review fails, an exception can be intercepted and thrown in advance.
         * </p>
         *
         * @param input input text content to be reviewed
         * @return After pre-review, the text results generated by native business logic processing
         */
        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "Conduct content compliance review before execution")
        String testBeforePhase(String input);

        /**
         * Test processing interception of post-aspect (AFTER stage).
         * <p>
         * Bind {@link SummaryAfterAgent}. The native method (stub implementation) will be executed first to get the result,
         * The returned results are then automatically passed to AI as input parameters, and the large model performs subsequent intelligent summary, refinement, and rewriting.
         * </p>
         *
         * @param input The original text string of the test input
         * @return The final text result summarized and refined by the large model
         */
        @AiUnitAgent(value = SummaryAfterAgent.class, executePhase = AiExecutePhase.AFTER, description = "After execution, summarize and extract the generated text.")
        String testAfterPhase(String input);

        /**
         * Test the capture and rollback of exception degradation aspects (EXCEPTION phase).
         * <p>
         * Bind {@link FallbackAgent}. When a native business method occurs during operation and throws an uncaught exception,
         * The interceptor will automatically intercept exceptions and hand over the context and exception information to the large model for intelligent disaster recovery and return and degradation response.
         * </p>
         *
         * @param input test input text that triggers an exception or test rollback
         * @return After an exception occurs, the text of the degradation result returned by the large model disaster recovery mechanism
         */
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "Trigger downgrade fallback mechanism when exception occurs")
        String testExceptionPhase(String input);

        /**
         * Test the handling of full takeover aspects (ALL phase).
         * <p>
         * Bind {@link JsonExtractorAgent}. The large model directly replaces and takes over the execution logic of the native method.
         * Native local methods will be skipped directly, and AI will directly analyze, extract and assemble the formatted JSON string result from the input text.
         * </p>
         *
         * @param input Input text containing underlying JSON information or extraction requirements
         * @return The large model directly takes over and parses the extracted formatted JSON string
         */
        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "Extract and parse JSON data at all lifecycle stages")
        String testAllPhase(String input);
    }
    ```

  - `DataTypeTestService.java`: Interfaces for testing different data types (String, POJO, Collection, Audio, Video).

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
     * Data type testing service interface
     * <p>
     * Demonstrates the various data types handled and supported by the AiSpect framework (including Pojos, List collections, multimedia image binaries,
     * Audio binary, video binary, speech synthesis, etc.), demonstrating multimodality processing capabilities and Agent orchestration mechanism.
     * </p>
     */
    public interface DataTypeTestService {

        /**
         * Cleans and formats input text string data.
         * <p>
         * Bind {@link DataCleanAgent}, take over all stages (ALL stage), and pass the original text to the large model for intelligent cleaning.
         * </p>
         *
         * @param text The original text string to be cleaned
         * @return String result after cleaning, error correction or formatting
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "Clean string data", executePhase = AiExecutePhase.ALL)
        String processString(String text);

        /**
         * Process user object data and update its state properties.
         * <p>
         * Bind {@link PojoProcessAgent} to take over through all phases (ALL phase).
         * The method contains the AOP `proceed` mechanism: first run the name plus prefix and status initialization business logic in the local `DataTypeTestServiceImpl`,
         * The processed `User` is then automatically converted into JSON and handed over to AI for age increment and status mark upgrade.
         * </p>
         *
         * @param user User transmission object entity, including name, age, initial status, etc.
         * @return The complete User object after local processing and intelligent annotation of the large model
         */
        @AiUnitAgent(value = PojoProcessAgent.class, description = "Process user object data and update status", executePhase = AiExecutePhase.ALL)
        User processPojo(User user);

        /**
         * Classify the collection data (string list) and return it in the mapping structure of Key-Value Map.
         * <p>
         * Bind {@link CollectionProcessAgent} to take over through all phases (ALL phase).
         * Demonstrates the ability of a large model to extract a collection of unstructured strings and organize them into structured categories according to topics or attributes.
         * </p>
         *
         * @param items Collection of string lists to be classified
         * @return Classification result mapping Map after grouping by category
         */
        @AiUnitAgent(value = CollectionProcessAgent.class, description = "Classify collection data", executePhase = AiExecutePhase.ALL)
        Map<String, List<String>> processCollection(List<String> items);

        /**
         * According to the specified editing instructions, perform multi-modal intelligent modifications to the input image binary data.
         * <p>
         * Bind {@link ImageProcessAgent} to take over through all phases (ALL phase).
         * The large model supports processing of input image byte arrays and directly returns the generated image byte arrays.
         * </p>
         *
         * @param image binary byte array of the original image
         * @param instruction text instruction that instructs the large model how to process or modify images
         * @return Modified or processed image binary byte array
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "Process images according to instructions", executePhase = AiExecutePhase.ALL)
        byte[] processImage(byte[] image, String instruction);

        /**
         * Multimodally analyze audio content and answer according to instructions.
         * <p>
         * Bind {@link AudioProcessAgent} and specify the usage model as {@code gemini-3.5-flash}.
         * Demonstrates the ability of a large model to directly read an audio binary stream and understand its speech, melody, or environmental context.
         * </p>
         *
         * @param audio binary byte array of audio (such as PCM / WAV format data)
         * @param mimeType MIME media type of audio (such as {@code audio/wav}, {@code audio/mp3}, etc.)
         * @param instruction prompt word instruction that requires the large model to analyze the audio
         * @return Text answers or transcription results extracted from large model analysis
         */
        @AiUnitAgent(value = AudioProcessAgent.class, description = "Analyze audio content", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String processAudio(byte[] audio, String mimeType, String instruction);

        /**
         * Multi-modally analyze video content and answer according to instructions.
         * <p>
         * Bind {@link VideoProcessAgent} and specify the usage model as {@code gemini-3.5-flash}.
         * Demonstrates the large model's ability to directly correlate and understand cross-modal frames and sound streams of video files.
         * </p>
         *
         * @param video binary byte array of video file (such as MP4 format data)
         * @param mimeType The MIME media type of the video (e.g. {@code video/mp4})
         * @param instruction prompt word instruction that requires the large model to analyze the video
         * @return Text answers or summaries extracted from large model analysis
         */
        @AiUnitAgent(value = VideoProcessAgent.class, description = "Analyze video content", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String processVideo(byte[] video, String mimeType, String instruction);

        /**
         * Automatically interpret videos in multiple modalities to generate structured scene descriptions and narrative summaries.
         * <p>
         * Bind {@link VideoStoryAgent} and specify the usage model as {@code gemini-3.5-flash}.
         * Without the need to pass in instructions, the large model automatically describes the overall theme, key scenes and sound effects/dialogue of the video in time sequence.
         * Demonstrate comprehensive capabilities in multi-modal video understanding and content creation.
         * </p>
         *
         * @param video binary byte array of video file (such as MP4 format data)
         * @param mimeType The MIME media type of the video (e.g. {@code video/mp4})
         * @return Video scene description and narrative summary text output by the large model
         */
        @AiUnitAgent(value = VideoStoryAgent.class, description = "Automatically generate video scene descriptions and narrative summaries", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String describeVideo(byte[] video, String mimeType);

        /**
         * Multi-modally analyze image content and return analysis instructions directly in text format.
         * <p>
         * Bind {@link ImageProcessAgent} and specify the usage model as {@code gemini-3.5-flash}.
         * Demonstrates the large model's ability to extract scenes, text (OCR), and entities within images and generate descriptions.
         * </p>
         *
         * @param image binary byte array of the original image
         * @param instruction prompt word instruction to extract and describe
         * @return Text description or analysis results of the image
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "Analyze image content and return text description", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String analyzeImage(byte[] image, String instruction);

        /**
         * Convert input text into high-fidelity sound audio through large models.
         * <p>
         * Bind {@link TextToSpeechAgent} and specify to use the native speech synthesis large model {@code gemini-3.1-flash-tts-preview}.
         * Demonstrates the ability to generate text into an audio binary stream through a large model multi-modal TTS interface and output it directly.
         * </p>
         *
         * @param text Input string text to be converted into sound
         * @return Binary byte array of the generated sound audio file (such as MP3 format audio byte stream)
         */
        @AiUnitAgent(value = TextToSpeechAgent.class, description = "Convert text to sound audio", modelName = "gemini-3.1-flash-tts-preview", executePhase = AiExecutePhase.ALL)
        byte[] textToSpeech(String text);

        /**
         * Bypassing the framework proxy annotation, support reflection or direct calling of the original User processing interface.
         *
         * @param user user transfer object entity
         * @return processed result object
         */
        Object processPojo(Object user);
    }
    ```

  - `GraphTestService.java`: Interfaces for testing Agent Graph orchestrations.

    ```java
    package com.aispect.example.spring.service;

    import java.util.Map;

    import com.aispect.agent.annotation.AiGraphAgent;
    import com.aispect.agent.annotation.AiNode;
    import com.aispect.agents.graph.DynamicWorkflowGraphAgent;

    /**
     * Test the workflow orchestration service interface of Graph Agent.
     * <p>
     * This interface cooperates with {@link AiGraphAgent} and binds {@link DynamicWorkflowGraphAgent} topology graph agent.
     * AI collaborative generation workflow for managing and scheduling a set of data-dependent multi-steps (steps identified by multiple {@link AiNode}).
     * The process includes: generating a story outline -> writing a story based on the outline -> summarizing the written story.
     * </p>
     */
    @AiGraphAgent(name = "workflowGraphAgent", value = DynamicWorkflowGraphAgent.class, description = "A generic workflow agent that orchestrates a series of tasks")
    public interface GraphTestService {

        /**
         * Generate an outline of the story based on the thematic background (step one).
         * <p>
         * Corresponds to the `generateOutline` node in the workflow and relies on the input parameter {@code "initialTopic"} in the global context (globalContext).
         * The generated story outline will be saved back into the global context as input for the next step.
         * </p>
         *
         * @param globalContext Context mapping Map shared globally by the process, which should contain at least the key {@code "initialTopic"}
         * @return generated synopsis text content
         */
        @AiNode(name = "generateOutline", description = "Given the global context which contains 'initialTopic', generate a story outline and return it.")
        String generateOutline(Map<String, Object> globalContext);

        /**
         * Create a short and complete story based on the generated story outline framework (step 2).
         * <p>
         * Corresponding to the `writeStory` node in the workflow, its input source is {@code "generateOutline_result"} generated in the previous step.
         * The resulting story will be stored in the global context.
         * </p>
         *
         * @param globalContext Context mapping Map shared globally by the process, which should contain at least the key {@code "generateOutline_result"}
         * @return short story text content created
         */
        @AiNode(name = "writeStory", description = "Given the global context which contains 'generateOutline_result', write a short story based on the outline and return it.")
        String writeStory(Map<String, Object> globalContext);

        /**
         * Intelligently summarize the story generated in the previous stage (step 3).
         * <p>
         * Corresponding to the `summarizeStory` node in the workflow, its input source is {@code "writeStory_result"} generated in the second step.
         * The final summary of the story is returned as the final output of the entire workflow or stored in the global context.
         * </p>
         *
         * @param globalContext Context mapping Map shared globally by the process, which should contain at least the key {@code "writeStory_result"}
         * @return A one-sentence summary text of the story
         */
        @AiNode(name = "summarizeStory", description = "Given the global context which contains 'writeStory_result', summarize the story in one sentence and return it.")
        String summarizeStory(Map<String, Object> globalContext);
    }
    ```

  - `AiSpectErrorTestService.java`: Interfaces for testing error handling and fallbacks.

    ```java
    package com.aispect.example.spring.service;

    import com.aispect.agent.annotation.AiNode;
    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agents.data.DataCleanAgent;

    /**
     * Service interface used to test abnormal information during the scanning period of AiSpect components.
     * <p>
     * This interface specifically includes a series of configuration usage of Agent and Node that can trigger {@link com.aispect.common.exception.AiScanException} errors.
     * It is designed to verify and test the framework's robust checking and friendly error prompting capabilities for annotation conflicts and illegal configurations during the Bootstrap (startup scan verification) phase.
     * </p>
     */
    public interface AiSpectErrorTestService {

        /**
         * Scenario 1: Test the Unit Agent conflict of repeatedly binding the same execution phase (default ALL phase) on the same method.
         * <p>
         * When multiple {@link AiUnitAgent} are annotated on the same method, they must specify different execution phases (executePhase).
         * If not specified (implicitly defaulting to ALL stage) or explicitly set to the same stage, a same-stage conflict verification error will be triggered.
         * </p>
         *
         * @param input The original text string of the test input
         * @return The response text processed by the agent. If an error is reported when normal scanning is started, it cannot be executed to this point.
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "Agent 1: Default stage")
        @AiUnitAgent(value = DataCleanAgent.class, description = "Agent 2: Also the default stage")
        String testSamePhaseConflict(String input);

        /**
         * Scenario 2: Test for conflicts mixing full-phase takeover (ALL phase) with local specific phases (such as BEFORE phase) on the same method.
         * <p>
         * {@link AiExecutePhase#ALL} means completely taking over the execution logic of the target method, which means any other phases (such as BEFORE or AFTER)
         * Can't live with it. If the Agent in both the ALL phase and the BEFORE/AFTER phase is marked at the same time, the framework will fail to schedule and trigger a scan period conflict exception.
         * </p>
         *
         * @param input The original text string of the test input
         * @return processed response result
         */
        @AiUnitAgent(value = DataCleanAgent.class, executePhase = AiExecutePhase.ALL, description = "Full stage takeover")
        @AiUnitAgent(value = DataCleanAgent.class, executePhase = AiExecutePhase.BEFORE, description = "Pre-stage processing")
        String testAllPhaseMixedConflict(String input);

        /**
         * Scenario 3: Test the {@link AiNode} conflict of repeatedly binding the same execution phase (default phase) on the same method.
         * <p>
         * When the method belongs to a Graph node and is bound to multiple {@link AiNode} annotations, if these nodes use the same execution phase,
         * AiNode's same-phase conflict verification error will be triggered when the framework starts scanning.
         * </p>
         *
         * @param input The original text string of the test input
         * @return the result after processing by the workflow node
         */
        @AiNode(name = "node1", description = "Node 1: Default stage")
        @AiNode(name = "node2", description = "Node 2: Also the default stage")
        String testNodeSamePhaseConflict(String input);

        /**
         * Scenario 4: Testing conflict of mixing ALL full-phase takeover of {@link AiNode} with a specific phase (BEFORE phase).
         * <p>
         * Similar to the Unit Agent, the ALL phase of AiNode cannot be mixed with other local execution phases and configured on the same method.
         * Otherwise, the single-node scheduling route of the Graph workflow will be destroyed and a component scan period conflict error will be triggered.
         * </p>
         *
         * @param input The original text string of the test input
         * @return the result after processing by the workflow node
         */
        @AiNode(name = "nodeAll", executePhase = AiExecutePhase.ALL, description = "Full stage takeover")
        @AiNode(name = "nodeBefore", executePhase = AiExecutePhase.BEFORE, description = "Pre-stage processing")
        String testNodeAllPhaseMixedConflict(String input);

    }
    ```

- **Service Implementations**
  - `PhaseTestServiceImpl.java`, `DataTypeTestServiceImpl.java`, `GraphTestServiceImpl.java`, `AiSpectErrorTestServiceImpl.java`: The native implementations (fallback/stub logic) for the respective services.

    ```java
    package com.aispect.example.spring.service;

    import org.springframework.stereotype.Service;

    /**
     * Stage test service implementation class
     * Implemented test method logic for different life cycle stages
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
     * Data type testing service implementation class
     * Provides default implementations of methods for different data types (strings, POJOs, collections, images, streams, etc.)
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
     * Test the service implementation of Graph Agent
     * Contains multiple @AiNode that can be scheduled by DynamicWorkflowGraphAgent
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

- **Agents**
  - `SummaryAfterAgent.java`: An agent to summarize text after a method execution.

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
     * post-summary agent
     * Inherit AbstractUnitAgent to call the large model to summarize the results after the target method is executed.
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

  - `CollectionProcessAgent.java`: An agent to categorize items in a collection.

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
     * collection processing agent
     * Intelligent classification of incoming list items through large models and returns JSON mapping results
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

  - `PojoProcessAgent.java`: An agent to process complex POJO structures.

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
     * POJO processing agent
     * Call the large model to modify the properties of the user object (User) and return the updated object
     */
    public class PojoProcessAgent extends AbstractUnitAgent<Object, User> {
        private static final ObjectMapper mapper = new ObjectMapper();

        @Override
        public User execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            // 1. Explicitly call the local original method business logic (AOP proceed mechanism)
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

  - `AudioProcessAgent.java`, `VideoProcessAgent.java`, `TextToSpeechAgent.java`, `VideoStoryAgent.java`: Agents for handling multi-modal tasks like audio and video processing.

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
     * Audio processing Agent intercepts the method and encapsulates the audio byte[] and its mimeType into inlineData and sends it to the large model.
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
     * Video processing agent intercepts the method and encapsulates the video byte[] and its mimeType into inlineData and sends it to the large model.
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
     * Speech synthesis agent converts text prompt words into audio.
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
            
            // If response.message() returns inlineData, it is returned directly as audio bytes
            if (response.message().inlineData() != null) {
                return response.message().inlineData();
            }
            
            // Bottom line: Returns the byte array corresponding to the text content
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
     * Watch Video Storytelling Agent.
     * <p>
     * The Agent receives a piece of video binary data and uses a large multi-modal model to automatically analyze the video images and sounds.
     * and generates detailed scene descriptions and narrative summaries without requiring additional instructions from the user.
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

            Message userMsg = new Message(Role.USER, "Please describe the content and scenes of this video in detail.", null, null, videoBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

- **Configuration & DTOs**
  - `AiGraphConfig.java`: Configuration class for defining Agent Graph workflows.

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
     * Configuration class for manually configuring Graph Agent related beans in the sample application.
     */
    @Configuration
    public class AiGraphConfig {

        /**
         * Register AiGraphAgentSpringLauncher.
         * Responsible for collecting all @AiGraphAgent when Spring starts and managing the life cycle.
         *
         * @param aiOperations The core AI client operation interface, provided by the underlying automatic assembly
         * @return launcher instance
         */
        @Bean
        public AiGraphAgentSpringLauncher aiGraphAgentSpringLauncher(AiOperations aiOperations) {
            return new AiGraphAgentSpringLauncher(aiOperations);
        }

        /**
         * Register the Graph Agent's HTTP route.
         * Here, it is left to the business layer to freely customize the routing path and request method, and distribute the execution tasks to the Launcher.
         *
         * @param launcher Graph launcher
         * @return dynamically registered endpoint route
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

  - `User.java`: A DTO used to demonstrate POJO binding.

    ```java
    package com.aispect.example.spring.dto;

    /**
     * User Data Transfer Object (DTO)
     * Used to test the delivery and processing of entity classes in AiSpect
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

- **Resources & Frontend**
  - `application.properties`: Spring Boot application configuration.

    ```properties
    logging.level.com.aispect=DEBUG

    # AiSpect AI Framework Configuration
    aispect.provider=gemini
    aispect.default-model=gemini-2.5-flash
    # You can set the API key directly here, or inject it via environment variables (e.g., from .env)
    aispect.api-key=${GEMINI_API_KEY:}
    ```

  - `index.html`: Provides a modern, dark-themed interactive API dashboard.

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

  - `app.js`: Frontend logic for configuring requests and previewing results (JSON formatting, SSE streaming).

    ```javascript
    /**
     * AiSpect demo control panel core JavaScript script
     * Responsible for rendering the left test navigation bar, dynamically constructing request configuration forms, initiating AOP interaction requests, and parsing and rendering various multi-modal AI responses.
     */

    // Configuration registry for all API test scenarios
    const testConfigs = [
        // --- Phase Tests (Aspect life cycle phase tests) ---
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
        
        // --- Data Type Tests (multimodal and different data type tests) ---
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
        
        // --- Graph Tests (Topology graph workflow orchestration test) ---
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

    // Currently selected test configuration item
    let currentTest = null;
    // Controller for canceling requests to prevent users from repeatedly submitting and concurrency conflicts
    let currentAbortController = null;

    /**
     * Initialize left sidebar navigation items
     * Traverse the configuration registry, dynamically create button elements and bind click selection events, and mount the groups to the corresponding DOM container.
     */
    function initSidebar() {
        const phaseNav = document.getElementById('nav-phase');
        const datatypeNav = document.getElementById('nav-datatype');
        const graphNav = document.getElementById('nav-graph');

        testConfigs.forEach(test => {
            const el = document.createElement('div');
            el.className = 'nav-item';
            // Using SVG Chevron Right Arrow Icon
            el.innerHTML = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"></polyline></svg> ${test.title}`;
            el.onclick = () => selectTest(test.id);
            el.id = `nav-${test.id}`;

            // Mount to the corresponding section by group
            if (test.group === 'phase') phaseNav.appendChild(el);
            else if (test.group === 'datatype') datatypeNav.appendChild(el);
            else if (test.group === 'graph') graphNav.appendChild(el);
        });
    }

    /**
     * Select the event handler of a test item
     * Activates the current navigation state, resetting the right content panel, form fields, and previously generated results area.
     * 
     * @param {string} id The unique ID of the test item
     */
    function selectTest(id) {
        // Remove the active style from all navigation items and assign active to the newly selected item
        document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
        document.getElementById(`nav-${id}`).classList.add('active');

        // Obtain and record the current test item configuration
        currentTest = testConfigs.find(t => t.id === id);
        
        // Update page title and description information
        document.getElementById('page-title').textContent = currentTest.title;
        document.getElementById('page-desc').textContent = `Testing endpoint: ${currentTest.url}`;
        
        // Render API type flag badge
        const badge = document.getElementById('endpoint-info');
        badge.textContent = `${currentTest.type} ${currentTest.url}`;
        badge.className = `endpoint-badge ${currentTest.type.toLowerCase()}`;

        // Dynamically render request parameter configuration form
        renderForm();
        // Clear the Response data generated by the last request
        resetResult();
        // Show the main grid container on the right
        document.getElementById('main-content').style.display = 'grid';
    }

    /**
     * Dynamically build and render the Request input form based on the current test configuration
     * Supports rendering text boxes (text), long text boxes (textarea), and file upload buttons (file).
     */
    function renderForm() {
        const container = document.getElementById('form-fields');
        container.innerHTML = ''; // Clear old form fields

        currentTest.inputs.forEach(input => {
            const group = document.createElement('div');
            group.className = 'form-group';
            
            const label = document.createElement('label');
            label.className = 'form-label';
            label.textContent = input.label;
            group.appendChild(label);

            // Select and render the corresponding form control according to the input type
            if (input.type === 'textarea') {
                const el = document.createElement('textarea');
                el.className = 'form-control';
                el.id = `input-${input.name}`;
                el.value = input.default || '';
                group.appendChild(el);
            } else if (input.type === 'file') {
                // Customized modern and beautiful file upload button
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
     * Reset the state of the Response response panel
     * Hide image and audio playback components, completely free the blob cache to prevent memory leaks, and restore empty state placeholders.
     */
    function resetResult() {
        document.getElementById('empty-state').style.display = 'flex';
        document.getElementById('loading-state').style.display = 'none';
        document.getElementById('result-text').style.display = 'none';
        document.getElementById('result-image').style.display = 'none';
        document.getElementById('result-audio').style.display = 'none';
        document.getElementById('result-audio').src = ''; // release audio stream
        document.getElementById('result-text').textContent = '';
        document.getElementById('result-text').style.color = '';
        document.getElementById('response-status').textContent = '';
    }

    /**
     * Switch the Loading waiting state of the send button and the result panel
     * 
     * @param {boolean} isLoading Whether the request is being sent/processed
     */
    function setLoading(isLoading) {
        const btn = document.getElementById('run-btn');
        if (isLoading) {
            // Button disabled, loading Loading animation
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
            // Restore button available state and original SVG send icon
            btn.disabled = false;
            btn.innerHTML = `<span>Send Request</span><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>`;
            document.getElementById('loading-state').style.display = 'none';
        }
    }

    /**
     * Core asynchronous methods that execute test API requests
     * Extract form input, automatically encapsulate it into GET Query, Multipart FormData, JSON or Plain Text format and send it.
     * The data returned by the large language model supports: text streaming rendering (isStream), image rendering (blob), automatic generation and playback of audio media streams (audio) and other functions.
     */
    async function runTest() {
        if (!currentTest) return;
        
        // If there is an executing request, forcefully abort it (to prevent repeated requests on multiple clicks)
        if (currentAbortController) {
            currentAbortController.abort();
        }
        // Create a new interrupt signal controller
        currentAbortController = new AbortController();

        setLoading(true);

        let url = currentTest.url;
        let options = {
            method: currentTest.type,
            signal: currentAbortController.signal
        };

        try {
            // --- 1. Assemble build request parameters ---
            if (currentTest.type === 'GET') {
                // GET requests: Splicing using URLSearchParams
                const params = new URLSearchParams();
                currentTest.inputs.forEach(inp => {
                    const val = document.getElementById(`input-${inp.name}`).value;
                    if (val) params.append(inp.name, val);
                });
                if ([...params].length > 0) url += `?${params.toString()}`;
            } else if (currentTest.isFormData) {
                // FormData request: Process uploaded multimedia files such as pictures/audio and videos and accompanying parameters
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
                // JSON object request: Set Content-Type header and perform JSON validation and serialization
                options.headers = { 'Content-Type': 'application/json' };
                const rawVal = document.getElementById(`input-body`).value;
                try {
                    JSON.parse(rawVal); // Format check
                    options.body = rawVal;
                } catch (e) {
                    throw new Error('Invalid JSON input format');
                }
            } else if (currentTest.isBody) {
                // Plain text Body request
                options.headers = { 'Content-Type': 'text/plain' };
                options.body = document.getElementById(`input-body`).value;
            }

            // Send an asynchronous network request
            const response = await fetch(url, options);
            // Display the HTTP response status code and status information returned by the interface
            document.getElementById('response-status').textContent = `${response.status} ${response.statusText}`;

            // --- 2. Receive and parse rendering response results ---
            if (currentTest.isStream) {
                // [Streaming response rendering]: Hide Loading, dynamically read the streaming text stream and push the text box in real time and roll back the scroll bar
                document.getElementById('loading-state').style.display = 'none';
                const textEl = document.getElementById('result-text');
                textEl.style.display = 'block';
                textEl.textContent = '';
                
                const reader = response.body.getReader();
                const decoder = new TextDecoder();
                
                while (true) {
                    const { value, done } = await reader.read();
                    if (done) break;
                    // Append rendering of decrypted UTF-8 character blocks
                    textEl.textContent += decoder.decode(value, { stream: true });
                    // The container automatically scrolls back to the bottom to show the latest output
                    textEl.parentElement.scrollTop = textEl.parentElement.scrollHeight;
                }
            } else if (currentTest.responseType === 'blob') {
                // [Multi-modal image rendering]: Convert the response directly into a binary blob, and generate an object URL to point to and render the image component
                const blob = await response.blob();
                const imgUrl = URL.createObjectURL(blob);
                document.getElementById('result-image').src = imgUrl;
                document.getElementById('result-image').style.display = 'block';
            } else if (currentTest.responseType === 'audio') {
                // [Multi-modal TTS voice playback adaptation]: Encapsulate the AI ​​synthesized sound byte stream into an audio blob, give it to the HTML5 player, and try to play it automatically
                const blob = await response.blob();
                const audioUrl = URL.createObjectURL(blob);
                const audioEl = document.getElementById('result-audio');
                audioEl.src = audioUrl;
                audioEl.style.display = 'block';
                // Execute automatic playback, and print warning logs if restricted by modern browser security rules.
                audioEl.play().catch(e => console.log('Audio autoplay blocked or failed', e));
            } else {
                // [Text and JSON response]: Try to beautify and format the returned string for JSON display, otherwise display the text directly
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
            // Capture the AbortError that the user actively interrupts and return directly without reporting an error. Otherwise, the user will be prompted with an error in red.
            if (err.name === 'AbortError') return;
            document.getElementById('result-text').style.display = 'block';
            document.getElementById('result-text').textContent = `Error: ${err.message}`;
            document.getElementById('result-text').style.color = '#f87171'; // Error message in red
        } finally {
            setLoading(false);
        }
    }

    // Initialize
    document.addEventListener('DOMContentLoaded', initSidebar);

    /**
     * Toggle the collapsed/expanded state of the navigation bar group
     * @param {HTMLElement} titleEl clicked nav-title element
     */
    function toggleSection(titleEl) {
        const sectionEl = titleEl.parentElement;
        sectionEl.classList.toggle('collapsed');
    }
    ```

  - `style.css`: Styles for the frontend dashboard.

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
