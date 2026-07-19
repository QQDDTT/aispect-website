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


## 完整源代码

### `java/com/aispect/example/quarkus/AiSpectConfig.java`

```java
package com.aispect.example.quarkus;

import com.aispect.api.AiClient;
import com.aispect.api.AiOperations;
import com.aispect.core.client.adapter.openai.OpenAiAdapter;
import com.aispect.core.client.model.Message;
import com.aispect.core.client.model.PromptContext;
import com.aispect.core.client.model.Response;
import com.aispect.core.client.model.Role;
import com.aispect.core.client.config.AiClientConfig;
import com.aispect.common.exception.AiSpectException;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.inject.Produces;

import java.util.stream.Stream;

/**
 * AiSpect 核心 CDI 配置类。
 * <p>
 * 负责生产 {@link AiClient} 和 {@link AiOperations} 两个核心 Bean，
 * 供 {@link com.aispect.engine.quarkus.AiAgentCdiInterceptor} 通过 {@code @Inject} 使用。
 * </p>
 * <p>
 * 优先读取 {@code GEMINI_API_KEY} 环境变量。若未设置，则使用内置 Mock 客户端进行演示。
 * </p>
 */
@ApplicationScoped
public class AiSpectConfig {

    /**
     * 生产 {@link AiClient} Bean。
     * <p>
     * 若 {@code GEMINI_API_KEY} 已配置，则使用 {@link OpenAiAdapter}（兼容 Gemini OpenAI-compatible 接口）；
     * 否则返回内置 Mock 实现用于演示。
     * </p>
     *
     * @return AiClient 实例
     */
    @Produces
    @ApplicationScoped
    public AiClient aiClient() {
        String apiKey = System.getenv("GEMINI_API_KEY");
        if (apiKey != null && !apiKey.trim().isEmpty()) {
            return new OpenAiAdapter();
        }
        // 未配置 API Key 时使用 Mock 客户端，返回固定的演示响应
        return new AiClient() {
            @Override
            public Response generate(PromptContext ctx, AiClientConfig config) throws AiSpectException {
                String content = "[Mock] AiSpect Quarkus Example — set GEMINI_API_KEY for real AI responses.";
                return new Response(new Message(Role.ASSISTANT, content), new Response.Usage(0, 0, 0));
            }

            @Override
            public Stream<Response> streamGenerate(PromptContext ctx, AiClientConfig config) throws AiSpectException {
                return Stream.of(generate(ctx, config));
            }
        };
    }

    /**
     * 生产 {@link AiOperations} Bean。
     * <p>
     * 将 {@link AiClient} 包装为标准的 {@link AiOperations} 接口，
     * 供拦截器在运行时调度大模型请求。
     * </p>
     *
     * @param aiClient 由 CDI 注入的 AiClient 实例
     * @return AiOperations 实例
     */
    @Produces
    @ApplicationScoped
    public AiOperations aiOperations(AiClient aiClient) {
        return new AiOperations() {
            @Override
            public AiClient getClient() {
                return aiClient;
            }

            @Override
            public Response prompt(PromptContext ctx) throws AiSpectException {
                return aiClient.generate(ctx, null);
            }

            @Override
            public Stream<Response> promptStream(PromptContext ctx) throws AiSpectException {
                return aiClient.streamGenerate(ctx, null);
            }

            @Override
            public Response prompt(String modelName, String agentName, PromptContext ctx) throws AiSpectException {
                AiClientConfig config = (modelName != null && !modelName.trim().isEmpty())
                        ? new AiClientConfig(null, null, modelName, null, null)
                        : null;
                return aiClient.generate(ctx, config);
            }

            @Override
            public Stream<Response> promptStream(String modelName, String agentName, PromptContext ctx) throws AiSpectException {
                AiClientConfig config = (modelName != null && !modelName.trim().isEmpty())
                        ? new AiClientConfig(null, null, modelName, null, null)
                        : null;
                return aiClient.streamGenerate(ctx, config);
            }
        };
    }
}

```

### `java/com/aispect/example/quarkus/CodeReviewResource.java`

```java
package com.aispect.example.quarkus;

import jakarta.inject.Inject;
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;
import java.util.Map;

@Path("/webhook/code-review")
public class CodeReviewResource {

    @Inject
    CodeReviewService codeReviewService;

    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response handleWebhook(Map<String, Object> payload) {
        // In a real scenario, extract diff from GitHub/GitLab webhook payload
        String diff = (String) payload.getOrDefault("diff", "");
        if (diff.isEmpty()) {
            return Response.status(400).entity("{\"error\": \"No diff provided\"}").build();
        }

        try {
            String reviewResult = codeReviewService.reviewDiff(diff);
            return Response.ok(reviewResult).build();
        } catch (Exception e) {
            return Response.serverError().entity("{\"error\": \"" + e.getMessage() + "\"}").build();
        }
    }
}

```

