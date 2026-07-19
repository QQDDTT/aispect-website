# Spring Boot 实战案例

AiSpect 提供了无缝集成 Spring Boot 的能力，让您无需编写复杂的 HTTP 请求和 Prompt 拼接代码，只需定义 Java 接口即可完成与大模型的交互。

本案例（位于 `examples/example-spring-boot` 目录）展示了如何快速构建一个包含“阶段拦截（Phase）测试”和“数据类型（Data Type）测试”的 API Dashboard，用于全方位体验 AiSpect 的特性。

---

## 1. 案例架构

以下是本案例中包含的所有代码文件及其用途的完整列表：

- **应用入口**
  - `Application.java`: Spring Boot 启动类。

- **控制器 (Controllers)**
  - `TestController.java`: 提供 REST/Web 端点，包括所有阶段 (Phase)、数据类型 (Data Type) 和图 (Graph) 的测试接口。

- **契约接口 (Services)**
  - `PhaseTestService.java`: 用于测试不同拦截器阶段（Before, After, Exception, All）的接口。
  - `DataTypeTestService.java`: 用于测试不同数据类型（String, POJO, Collection, Audio, Video）的接口。
  - `GraphTestService.java`: 用于测试智能体图 (Agent Graph) 编排的接口。
  - `AiSpectErrorTestService.java`: 用于测试错误处理和降级机制的接口。

- **服务实现类**
  - `PhaseTestServiceImpl.java`, `DataTypeTestServiceImpl.java`, `GraphTestServiceImpl.java`, `AiSpectErrorTestServiceImpl.java`: 对应服务的原生实现（降级/桩逻辑）。

- **智能体 (Agents)**
  - `SummaryAfterAgent.java`: 用于在方法执行后对文本进行总结的智能体。
  - `CollectionProcessAgent.java`: 用于对集合中的元素进行分类的智能体。
  - `PojoProcessAgent.java`: 用于处理复杂 POJO 结构的智能体。
  - `AudioProcessAgent.java`, `VideoProcessAgent.java`, `TextToSpeechAgent.java`, `VideoStoryAgent.java`: 用于处理音频、视频等多模态任务的智能体。

- **配置与 DTO**
  - `AiGraphConfig.java`: 用于定义 Agent Graph 工作流的配置类。
  - `User.java`: 用于演示 POJO 绑定的 DTO。

- **资源与前端页面**
  - `application.properties`: Spring Boot 应用配置文件。
  - `index.html`: 提供一个现代化的暗黑风格交互式 API 测试控制台。
  - `app.js`: 前端逻辑，用于配置请求和预览结果（支持 JSON 格式化和 SSE 流式输出）。
  - `style.css`: 前端控制台的样式表。

---

在使用了 AiSpect 的 Spring Boot 应用中，您的 `Service` 接口通过标注特定注解就能具备 AI 能力。例如处理 POJO 或复杂数据集合：

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.example.spring.agents.PojoProcessAgent;
import com.aispect.example.spring.dto.User;
import java.util.List;
import java.util.Map;

public interface DataTypeTestService {

    // POJO 处理：代理到 PojoProcessAgent
    @AiUnitAgent(name = PojoProcessAgent.class, description = "Updates the status of a user object")
    User processPojo(User user);
    
    // 集合处理
    @AiUnitAgent(description = "Categorizes a list of items into fruits and vegetables")
    Map<String, List<String>> processCollection(List<String> items);
    
    // ... 其他数据类型与拦截器阶段测试
}
```

框架的 AOP 拦截器会自动拦截带有 `@AiUnitAgent` 的方法执行，并在调用时自动提取参数、执行大模型通信，返回您所需的强类型对象（如 POJO、List、Map、甚至 Stream 和 byte[]）。

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
打开浏览器访问 `http://localhost:8080`。您可以体验全新的 **AiSpect API Dashboard**，在侧边栏中切换不同类型的测试用例：
- **Phase Tests**: 体验在 Before、After 和 Exception 阶段进行自定义逻辑拦截和干预。
- **Data Type Tests**: 体验 String、POJO (JSON)、Collection、Image (多模态) 以及 Stream (SSE 流式输出) 的大模型处理与自动绑定能力。
