# Agents API 文档

在 `aispect-agents` 模块中，官方默认提供了多个预置的 AI 智能体（Agents）。您只需在接口上使用 `@AiUnitAgent` 注解，即可将普通的 Java 方法映射到这些大模型智能体上。

---

## 📝 1. 数据清理与扩写 (DataCleanAgent)

**全限定类名**: `com.aispect.agents.data.DataCleanAgent`

**功能描述**: 
专注于文本的处理、润色、清理和扩写。它会接收用户的输入文本，并使用更加丰富和专业的词汇来补充细节，将简短的文本扩写为具有深度的段落。

**参数契约**:
- **参数 1 (`String`)**: 用户提供的原始文本。

**使用示例**:
```java
@AiUnitAgent(
    name = DataCleanAgent.class, 
    description = "Expands the given text with more details and richer vocabulary."
)
String processText(String originalText);
```

---

## 🖼️ 2. 图像处理与识别 (ImageProcessAgent)

**全限定类名**: `com.aispect.agents.data.ImageProcessAgent`

**功能描述**: 
一个多模态智能体，专门用于接收和理解图像数据。您可以附带特定的指令，让大模型（如 Gemini 1.5 Flash 或 Imagen 等）针对上传的图片进行识别、分析或修改建议。

**参数契约**:
- **参数 1 (`byte[]`)**: 图片的二进制数据（将被框架自动转换为多模态的 `inlineData` 发送给大模型）。
- **参数 2 (`String`)**: 用户针对该图片发出的具体指令（如“请将图片中的尼克换成朱迪”）。

**使用示例**:
```java
@AiUnitAgent(
    name = ImageProcessAgent.class, 
    description = "Processes an image based on instructions.", 
    modelName = "imagen-4-generate"
)
byte[] processImage(byte[] image, String instruction); // 根据业务也可配置返回 String
```

---

## 📖 3. 看图编故事 (ImageStoryAgent)

**全限定类名**: `com.aispect.agents.data.ImageStoryAgent`

**功能描述**: 
同样是一个多模态智能体，但其内部系统提示词被定制为“极具创造力的故事讲述者”。它会观察图片中的细节、人物和场景，并结合用户可能给出的提示，生成富有想象力的故事。

**参数契约**:
- **参数 1 (`byte[]`)**: 图片的二进制数据。
- **参数 2 (`String`)**: 额外的情节要求或提示词（如果为空，默认要求“看图讲故事”）。

**使用示例**:
```java
@AiUnitAgent(
    name = ImageStoryAgent.class, 
    description = "Tells an imaginative story based on the image.", 
    modelName = "gemini-2.5-flash"
)
String storyFromImage(byte[] image, String instruction);
```

---

## 🗂️ 4. JSON 提取器 (JsonExtractorAgent)

**全限定类名**: `com.aispect.agents.data.JsonExtractorAgent`

**功能描述**: 
用于从非结构化的杂乱文本中提取出 JSON 字符串。它会拦截原有方法的执行，将传入的文本交给 AI 解析，并自动过滤掉多余的 Markdown 标记（如 ```json），仅返回纯净的 JSON 字符串。

**参数契约**:
- **参数 1 (`String` 或任意 `Object`)**: 包含杂乱数据的原始文本。

**使用示例**:
```java
@AiUnitAgent(
    name = JsonExtractorAgent.class, 
    description = "提取其中的 JSON 数据"
)
String extractJson(String rawText);
```
