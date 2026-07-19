# Quarkus 实战案例

AiSpect 通过 CDI（上下文和依赖注入）与 Quarkus 无缝集成。您只需使用 `@Inject` 即可在应用程序中启用 AI 能力。

本案例（位于 `examples/example-quarkus` 目录）展示了如何在 Quarkus 应用程序中集成 AiSpect，并暴露了用于代码审查、阶段测试和数据类型测试的各种 REST API。

---

## 1. 案例架构

以下是本案例中包含的所有代码文件及其用途的完整列表：

- **配置类**
  - `AiSpectConfig.java`: 生产 `AiClient` 和 `AiOperations` Bean 的 CDI 配置类。

- **控制器 (Resources)**
  - `CodeReviewResource.java`: 处理代码审查 Webhook 的 REST 端点。
  - `PhaseTestResource.java`: 用于测试不同拦截阶段的 REST 端点。
  - `DataTypeTestResource.java`: 用于测试不同数据类型的 REST 端点。

- **服务 (Services)**
  - `CodeReviewService.java`: 代码审查的业务逻辑及 `@AiUnitAgent` 定义。
  - `PhaseTestService.java`: 阶段测试的业务逻辑及 `@AiUnitAgent` 定义。
  - `DataTypeTestService.java`: 数据类型处理的业务逻辑及 `@AiUnitAgent` 定义。

- **资源文件**
  - `application.properties`: Quarkus 应用程序配置文件。

---

## 2. 引擎配置

在 Quarkus 中，您通过 CDI 配置类提供 `AiClient` 和 `AiOperations` Bean。

```java
import com.aispect.api.AiClient;
import com.aispect.api.AiOperations;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Produces;

@ApplicationScoped
public class AiSpectConfig {

    // 生产 AiClient Bean
    @Produces
    @ApplicationScoped
    public AiClient aiClient() {
        return new OpenAiAdapter();
    }
    
    // 生产 AiOperations Bean
    @Produces
    @ApplicationScoped
    public AiOperations aiOperations(AiClient aiClient) {
        // 为拦截器将 AiClient 包装成 AiOperations
        return new AiOperationsWrapper(aiClient);
    }
}
```

## 3. 使用注解

只需使用 `@AiUnitAgent` 标注您的 CDI Bean 方法。Quarkus 拦截器会自动处理 AI 逻辑。

```java
import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;
import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class CodeReviewService {

    // 分析代码差异并返回 JSON 格式的总结
    @AiUnitAgent(value = DataCleanAgent.class, description = "分析代码差异并返回缺陷和建议的 JSON 总结。")
    public String reviewDiff(String diffContent) {
        // AI 处理失败时的降级逻辑
        return "{ \"status\": \"fallback\" }";
    }
}
```

## 4. 在资源中注入

然后，您可以使用标准的 JAX-RS 将此服务注入到您的 REST 端点中。

```java
import jakarta.inject.Inject;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;

@Path("/webhook/code-review")
public class CodeReviewResource {

    @Inject
    CodeReviewService codeReviewService;

    @POST
    public String handleWebhook(String diff) {
        // 调用会自动被 AiSpect 拦截
        return codeReviewService.reviewDiff(diff);
    }
}
```

---

## 5. 运行体验

### 步骤一：配置 API 密钥
将您的 API 密钥设置为环境变量：
```bash
export GEMINI_API_KEY="您的_API_KEY_在这里"
```

### 步骤二：以开发模式启动 Quarkus
在项目根目录运行 Gradle 命令：
```bash
./gradlew :example-quarkus:quarkusDev
```
