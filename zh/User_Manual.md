# AiSpect 使用手册 (User Manual)

欢迎使用 AiSpect！通过本手册，您可以快速了解如何在外部项目中接入并使用 AiSpect 的智能能力。

## 1. 概念初探

AiSpect 是一个“基于注解”的 Agent 编排框架。
您无需自己管理复杂的大模型通信逻辑，只需要在目标函数上添加一个 `@AiAgent` 注解，即可在这个函数执行前或执行后，利用 AI 修改数据、做判断或丰富结果。

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
定义您的业务方法，并贴上 `@AiAgent` 注解。
```java
@Service
public class OrderService {

    // 让 AI 在真正生成订单前，对入参进行合法性或意图检查
    @AiAgent(name = "OrderValidatorAgent", agentClass = OrderValidatorAgentImpl.class)
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
public class OrderValidatorAgentImpl implements Agent {
    
    @Override
    public void preProcess(InvocationContext ctx) {
        // 这里的逻辑会在这条链路执行前被调用，底层通过 AiClient 与大模型通信
        // 比如：判断 userRequirement 中是否包含高危词汇，如果包含则抛出异常短路流程
    }

    @Override
    public void postProcess(Object result, InvocationContext ctx) {
        // 这里的逻辑会在这条链路执行后被调用
    }
}
```

## 3. 进阶玩法
通过实现更复杂的 `Agent` 接口，您可以在 `preProcess` 中实现外部工具调用（MCP），或是对多个带有 `@AiAgent` 的组件进行 DAG 图结构编排（即将上线）。