### `java/com/aispect/example/quarkus/CodeReviewService.java`

```java
package com.aispect.example.quarkus;

import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agents.data.DataCleanAgent;

import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class CodeReviewService {

    @AiUnitAgent(value = DataCleanAgent.class, description = "Analyzes the code diff and returns a JSON summary of defects and suggestions.")
    public String reviewDiff(String diffContent) {
        // Fallback in case CDI interception is not set up correctly or no API key is provided
        return """
               {
                 "status": "fallback",
                 "message": "No review performed. Received diff length: %d"
               }""".formatted(diffContent.length());
    }
}

```

### `java/com/aispect/example/quarkus/DataTypeTestResource.java`

```java
package com.aispect.example.quarkus;

import jakarta.inject.Inject;
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;

@Path("/test/datatype")
public class DataTypeTestResource {

    @Inject
    DataTypeTestService dataTypeTestService;

    @POST
    @Path("/string")
    @Consumes(MediaType.TEXT_PLAIN)
    @Produces(MediaType.TEXT_PLAIN)
    public String typeString(String text) {
        return dataTypeTestService.processString(text);
    }
}

```

### `java/com/aispect/example/quarkus/DataTypeTestService.java`

```java
package com.aispect.example.quarkus;

import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.data.DataCleanAgent;
import com.aispect.agents.data.ImageProcessAgent;
import com.aispect.engine.quarkus.AiAgentInterceptorBinding;

import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
@AiAgentInterceptorBinding
public class DataTypeTestService {

    @AiUnitAgent(value = DataCleanAgent.class, description = "清理字符串数据", executePhase = AiExecutePhase.ALL)
    public String processString(String text) {
        return text;
    }

    @AiUnitAgent(value = ImageProcessAgent.class, description = "根据指令处理图片", executePhase = AiExecutePhase.ALL)
    public byte[] processImage(byte[] image, String instruction) {
        return image;
    }
}

```

### `java/com/aispect/example/quarkus/PhaseTestResource.java`

```java
package com.aispect.example.quarkus;

import jakarta.inject.Inject;
import jakarta.ws.rs.Consumes;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.QueryParam;
import jakarta.ws.rs.core.MediaType;

@Path("/test/phase")
public class PhaseTestResource {

    @Inject
    PhaseTestService phaseTestService;

    @GET
    @Path("/before")
    @Produces(MediaType.TEXT_PLAIN)
    public String phaseBefore(@QueryParam("input") String input) {
        if (input == null || input.isEmpty()) {
            input = "safe data";
        }
        return phaseTestService.testBeforePhase(input);
    }

    @GET
    @Path("/exception")
    @Produces(MediaType.TEXT_PLAIN)
    public String phaseException(@QueryParam("input") String input) {
        if (input == null || input.isEmpty()) {
            input = "error";
        }
        return phaseTestService.testExceptionPhase(input);
    }

    @POST
    @Path("/all")
    @Consumes(MediaType.TEXT_PLAIN)
    @Produces(MediaType.TEXT_PLAIN)
    public String phaseAll(String text) {
        return phaseTestService.testAllPhase(text);
    }
}

```

### `java/com/aispect/example/quarkus/PhaseTestService.java`

```java
package com.aispect.example.quarkus;

import com.aispect.agent.annotation.AiUnitAgent;
import com.aispect.agent.enumeration.AiExecutePhase;
import com.aispect.agents.data.JsonExtractorAgent;
import com.aispect.agents.flow.FallbackAgent;
import com.aispect.agents.security.ContentModerationAgent;
import com.aispect.engine.quarkus.AiAgentInterceptorBinding;

import jakarta.enterprise.context.ApplicationScoped;

@ApplicationScoped
@AiAgentInterceptorBinding
public class PhaseTestService {

    @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "执行前进行内容合规审查")
    public String testBeforePhase(String input) {
        return "Original method executed successfully with input: " + input;
    }

    // In Quarkus, we reuse the basic flow agents. Note that we don't have SummaryAfterAgent in core agents,
    // so we'll just demonstrate the BEFORE, EXCEPTION, and ALL phases which use core agents.
    
    @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "异常时触发降级回退机制")
    public String testExceptionPhase(String input) {
        if ("error".equalsIgnoreCase(input)) {
            throw new RuntimeException("Simulated internal server error in testExceptionPhase");
        }
        return "No error occurred.";
    }

    @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "在所有生命周期环节提取并解析JSON数据")
    public String testAllPhase(String input) {
        return "This text should not be returned, because ALL phase agent handles it.";
    }
}

```

### `resources/application.properties`

```properties
quarkus.http.port=8081
quarkus.log.category."com.aispect".level=DEBUG

```

