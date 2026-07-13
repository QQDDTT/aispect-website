# Spring Boot 实战案例

AiSpect 提供了无缝集成 Spring Boot 的能力，让您无需编写复杂的 HTTP 请求和 Prompt 拼接代码，只需定义 Java 接口即可完成与大模型的交互。

本案例（位于 `examples/example-spring-boot` 目录）展示了如何快速构建一个包含“阶段拦截（Phase）测试”和“数据类型（Data Type）测试”的 API Dashboard，用于全方位体验 AiSpect 的特性。

---

## 1. 案例架构

- **控制器 (`TestController`)**: 提供 REST/Web 端点，包括各个 Phase 和 Data Type 测试接口。
- **契约接口 (`PhaseTestService` & `DataTypeTestService`)**: 业务的核心。无需编写复杂的调用逻辑，通过 `@AiUnitAgent` 注解即可代理给各个 Agent。
- **前端页面 (`index.html`)**: 提供了一个现代化的暗黑风格测试控制台，左侧导航栏可选定接口，右侧可配置请求并实时预览响应内容（支持 JSON 格式化和流式输出）。

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
