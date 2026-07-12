# Spring Boot 实战案例

AiSpect 提供了无缝集成 Spring Boot 的能力，让您无需编写复杂的 HTTP 请求和 Prompt 拼接代码，只需定义 Java 接口即可完成与大模型的交互。

本案例（位于 `examples/example-spring-boot` 目录）展示了如何快速构建一个拥有“文本润色”、“图片处理”和“看图编故事”能力的 Web 应用程序。

---

## 1. 案例架构

- **控制器 (`DocumentController`)**: 提供 REST/Web 端点，处理前端发来的文本和文件上传请求。
- **契约接口 (`AiEditorService`)**: 业务的核心。无需实现类，所有方法均打上 `@AiUnitAgent` 注解，由 AiSpect 引擎在运行时动态代理并调用相应的 Agent。
- **前端页面 (`index.html`)**: 提供简洁的左右分栏交互界面，左侧编辑/上传，右侧预览 AI 结果。

---

## 2. 核心代码解析

在使用 AiSpect 的 Spring Boot 应用中，最惊艳的部分在于您的 `Service` 层代码仅仅是一个接口：

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import com.aispect.agents.data.ImageProcessAgent;
import com.aispect.agents.data.ImageStoryAgent;

public interface AiEditorService {

    // 文本处理：代理到 DataCleanAgent
    @AiUnitAgent(name = DataCleanAgent.class, description = "扩写文本")
    String processText(String originalText);
    
    // 图像分析：代理到 ImageProcessAgent
    @AiUnitAgent(name = ImageProcessAgent.class, description = "处理图像", modelName="gemini-2.5-flash")
    String processImage(byte[] image, String instruction);

    // 图像故事：代理到 ImageStoryAgent
    @AiUnitAgent(name = ImageStoryAgent.class, description = "根据图片讲故事", modelName="gemini-2.5-flash")
    String storyFromImage(byte[] image, String instruction);
}
```

框架的 `AiAgentBeanPostProcessor` 会自动扫描被 Spring 管理的 Bean，识别这些注解，并在调用时自动执行提示词模板装配、参数提取（将 `byte[]` 转化为大模型的多模态图片 Part），以及与 Gemini 等模型的 API 通信。

---

## 3. 运行体验

### 步骤一：配置 API 密钥
通过环境变量或启动参数注入您的模型 API 密钥：
```bash
export GEMINI_API_KEY="您的_API_KEY_在这里"
```

### 步骤二：启动项目
在项目根目录运行 Gradle 命令启动 Spring Boot：
```bash
./gradlew :example-spring-boot:bootRun
```

### 步骤三：访问页面
打开浏览器访问 `http://localhost:8080`，您可以在左侧文本框内输入文字并点击 **Process** 体验文本润色，或者上传一张图片并点击 **Tell Story (看图编故事)** 体验多模态的图文生成能力。
