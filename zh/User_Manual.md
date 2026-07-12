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
  ai:
    endpoint: "https://api.openai.com/v1"
    api-key: "sk-xxxxxx"
    model-name: "gpt-4o"
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

## 3. 进阶玩法
通过实现更复杂的智能体，您可以在 `orchestrate` 中实现外部工具调用（MCP），或是对多个带有 `@AiUnitAgent` 的组件进行 DAG 图结构编排（即将上线）。
