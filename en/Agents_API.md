# Agents API Documentation

In the `aispect-agents` module, several pre-built AI agents are provided by default. Simply apply the `@AiUnitAgent` annotation to an interface, and you can map regular Java methods to these large language model agents.

---

## 📝 1. Data Cleaning and Expansion (DataCleanAgent)

**Fully Qualified Class Name**: `com.aispect.agents.data.DataCleanAgent`

**Functional Description**: 
Focuses on processing, polishing, cleaning, and expanding text. It receives the user's input text and uses richer, more professional vocabulary to add details, expanding brief text into in-depth paragraphs.

**Parameter Contract**:
- **Parameter 1 (`String`)**: The original text provided by the user.

**Usage Example**:
```java
@AiUnitAgent(
    name = DataCleanAgent.class, 
    description = "Expands the given text with more details and richer vocabulary."
)
String processText(String originalText);
```

---

## 🖼️ 2. Image Processing and Recognition (ImageProcessAgent)

**Fully Qualified Class Name**: `com.aispect.agents.data.ImageProcessAgent`

**Functional Description**: 
A multi-modal agent specifically designed to receive and understand image data. You can attach specific instructions to let the LLM (like Gemini 1.5 Flash or Imagen) identify, analyze, or suggest modifications based on the uploaded image.

**Parameter Contract**:
- **Parameter 1 (`byte[]`)**: The binary data of the image (automatically converted by the framework into multi-modal `inlineData` sent to the LLM).
- **Parameter 2 (`String`)**: Specific instructions from the user regarding the image (e.g., "Please replace Nick with Judy in the picture").

**Usage Example**:
```java
@AiUnitAgent(
    name = ImageProcessAgent.class, 
    description = "Processes an image based on instructions.", 
    modelName = "imagen-4-generate"
)
byte[] processImage(byte[] image, String instruction); // Can also be configured to return String depending on the business logic
```

---

## 📖 3. Image Storytelling (ImageStoryAgent)

**Fully Qualified Class Name**: `com.aispect.agents.data.ImageStoryAgent`

**Functional Description**: 
Also a multi-modal agent, but its internal system prompt is tailored as a "highly creative storyteller". It observes details, characters, and scenes in the image, and combined with any prompts given by the user, generates an imaginative story.

**Parameter Contract**:
- **Parameter 1 (`byte[]`)**: The binary data of the image.
- **Parameter 2 (`String`)**: Additional plot requirements or prompts (if empty, defaults to requesting a story based on the image).

**Usage Example**:
```java
@AiUnitAgent(
    name = ImageStoryAgent.class, 
    description = "Tells an imaginative story based on the image.", 
    modelName = "gemini-2.5-flash"
)
String storyFromImage(byte[] image, String instruction);
```

---

## 🗂️ 4. JSON Extractor (JsonExtractorAgent)

**Fully Qualified Class Name**: `com.aispect.agents.data.JsonExtractorAgent`

**Functional Description**: 
Used to extract JSON strings from unstructured, messy text. It intercepts the original method execution, passes the incoming text to the AI for parsing, and automatically filters out redundant Markdown tags (like ```json), returning only pure JSON strings.

**Parameter Contract**:
- **Parameter 1 (`String` or any `Object`)**: The original text containing messy data.

**Usage Example**:
```java
@AiUnitAgent(
    name = JsonExtractorAgent.class, 
    description = "Extract JSON data from it"
)
String extractJson(String rawText);
```
