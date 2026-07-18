# AiSpect 使用手册 (User Manual)

欢迎使用 AiSpect！通过本手册，您可以快速了解如何在外部项目中接入并使用 AiSpect 的智能能力。

## 1. 概念初探

AiSpect 是一个“基于注解”的 Agent 编排框架。
您无需自己管理复杂的大模型通信逻辑，只需要在目标函数上添加一个 `@AiUnitAgent` 注解，即可在调用该函数时由对应的 Agent 拦截，并利用 AI 处理数据、做判断或丰富结果。

## 2. 快速接入 (以 Spring Boot 为例)

### 步骤一：引入依赖 JAR 包
将下载好的 `aispect-engine-spring` 及其底层依赖包放入您的工程 classpath 下。
*(或者通过我们私有 Maven 仓库引入)*

### 步骤二：配置大模型密钥
在您的 `application.yml` 中添加配置：
```yaml
aispect:
  provider: gemini
  default-model: gemini-2.5-flash
  api-key: "${GEMINI_API_KEY:sk-xxxxxx}"
```

### 步骤三：编写业务与注解
定义您的业务方法，并贴上 `@AiUnitAgent` 注解。
```java
@Service
public class OrderService {

    // 让 AI 在真正生成订单前，对入参进行合法性或意图检查
    @AiUnitAgent(name = OrderValidatorAgentImpl.class, description = "校验订单要求")
    public Order createOrder(String userRequirement) {
        // ... 真实的数据库保存逻辑 ...
        return new Order();
    }
}
```

### 步骤四：实现 Agent 契约
实现对应的 `agentClass`：
```java
@Component
public class OrderValidatorAgentImpl extends AbstractUnitAgent<Object, Order> {
    
    @Override
    public Order orchestrate(AiInvocationContext<Object> ctx) throws AiSpectException {
        // 这里的逻辑会在原方法执行时被调用，底层可通过 getAiOperations(ctx) 与大模型通信
        // 比如：判断 args 中的 userRequirement 中是否包含高危词汇，如果包含则抛出异常短路流程，
        // 或者调用大模型生成特定的订单信息并直接返回。
        return new Order();
    }
}
```

## 3. 进阶玩法：图式智能体与资源节点编排

对于更加复杂的业务分发，您可以超越单一的 Unit Agent，使用图式代理来接管整个类的工作流。

### 核心注解
- **`@AiGraphAgent`**: 标注在**类级别**，作为复杂工作流的全局入口。
- **`@AiNode`**: 标注在**方法级别**，作为大模型可主动调度的子功能节点（类似于 Tool/Skill）。

### 严格的排错校验机制
为了保证流程编排的严谨性，AiSpect 引擎在启动阶段（Bootstrap）引入了**全量错误收集**能力。扫描器绝不会在遇到第一个配置错误时短路退出，而是会收集完所在函数上的所有配置缺陷，并一次性打印出所有错误日志。

**开发者必须遵守以下排错约束，否则应用启动将会被框架安全拦截**：
1. **强关联约束**：任何标注了 `@AiNode` 的方法，其宿主类必须已经被 `@AiGraphAgent` 标注。框架拒绝加载没有明确入口归属的“孤儿节点”。
2. **阶段互斥约束**：在同一个方法上如果使用了多个智能体注解（无论是 `@AiUnitAgent` 还是 `@AiNode`），它们必须具备不同的执行阶段（`executePhase`）。此外，绝不允许将 `ALL` 全阶段与 `BEFORE`/`AFTER` 等特定局部阶段混用。
