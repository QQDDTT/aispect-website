# AiSpect 下载中心 (Downloads)

AiSpect 的核心代码目前闭源，但我们对外提供预编译的 JAR 包和 Maven/Gradle 仓库地址。

## 最新版本下载 (v1.0.0-Beta)

请根据您的项目架构，下载对应的适配 JAR 包和依赖。

### 核心包（必选）
提供最底层的注解定义和核心 AI 通信拦截能力：
- 📦 `aispect-agents-1.0.0.jar` —— [点击下载](#) (占位链接。零依赖声明包)
- 📦 `aispect-core-1.0.0.jar` —— [点击下载](#) (占位链接。包含底层网络、大模型通信与代理引擎)

### 引擎适配包（选择其一即可）
根据您的运行时环境下载对应的适配包：
- 🍃 **Spring Boot 专属**: `aispect-engine-spring-1.0.0.jar` —— [点击下载](#) (占位链接)
- ☕ **纯 Java SE 专属**: `aispect-engine-se-1.0.0.jar` —— [点击下载](#) (占位链接)

---

## 3. 外部依赖说明

AiSpect `agent-api` 保持绝对零外部依赖。但在运行期间，由引擎拉取的 `aispect-core` 内部封装了网络客户端工具（如 OkHttp 3.x 等）用于大模型通信。在使用 Spring Boot 时，`aispect-engine-spring` Starter 会自动管理这些级联依赖版本，您无需手动干预。
