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
    description = "扩展给定文本，提供更多细节和丰富的词汇。"
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
    description = "根据指令处理或分析图像。", 
    modelName = "gemini-2.5-flash"
)
String processImage(byte[] image, String instruction);
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
    description = "根据图像讲述富有想象力的故事。", 
    modelName = "gemini-2.5-flash"
)
String storyFromImage(byte[] image, String instruction);
```
