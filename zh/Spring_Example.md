# Spring Boot 实战案例

AiSpect 提供了无缝集成 Spring Boot 的能力，让您无需编写复杂的 HTTP 请求和 Prompt 拼接代码，只需定义 Java 接口即可完成与大模型的交互。

本案例（位于 `examples/example-spring-boot` 目录）展示了如何快速构建一个拥有“文本润色”、“图片处理”和“看图编故事”能力的 Web 应用程序。

---

## 1. 案例架构

- **控制器 (`DocumentController`)**: 提供 REST/Web 端点，处理前端发来的文本和文件上传请求。
- **契约接口 (`AiEditorService`)**: 业务的核心。无需实现类，所有方法均打上 `@AiUnitAgent` 注解，由 AiSpect 引擎在运行时动态代理并调用相应的 Agent。
- **前端页面 (`index.html`)**: 提供简洁的左右分栏交互界面，左侧编辑/上传，右侧预览 AI 结果。

---

在使用了 AiSpect 的 Spring Boot 应用中，您的 `Service` 接口通过标注特定注解就能具备 AI 能力。并且您可以为其编写实现类来提供兜底（Fallback）或者本地化处理逻辑：

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import com.aispect.agents.data.ImageProcessAgent;
import com.aispect.agents.data.ImageStoryAgent;

public interface AiEditorService {

    // 文本处理：代理到 DataCleanAgent
    @AiUnitAgent(name = DataCleanAgent.class, description = "Expands the given text with more details and richer vocabulary.")
    String processText(String originalText);
    
    // 图像分析：代理到 ImageProcessAgent
    @AiUnitAgent(name = ImageProcessAgent.class, description = "Processes an image based on instructions.", modelName="imagen-4-generate")
    byte[] processImage(byte[] image, String instruction);

    // 图像故事：代理到 ImageStoryAgent
    @AiUnitAgent(name = ImageStoryAgent.class, description = "Tells an imaginative story based on the image.", modelName="gemini-2.5-flash")
    String storyFromImage(byte[] image, String instruction);
}
```

框架的 AOP 拦截器（如 `AgentInterceptor`）会自动拦截带有 `@AiUnitAgent` 的方法执行，并在调用时自动提取参数（如将 `byte[]` 转化为大模型的多模态输入）、执行大模型通信，若发生异常或被显式拒绝拦截，则会优雅降级执行您的本地 `AiEditorServiceImpl`。

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
