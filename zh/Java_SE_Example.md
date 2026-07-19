# Java SE 实战案例

AiSpect 可以在没有任何依赖注入容器（如 Spring 或 Quarkus）的纯 Java SE 环境中使用。您可以手动初始化引擎并为标准的 Java 接口创建代理。

本案例（位于 `examples/example-java-se` 目录）展示了如何手动初始化框架，并涵盖了“阶段拦截（Phase）测试”和“数据类型（Data Type）测试”。

---

## 1. 案例架构

以下是本案例中包含的所有代码文件及其用途的完整列表：

- **应用入口与核心逻辑**
  - `Main.java`: 单文件中包含了完整的案例。它包括 `PhaseTestService` 和 `DataTypeTestService` 接口及其桩实现，自定义的智能体（如 `SummaryAfterAgent` 和 `CollectionClassifierAgent`），用于无 API 密钥演示的 `MockAiClient`，以及手动构建 `AiSpect` 引擎并创建代理的 `main` 编排方法。

- **资源文件**
  - `simplelogger.properties`: SLF4J simple logger 的配置文件。

---

## 2. 引擎初始化

在纯 Java SE 环境中，您可以使用 `AiSpect.builder()` 来构建引擎，然后使用 `proxy` 方法为您的接口创建代理。

```java
import com.aispect.api.AiClient;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import com.aispect.engine.se.AiSpect;

public class Main {
    public static void main(String[] args) throws Exception {
        // 1. 初始化 AI 客户端
        AiClient aiClient = new OpenAiAdapter();
        
        // 2. 构建 AiSpect 引擎
        AiSpect engine = AiSpect.builder()
                .aiClient(aiClient)
                .build();
                
        // 3. 为您的服务创建代理
        PhaseTestService rawService = new PhaseTestServiceImpl();
        PhaseTestService proxiedService = engine.proxy(rawService, PhaseTestService.class);
        
        // 4. 调用被代理的方法
        String result = proxiedService.testBeforePhase("safe data");
        System.out.println(result);
    }
}
```

## 2. 使用注解

就像在 Spring 或 Quarkus 中一样，您可以在接口方法上使用 `@AiUnitAgent` 注解来定义 AI 行为。

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.security.ContentModerationAgent;

public interface PhaseTestService {

    // 执行前进行内容合规审查
    @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "执行前进行内容合规审查")
    String testBeforePhase(String input);
}
```

---

## 3. 运行体验

### 步骤一：配置 API 密钥
将您的 API 密钥设置为环境变量：
```bash
export GEMINI_API_KEY="您的_API_KEY_在这里"
```

### 步骤二：运行应用
在项目根目录运行 Gradle 命令：
```bash
./gradlew :example-java-se:run
```
