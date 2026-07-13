# AiSpect Download Center

AiSpect's core code is currently closed-source, but we provide pre-compiled JAR packages and Maven/Gradle repository addresses to the public.

## Latest Release Download (v1.0.0-Beta)

Please download the corresponding adapter JARs and dependencies according to your project architecture.

### Core Packages (Required)
Provides the lowest-level annotation definitions and core AI communication interception capabilities:
- 📦 `aispect-agents-1.0.0.jar` —— [Download](../downloads/aispect-agents-1.0.0.jar) (Zero-dependency declaration package)
- 📦 `aispect-core-1.0.0.jar` —— [Download](../downloads/aispect-core-1.0.0.jar) (Contains underlying networking, LLM communication, and proxy engine)

### Engine Adapters (Choose one)
Download the corresponding adapter package according to your runtime environment:
- 🍃 **Spring Boot Exclusive**: `aispect-engine-spring-1.0.0.jar` —— [Download](../downloads/aispect-engine-spring-1.0.0.jar)
- ☕ **Pure Java SE Exclusive**: `aispect-engine-se-1.0.0.jar` —— [Download](../downloads/aispect-engine-se-1.0.0.jar)

---

## Maven and Gradle Configuration

You can import AiSpect into your project directly using the following configurations.

### 🍎 Maven (`pom.xml`)

If you are using Spring Boot, just import the Engine and Agents packages:

```xml
<dependencies>
    <!-- AiSpect Spring Boot Engine Adapter -->
    <dependency>
        <groupId>com.aispect</groupId>
        <artifactId>aispect-engine-spring</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
    
    <!-- Official Agents Library -->
    <dependency>
        <groupId>com.aispect</groupId>
        <artifactId>aispect-agents</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

### 🐘 Gradle (`build.gradle`)

The corresponding Gradle dependency import method:

```groovy
dependencies {
    // AiSpect Spring Boot Engine Adapter
    implementation 'com.aispect:aispect-engine-spring:1.0.0-SNAPSHOT'
    
    // Official Agents Library
    implementation 'com.aispect:aispect-agents:1.0.0-SNAPSHOT'
}
```

---

## External Dependencies Note

The AiSpect core annotation layer maintains absolutely zero external dependencies. However, during runtime, the `aispect-core` pulled by the engine encapsulates network client tools (such as OkHttp 3.x) for communication with large language models. When using Spring Boot, `aispect-engine-spring` automatically manages these cascading dependency versions, so no manual intervention is required.
