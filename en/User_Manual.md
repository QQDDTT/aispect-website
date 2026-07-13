# AiSpect User Manual

Welcome to AiSpect! Through this manual, you can quickly learn how to integrate and use AiSpect's intelligent capabilities in external projects.

## 1. Concept Overview

AiSpect is an "annotation-based" Agent orchestration framework.
You do not need to manage complex LLM communication logic yourself. Just add an `@AiUnitAgent` annotation to your target function, and when that function is called, it will be intercepted by the corresponding Agent, using AI to process data, make judgments, or enrich results.

## 2. Quick Integration (Spring Boot Example)

### Step 1: Include Dependency JARs
Place the downloaded `aispect-engine-spring` and its underlying dependency packages into your project's classpath.
*(Or include it via our private Maven repository)*

### Step 2: Configure the LLM API Key
Add the configuration in your `application.yml`:
```yaml
aispect:
  provider: gemini
  default-model: gemini-2.5-flash
  api-key: "${GEMINI_API_KEY:sk-xxxxxx}"
```

### Step 3: Write Business Logic with Annotations
Define your business method and attach the `@AiUnitAgent` annotation.
```java
@Service
public class OrderService {

    // Let AI perform validity or intent checks on parameters before actually generating the order
    @AiUnitAgent(name = OrderValidatorAgentImpl.class, description = "Validate order requirements")
    public Order createOrder(String userRequirement) {
        // ... Actual database save logic ...
        return new Order();
    }
}
```

### Step 4: Implement Agent Contract
Implement the corresponding `agentClass`:
```java
@Component
public class OrderValidatorAgentImpl extends AbstractUnitAgent<Object, Order> {
    
    @Override
    public Order orchestrate(AiInvocationContext<Object> ctx) throws AiSpectException {
        // This logic is called when the original method is executed. The underlying AI model can be accessed via getAiOperations(ctx)
        // For example: check if userRequirement in args contains high-risk words, throw an exception to short-circuit if so,
        // or call the LLM to generate specific order information and return it directly.
        return new Order();
    }
}
```

## 3. Advanced Usage
By implementing more complex agents, you can execute external tool calls (MCP) within `orchestrate`, or perform DAG graph orchestration (coming soon) for multiple components annotated with `@AiUnitAgent`.
