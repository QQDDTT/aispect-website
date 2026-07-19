# Spring Boot 实战案例

AiSpect 提供了无缝集成 Spring Boot 的能力，让您无需编写复杂的 HTTP 请求和 Prompt 拼接代码，只需定义 Java 接口即可完成与大模型的交互。

本案例（位于 `examples/example-spring-boot` 目录）展示了如何快速构建一个包含“阶段拦截（Phase）测试”和“数据类型（Data Type）测试”的 API Dashboard，用于全方位体验 AiSpect 的特性。

---

## 1. 案例架构

以下是本案例中包含的所有代码文件及其用途的完整列表：

- **应用入口**
  - `Application.java`: Spring Boot 启动类。

    ```java
    package com.aispect.example.spring;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    import com.aispect.engine.spring.annotation.EnableAiSpect;

    /**
     * Spring Boot 应用启动类
     * 包含 @EnableAiSpect 注解以启用 AiSpect 引擎
     */
    @SpringBootApplication
    @EnableAiSpect
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```

- **控制器 (Controllers)**
  - `TestController.java`: 提供 REST/Web 端点，包括所有阶段 (Phase)、数据类型 (Data Type) 和图 (Graph) 的测试接口。

    ```java
    package com.aispect.example.spring.controller;

    import java.util.List;
    import java.util.Map;

    import org.springframework.http.MediaType;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.multipart.MultipartFile;

    import com.aispect.example.spring.dto.User;
    import com.aispect.example.spring.service.DataTypeTestService;
    import com.aispect.example.spring.service.PhaseTestService;

    /**
     * 测试控制器，提供各种 AiSpect 功能的测试接口
     */
    @RestController
    @RequestMapping("/test")
    public class TestController {

        private final PhaseTestService phaseTestService;
        private final DataTypeTestService dataTypeTestService;

        public TestController(PhaseTestService phaseTestService, DataTypeTestService dataTypeTestService) {
            this.phaseTestService = phaseTestService;
            this.dataTypeTestService = dataTypeTestService;
        }

        // -- Phase Tests --

        /**
         * 测试前置阶段 (Before Phase)
         * 
         * @param input 输入字符串
         * @return 经过前置处理的字符串
         */
        @GetMapping("/phase/before")
        public String phaseBefore(@RequestParam(defaultValue = "safe data") String input) {
            return phaseTestService.testBeforePhase(input);
        }

        /**
         * 测试后置阶段 (After Phase)
         * 
         * @param input 输入字符串
         * @return 经过后置摘要处理的字符串
         */
        @GetMapping("/phase/after")
        public String phaseAfter(@RequestParam(defaultValue = "dummy") String input) {
            return phaseTestService.testAfterPhase(input);
        }

        /**
         * 测试异常阶段 (Exception Phase)
         * 
         * @param input 输入字符串
         * @return 异常降级或正常处理的结果
         */
        @GetMapping("/phase/exception")
        public String phaseException(@RequestParam(defaultValue = "error") String input) {
            return phaseTestService.testExceptionPhase(input);
        }

        /**
         * 测试全生命周期阶段 (All Phase)
         * 
         * @param text 输入文本
         * @return 全生命周期处理结果
         */
        @PostMapping("/phase/all")
        public String phaseAll(@RequestBody String text) {
            return phaseTestService.testAllPhase(text);
        }

        // -- Data Type Tests --

        /**
         * 测试字符串数据类型处理
         * 
         * @param text 输入文本
         * @return 经过 AI 清理的字符串
         */
        @PostMapping("/datatype/string")
        public String typeString(@RequestBody String text) {
            return dataTypeTestService.processString(text);
        }

        /**
         * 测试 POJO 数据类型处理
         * 
         * @param user 用户对象
         * @return 经过 AI 修改状态后的用户对象
         */
        @PostMapping(value = "/datatype/pojo", consumes = MediaType.APPLICATION_JSON_VALUE)
        public User typePojo(@RequestBody User user) {
            return dataTypeTestService.processPojo(user);
        }

        /**
         * 测试集合数据类型处理
         * 
         * @param items 字符串集合
         * @return 经过 AI 分类后的集合映射
         */
        @PostMapping(value = "/datatype/collection")
        public Map<String, List<String>> typeCollection(@RequestBody List<String> items) {
            return dataTypeTestService.processCollection(items);
        }

        /**
         * 测试图像数据类型处理
         * 
         * @param file        图像文件
         * @param instruction 处理指令
         * @return 经过 AI 处理的图像字节数组
         * @throws Exception 异常
         */
        @PostMapping("/datatype/image")
        public byte[] typeImage(@RequestParam("image") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processImage(file.getBytes(), instruction);
        }

        /**
         * 测试音频数据类型处理
         * 
         * @param file        音频文件
         * @param instruction 分析指令
         * @return 经过 AI 分析出的文本说明
         * @throws Exception 异常
         */
        @PostMapping("/datatype/audio")
        public String typeAudio(@RequestParam("audio") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processAudio(file.getBytes(), file.getContentType(), instruction);
        }

        /**
         * 测试视频数据类型处理
         * 
         * @param file        视频文件
         * @param instruction 分析指令
         * @return 经过 AI 分析出的文本说明
         * @throws Exception 异常
         */
        @PostMapping("/datatype/video")
        public String typeVideo(@RequestParam("video") MultipartFile file, @RequestParam("instruction") String instruction)
                throws Exception {
            return dataTypeTestService.processVideo(file.getBytes(), file.getContentType(), instruction);
        }

        /**
         * 测试图片内容深度分析
         * 
         * @param file        图片文件
         * @param instruction 描述指令
         * @return 图片分析文字说明
         * @throws Exception 异常
         */
        @PostMapping("/datatype/image-analyze")
        public String analyzeImage(@RequestParam("image") MultipartFile file,
                @RequestParam("instruction") String instruction) throws Exception {
            return dataTypeTestService.analyzeImage(file.getBytes(), instruction);
        }

        /**
         * 测试文字转声音 (TTS)
         * 
         * @param text 输入的字符串文本
         * @return 合成后的声音音频二进制流
         */
        @PostMapping(value = "/datatype/tts", produces = "audio/wav")
        public byte[] typeTts(@RequestBody String text) {
            return dataTypeTestService.textToSpeech(text);
        }
    }
    ```

- **契约接口 (Services)**
  - `PhaseTestService.java`: 用于测试不同拦截器阶段（Before, After, Exception, All）的接口。

    ```java
    package com.aispect.example.spring.service;

    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agents.security.ContentModerationAgent;
    import com.aispect.agents.flow.FallbackAgent;
    import com.aispect.agents.data.JsonExtractorAgent;
    import com.aispect.example.spring.agents.SummaryAfterAgent;

    /**
     * 阶段测试服务接口。
     * <p>
     * 配合 {@link AiUnitAgent} 注解，演示并测试 AiSpect 框架在不同执行切面阶段 (BEFORE, AFTER, EXCEPTION, ALL)
     * 的拦截代理、上下文传递以及大模型处理接入能力。
     * </p>
     */
    public interface PhaseTestService {

        /**
         * 测试前置切面（BEFORE 阶段）的处理拦截。
         * <p>
         * 绑定 {@link ContentModerationAgent}，在本地原生方法执行之前触发 AI 过滤。
         * 大模型将对输入文本进行前置合规性与安全审查。若审核不通过可提前拦截抛出异常。
         * </p>
         *
         * @param input 待审核的输入文本内容
         * @return 经过前置审核后，原生业务逻辑处理产生的文本结果
         */
        @AiUnitAgent(value = ContentModerationAgent.class, executePhase = AiExecutePhase.BEFORE, description = "执行前进行内容合规审查")
        String testBeforePhase(String input);

        /**
         * 测试后置切面（AFTER 阶段）的处理拦截。
         * <p>
         * 绑定 {@link SummaryAfterAgent}。原生方法（桩实现）将先被执行以获取结果，
         * 随后自动将返回的结果作为入参传入 AI，由大模型进行后续的智能总结、提炼与重写。
         * </p>
         *
         * @param input 测试输入的原始文本字符串
         * @return 最终由大模型总结提炼后的文本结果
         */
        @AiUnitAgent(value = SummaryAfterAgent.class, executePhase = AiExecutePhase.AFTER, description = "执行后对生成文本进行总结提取")
        String testAfterPhase(String input);

        /**
         * 测试异常降级切面（EXCEPTION 阶段）的捕获与回退。
         * <p>
         * 绑定 {@link FallbackAgent}。当原生业务方法在运行中发生并抛出未捕获的异常时，
         * 拦截器将自动拦截异常，并将上下文及异常信息交给大模型进行智能容灾与退回降级响应。
         * </p>
         *
         * @param input 触发异常或测试回退的测试输入文本
         * @return 发生异常后，由大模型容灾机制所返回的降级结果文本
         */
        @AiUnitAgent(value = FallbackAgent.class, executePhase = AiExecutePhase.EXCEPTION, description = "异常时触发降级回退机制")
        String testExceptionPhase(String input);

        /**
         * 测试全权接管切面（ALL 阶段）的处理。
         * <p>
         * 绑定 {@link JsonExtractorAgent}。大模型直接代替和接管了原生方法的执行逻辑，
         * 原生本地方法将直接被跳过，直接通过 AI 从输入文本中分析、提取并组装出格式化的 JSON 字符串结果。
         * </p>
         *
         * @param input 含有潜在 JSON 信息或提取要求的输入文本
         * @return 大模型直接接管并解析提取出的格式化 JSON 字符串
         */
        @AiUnitAgent(value = JsonExtractorAgent.class, executePhase = AiExecutePhase.ALL, description = "在所有生命周期环节提取并解析JSON数据")
        String testAllPhase(String input);
    }
    ```

  - `DataTypeTestService.java`: 用于测试不同数据类型（String, POJO, Collection, Audio, Video）的接口。

    ```java
    package com.aispect.example.spring.service;

    import java.util.List;
    import java.util.Map;

    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agents.data.DataCleanAgent;
    import com.aispect.agents.data.ImageProcessAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.example.spring.agents.AudioProcessAgent;
    import com.aispect.example.spring.agents.CollectionProcessAgent;
    import com.aispect.example.spring.agents.PojoProcessAgent;
    import com.aispect.example.spring.agents.TextToSpeechAgent;
    import com.aispect.example.spring.agents.VideoProcessAgent;
    import com.aispect.example.spring.agents.VideoStoryAgent;
    import com.aispect.example.spring.dto.User;

    /**
     * 数据类型测试服务接口
     * <p>
     * 演示 AiSpect 框架处理和支持的各种不同数据类型（包括 Pojo、List 集合、多媒体图像二进制、
     * 音频二进制、视频二进制以及语音合成等），展示多模态（Multimodality）处理能力和 Agent 编排机制。
     * </p>
     */
    public interface DataTypeTestService {

        /**
         * 清理并格式化输入的文本字符串数据。
         * <p>
         * 绑定 {@link DataCleanAgent}，通过全阶段（ALL 阶段）接管，将原始文本传给大模型做智能清洗。
         * </p>
         *
         * @param text 待清洗处理的原始文本字符串
         * @return 清洗、纠错或格式化后的字符串结果
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "清理字符串数据", executePhase = AiExecutePhase.ALL)
        String processString(String text);

        /**
         * 处理用户对象数据并更新其状态属性。
         * <p>
         * 绑定 {@link PojoProcessAgent}，通过全阶段（ALL 阶段）接管。
         * 方法内部包含 AOP `proceed` 机制：先运行本地 `DataTypeTestServiceImpl` 中的名字加前缀和状态初始化业务逻辑，
         * 随后自动将处理后的 `User` 转为 JSON 并交由 AI 进行年龄递增与状态标记升级。
         * </p>
         *
         * @param user 用户传输对象实体，包含姓名、年龄、初始状态等
         * @return 经过本地处理与大模型智能标注后的完整 User 对象
         */
        @AiUnitAgent(value = PojoProcessAgent.class, description = "处理用户对象数据并更新状态", executePhase = AiExecutePhase.ALL)
        User processPojo(User user);

        /**
         * 对集合数据（字符串列表）进行分类整理，并以 Key-Value Map 的映射结构返回。
         * <p>
         * 绑定 {@link CollectionProcessAgent}，通过全阶段（ALL 阶段）接管。
         * 演示大模型提取非结构化字符串集合并按照主题或属性进行结构化分类整理的能力。
         * </p>
         *
         * @param items 待分类的字符串列表集合
         * @return 按照类别分组后的分类结果映射 Map
         */
        @AiUnitAgent(value = CollectionProcessAgent.class, description = "对集合数据进行分类处理", executePhase = AiExecutePhase.ALL)
        Map<String, List<String>> processCollection(List<String> items);

        /**
         * 根据指定的编辑指令，对输入的图片二进制数据进行多模态智能修改。
         * <p>
         * 绑定 {@link ImageProcessAgent}，通过全阶段（ALL 阶段）接管。
         * 大模型支持对输入的图片字节数组进行处理加工，并直接返回生成后的图片字节数组。
         * </p>
         *
         * @param image 原始图片的二进制字节数组
         * @param instruction 指导大模型如何处理或修改图片的文本指令
         * @return 修改或处理后的图片二进制字节数组
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "根据指令处理图片", executePhase = AiExecutePhase.ALL)
        byte[] processImage(byte[] image, String instruction);

        /**
         * 多模态分析音频内容，并根据指令回答。
         * <p>
         * 绑定 {@link AudioProcessAgent}，指定使用模型为 {@code gemini-3.5-flash}。
         * 演示大模型直接读取音频二进制流并理解其语音、旋律或环境背景的能力。
         * </p>
         *
         * @param audio 音频的二进制字节数组（例如 PCM / WAV 格式数据）
         * @param mimeType 音频的 MIME 媒体类型（如 {@code audio/wav}、{@code audio/mp3} 等）
         * @param instruction 要求大模型分析音频的提示词指令
         * @return 大模型分析提取出的文本回答或转录结果
         */
        @AiUnitAgent(value = AudioProcessAgent.class, description = "分析音频内容", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String processAudio(byte[] audio, String mimeType, String instruction);

        /**
         * 多模态分析视频内容，并根据指令回答。
         * <p>
         * 绑定 {@link VideoProcessAgent}，指定使用模型为 {@code gemini-3.5-flash}。
         * 演示大模型直接对视频文件的帧画面和声音流进行跨模态关联与理解的能力。
         * </p>
         *
         * @param video 视频文件的二进制字节数组（如 MP4 格式数据）
         * @param mimeType 视频的 MIME 媒体类型（如 {@code video/mp4}）
         * @param instruction 要求大模型分析视频的提示词指令
         * @return 大模型分析提取出的文本回答或总结
         */
        @AiUnitAgent(value = VideoProcessAgent.class, description = "分析视频内容", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String processVideo(byte[] video, String mimeType, String instruction);

        /**
         * 多模态自动解读视频，生成结构化场景描述与叙事摘要。
         * <p>
         * 绑定 {@link VideoStoryAgent}，指定使用模型为 {@code gemini-3.5-flash}。
         * 无需传入指令，大模型自动按时序描述视频的整体主题、关键场景与音效/对白，
         * 演示多模态视频理解与内容创作的综合能力。
         * </p>
         *
         * @param video    视频文件的二进制字节数组（如 MP4 格式数据）
         * @param mimeType 视频的 MIME 媒体类型（如 {@code video/mp4}）
         * @return 大模型输出的视频场景描述与叙事摘要文本
         */
        @AiUnitAgent(value = VideoStoryAgent.class, description = "自动生成视频场景描述与叙事摘要", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String describeVideo(byte[] video, String mimeType);

        /**
         * 多模态分析图片内容并直接以文本格式返回分析说明。
         * <p>
         * 绑定 {@link ImageProcessAgent}，指定使用模型为 {@code gemini-3.5-flash}。
         * 演示大模型提取图片内的场景、文字 (OCR)、实体并生成描述的能力。
         * </p>
         *
         * @param image 原始图片的二进制字节数组
         * @param instruction 提取和描述的提示词指令
         * @return 对图片的文本描述或分析结果
         */
        @AiUnitAgent(value = ImageProcessAgent.class, description = "分析图片内容并返回文字说明", modelName = "gemini-3.5-flash", executePhase = AiExecutePhase.ALL)
        String analyzeImage(byte[] image, String instruction);

        /**
         * 将输入的文字通过大模型转换为高逼真的声音音频。
         * <p>
         * 绑定 {@link TextToSpeechAgent}，指定使用原生语音合成大模型 {@code gemini-3.1-flash-tts-preview}。
         * 演示通过大模型多模态 TTS 接口将文字生成音频二进制流并直接输出的能力。
         * </p>
         *
         * @param text 待转为声音的输入字符串文本
         * @return 生成的声音音频文件的二进制字节数组（如 MP3 格式音频字节流）
         */
        @AiUnitAgent(value = TextToSpeechAgent.class, description = "将文字转换为声音音频", modelName = "gemini-3.1-flash-tts-preview", executePhase = AiExecutePhase.ALL)
        byte[] textToSpeech(String text);

        /**
         * 绕过框架代理注解，支持反射或兜底直接调用的原始 User 处理接口。
         *
         * @param user 用户传输对象实体
         * @return 处理后的结果对象
         */
        Object processPojo(Object user);
    }
    ```

  - `GraphTestService.java`: 用于测试智能体图 (Agent Graph) 编排的接口。

    ```java
    package com.aispect.example.spring.service;

    import java.util.Map;

    import com.aispect.agent.annotation.AiGraphAgent;
    import com.aispect.agent.annotation.AiNode;
    import com.aispect.agents.graph.DynamicWorkflowGraphAgent;

    /**
     * 测试 Graph Agent 的工作流编排服务接口。
     * <p>
     * 该接口配合 {@link AiGraphAgent} 绑定了 {@link DynamicWorkflowGraphAgent} 拓扑图智能体，
     * 用于管理和调度一组前后存在数据依赖的多步骤（由多个 {@link AiNode} 标识的步骤）的 AI 协同生成工作流。
     * 流程包含：生成故事大纲 -> 基于大纲写故事 -> 总结写好的故事。
     * </p>
     */
    @AiGraphAgent(name = "workflowGraphAgent", value = DynamicWorkflowGraphAgent.class, description = "A generic workflow agent that orchestrates a series of tasks")
    public interface GraphTestService {

        /**
         * 根据主题背景生成故事的大纲框架（第一步）。
         * <p>
         * 对应工作流中的 `generateOutline` 节点，依赖全局上下文（globalContext）中的输入参数 {@code "initialTopic"}。
         * 生成的故事大纲将被保存回全局上下文中作为下一步的输入。
         * </p>
         *
         * @param globalContext 流程全局共享的上下文映射 Map，其中至少应包含键 {@code "initialTopic"}
         * @return 生成的故事大纲文本内容
         */
        @AiNode(name = "generateOutline", description = "Given the global context which contains 'initialTopic', generate a story outline and return it.")
        String generateOutline(Map<String, Object> globalContext);

        /**
         * 根据生成的故事大纲框架创作一个简短完整的故事（第二步）。
         * <p>
         * 对应工作流中的 `writeStory` 节点，其输入源为前一步生成的 {@code "generateOutline_result"}。
         * 产出的故事将存入全局上下文中。
         * </p>
         *
         * @param globalContext 流程全局共享的上下文映射 Map，其中至少应包含键 {@code "generateOutline_result"}
         * @return 创作出的简短故事文本内容
         */
        @AiNode(name = "writeStory", description = "Given the global context which contains 'generateOutline_result', write a short story based on the outline and return it.")
        String writeStory(Map<String, Object> globalContext);

        /**
         * 智能总结上一阶段生成的故事（第三步）。
         * <p>
         * 对应工作流中的 `summarizeStory` 节点，其输入源为第二步生成的 {@code "writeStory_result"}。
         * 故事的最终总结会被返回，作为整个工作流的最终产出或存入全局上下文。
         * </p>
         *
         * @param globalContext 流程全局共享的上下文映射 Map，其中至少应包含键 {@code "writeStory_result"}
         * @return 一句话的故事精华总结文本
         */
        @AiNode(name = "summarizeStory", description = "Given the global context which contains 'writeStory_result', summarize the story in one sentence and return it.")
        String summarizeStory(Map<String, Object> globalContext);
    }
    ```

  - `AiSpectErrorTestService.java`: 用于测试错误处理和降级机制的接口。

    ```java
    package com.aispect.example.spring.service;

    import com.aispect.agent.annotation.AiNode;
    import com.aispect.agent.annotation.AiUnitAgent;
    import com.aispect.agent.enumeration.AiExecutePhase;
    import com.aispect.agents.data.DataCleanAgent;

    /**
     * 用于测试 AiSpect 组件扫描期异常信息的服务接口。
     * <p>
     * 该接口特意包含了一系列能触发 {@link com.aispect.common.exception.AiScanException} 的错误 Agent 和 Node 的配置用法，
     * 旨在验证和测试框架在 Bootstrap（启动扫描校验）阶段对注解冲突与不合法配置的健壮检查与友好错误提示能力。
     * </p>
     */
    public interface AiSpectErrorTestService {

        /**
         * 场景 1：测试同一个方法上重复绑定相同执行阶段（默认的 ALL 阶段）的 Unit Agent 冲突。
         * <p>
         * 当同一个方法上标注了多个 {@link AiUnitAgent} 时，它们必须指定不同的执行阶段（executePhase）。
         * 若未指定（隐式默认为 ALL 阶段）或显式设为相同的阶段，将会触发同阶段冲突校验错误。
         * </p>
         *
         * @param input 测试输入的原始文本字符串
         * @return 经过智能体处理后的响应文本，在正常扫描启动时若报错则无法执行到此
         */
        @AiUnitAgent(value = DataCleanAgent.class, description = "Agent 1: 默认阶段")
        @AiUnitAgent(value = DataCleanAgent.class, description = "Agent 2: 同样是默认阶段")
        String testSamePhaseConflict(String input);

        /**
         * 场景 2：测试将全阶段接管（ALL 阶段）与局部特定阶段（如 BEFORE 阶段）在同一方法上混用的冲突。
         * <p>
         * {@link AiExecutePhase#ALL} 表示完全接管目标方法的执行逻辑，这意味着其他任何阶段（如 BEFORE 或 AFTER）
         * 无法与之共存。如果同时标注了 ALL 阶段和 BEFORE/AFTER 阶段的 Agent，框架将无法调度，并触发扫描期冲突异常。
         * </p>
         *
         * @param input 测试输入的原始文本字符串
         * @return 处理后的响应结果
         */
        @AiUnitAgent(value = DataCleanAgent.class, executePhase = AiExecutePhase.ALL, description = "全阶段接管")
        @AiUnitAgent(value = DataCleanAgent.class, executePhase = AiExecutePhase.BEFORE, description = "前置阶段处理")
        String testAllPhaseMixedConflict(String input);

        /**
         * 场景 3：测试同一个方法上重复绑定相同执行阶段（默认阶段）的 {@link AiNode} 冲突。
         * <p>
         * 当该方法属于一个 Graph 节点且绑定了多个 {@link AiNode} 注解时，若这些节点使用了相同的执行阶段，
         * 会在框架启动扫描时触发 AiNode 的同阶段冲突校验错误。
         * </p>
         *
         * @param input 测试输入的原始文本字符串
         * @return 经过工作流节点处理后的结果
         */
        @AiNode(name = "node1", description = "Node 1: 默认阶段")
        @AiNode(name = "node2", description = "Node 2: 同样是默认阶段")
        String testNodeSamePhaseConflict(String input);

        /**
         * 场景 4：测试将 {@link AiNode} 的 ALL 全阶段接管与特定阶段（BEFORE 阶段）混用的冲突。
         * <p>
         * 与 Unit Agent 类似，AiNode 的 ALL 阶段也不能与其他局部执行阶段混合配置于同一个方法上，
         * 否则将破坏 Graph 工作流的单节点调度路由，触发组件扫描期冲突错误。
         * </p>
         *
         * @param input 测试输入的原始文本字符串
         * @return 经过工作流节点处理后的结果
         */
        @AiNode(name = "nodeAll", executePhase = AiExecutePhase.ALL, description = "全阶段接管")
        @AiNode(name = "nodeBefore", executePhase = AiExecutePhase.BEFORE, description = "前置阶段处理")
        String testNodeAllPhaseMixedConflict(String input);

    }
    ```

- **服务实现类**
  - `PhaseTestServiceImpl.java`, `DataTypeTestServiceImpl.java`, `GraphTestServiceImpl.java`, `AiSpectErrorTestServiceImpl.java`: 对应服务的原生实现（降级/桩逻辑）。

    ```java
    package com.aispect.example.spring.service;

    import org.springframework.stereotype.Service;

    /**
     * 阶段测试服务实现类
     * 实现了不同生命周期阶段的测试方法逻辑
     */
    @Service
    public class PhaseTestServiceImpl implements PhaseTestService {

        @Override
        public String testBeforePhase(String input) {
            return "Original method executed successfully with input: " + input;
        }

        @Override
        public String testAfterPhase(String input) {
            // AI will intercept this returned string and summarize it
            return "The quick brown fox jumps over the lazy dog. This is a very long and detailed sentence that needs to be summarized by the AI agent in the AFTER phase.";
        }

        @Override
        public String testExceptionPhase(String input) {
            // Simulate a runtime exception
            if ("error".equalsIgnoreCase(input)) {
                throw new RuntimeException("Simulated internal server error in testExceptionPhase");
            }
            return "No error occurred.";
        }

        @Override
        public String testAllPhase(String input) {
            // This should be short-circuited by JsonExtractorAgent and never run.
            return "This text should not be returned, because ALL phase agent handles it.";
        }
    }
    ```

    ```java
    package com.aispect.example.spring.service;

    import java.util.Collections;
    import java.util.List;
    import java.util.Map;

    import org.springframework.stereotype.Service;

    import com.aispect.example.spring.dto.User;

    /**
     * 数据类型测试服务实现类
     * 提供对不同数据类型（字符串、POJO、集合、图像、流等）的方法默认实现
     */
    @Service
    public class DataTypeTestServiceImpl implements DataTypeTestService {

        @Override
        public String processString(String text) {
            return text;
        }

        @Override
        public User processPojo(User user) {
            if (user == null) {
                user = new User();
            }
            if (user.getName() != null && !user.getName().startsWith("[Local]")) {
                user.setName("[Local] " + user.getName());
            }
            if (user.getStatus() == null || user.getStatus().isEmpty()) {
                user.setStatus("PENDING");
            }
            return user;
        }

        @Override
        public Map<String, List<String>> processCollection(List<String> items) {
            return Collections.emptyMap();
        }

        @Override
        public byte[] processImage(byte[] image, String instruction) {
            return image;
        }

        @Override
        public String processAudio(byte[] audio, String mimeType, String instruction) {
            return "";
        }

        @Override
        public String processVideo(byte[] video, String mimeType, String instruction) {
            return "";
        }

        @Override
        public String describeVideo(byte[] video, String mimeType) {
            return "";
        }

        @Override
        public String analyzeImage(byte[] image, String instruction) {
            return "";
        }

        @Override
        public byte[] textToSpeech(String text) {
            return new byte[0];
        }

        @Override
        public Object processPojo(Object user) {
            throw new UnsupportedOperationException("Unimplemented method 'processPojo'");
        }
    }
    ```

    ```java
    package com.aispect.example.spring.service;

    import java.util.Map;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Service;

    /**
     * 测试 Graph Agent 的服务实现
     * 包含了能够被 DynamicWorkflowGraphAgent 调度的多个 @AiNode
     */
    @Service
    public class GraphTestServiceImpl implements GraphTestService {

        private static final Logger logger = LoggerFactory.getLogger(GraphTestServiceImpl.class);

        @Override
        public String generateOutline(Map<String, Object> globalContext) {
            Object topic = globalContext.get("initialTopic");
            if (topic == null) {
                topic = "Default Topic";
            }
            logger.info("Generating outline for topic: {}", topic);
            return "Outline for: " + topic + " - 1. Introduction, 2. Climax, 3. Resolution";
        }

        @Override
        public String writeStory(Map<String, Object> globalContext) {
            Object outline = globalContext.get("generateOutline_result");
            logger.info("Writing story based on outline: {}", outline);
            return "Once upon a time, there was a great adventure. " + outline + ". The End.";
        }

        @Override
        public String summarizeStory(Map<String, Object> globalContext) {
            Object story = globalContext.get("writeStory_result");
            logger.info("Summarizing story: {}", story);
            return "Summary: A thrilling adventure happened.";
        }
    }
    ```

    ```java
    package com.aispect.example.spring.service;

    import org.springframework.stereotype.Service;

    @Service
    public class AiSpectErrorTestServiceImpl implements AiSpectErrorTestService {

        @Override
        public String testSamePhaseConflict(String input) {
            return "testSamePhaseConflict";
        }

        @Override
        public String testAllPhaseMixedConflict(String input) {
            return "testAllPhaseMixedConflict";
        }

        @Override
        public String testNodeSamePhaseConflict(String input) {
            return "testNodeSamePhaseConflict";
        }

        @Override
        public String testNodeAllPhaseMixedConflict(String input) {
            return "testNodeAllPhaseMixedConflict";
        }

    }
    ```

- **智能体 (Agents)**
  - `SummaryAfterAgent.java`: 用于在方法执行后对文本进行总结的智能体。

    ```java
    package com.aispect.example.spring.agents;

    import java.util.Collections;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    /**
     * 后置摘要智能体
     * 继承 AbstractUnitAgent，实现在目标方法执行完毕后，调用大模型对结果进行总结
     */
    public class SummaryAfterAgent extends AbstractUnitAgent<Object, String> {
        @Override
        public String postExecute(AiInvocationContext<Object> ctx, String result) throws AiSpectException {
            String input = result != null ? result : "";

            String systemPrompt = "You are a summarizing assistant. Summarize the user's text into a concise, professional sentence.";
            Message userMsg = new Message(Role.USER, input);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

  - `CollectionProcessAgent.java`: 用于对集合中的元素进行分类的智能体。

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;
    import com.fasterxml.jackson.core.type.TypeReference;
    import com.fasterxml.jackson.databind.ObjectMapper;

    import java.util.Collections;
    import java.util.List;
    import java.util.Map;

    import com.fasterxml.jackson.core.JsonProcessingException;

    /**
     * 集合处理智能体
     * 通过大模型对传入的列表项进行智能分类，并返回 JSON 映射结果
     */
    public class CollectionProcessAgent extends AbstractUnitAgent<Object, Map<String, List<String>>> {
        private static final ObjectMapper mapper = new ObjectMapper();

        @Override
        public Map<String, List<String>> execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            String input = (args != null && args.length > 0) ? args[0].toString() : "[]";
            
            String systemPrompt = "You are a categorization assistant. Group the following list of items into categories. Output ONLY valid JSON mapping category name (String) to a list of items (List<String>), no markdown.";
            Message userMsg = new Message(Role.USER, input);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            String result = response.message().content().replace("
    ```

  - `PojoProcessAgent.java`: 用于处理复杂 POJO 结构的智能体。

    ```java
    package com.aispect.example.spring.agents;

    import java.util.Collections;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;
    import com.aispect.example.spring.dto.User;
    import com.fasterxml.jackson.core.JsonProcessingException;
    import com.fasterxml.jackson.databind.ObjectMapper;

    /**
     * POJO 处理智能体
     * 调用大模型对用户对象 (User) 的属性进行修改并返回更新后的对象
     */
    public class PojoProcessAgent extends AbstractUnitAgent<Object, User> {
        private static final ObjectMapper mapper = new ObjectMapper();

        @Override
        public User execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            // 1. 显式调用本地原方法业务逻辑 (AOP proceed 机制)
            User localUser;
            try {
                localUser = (User) ctx.proceed();
            } catch (Throwable e) {
                throw new AiSpectException("Local invocation failed", e);
            }

            if (localUser == null) {
                localUser = new User();
            }
            
            String inputJson;
            try {
                inputJson = mapper.writeValueAsString(localUser);
            } catch(JsonProcessingException e) {
                inputJson = "{}";
            }

            String systemPrompt = "You are an AI post-processor. Read the User JSON (which has been locally processed). Update the 'status' to 'VERIFIED' and add 10 to 'age'. Return ONLY valid JSON, no markdown.";
            Message userMsg = new Message(Role.USER, inputJson);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            String aiResult = response.message().content().replace("
    ```

  - `AudioProcessAgent.java`, `VideoProcessAgent.java`, `TextToSpeechAgent.java`, `VideoStoryAgent.java`: 用于处理音频、视频等多模态任务的智能体。

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * 音频处理 Agent，拦截方法并把音频 byte[] 及其 mimeType 封装为 inlineData 发送给大模型。
     */
    public class AudioProcessAgent extends AbstractUnitAgent<Object, String> {

        @Override
        public String execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            byte[] audioBytes = (args != null && args.length > 0 && args[0] instanceof byte[]) ? (byte[]) args[0] : new byte[0];
            String mimeType = (args != null && args.length > 1 && args[1] instanceof String) ? (String) args[1] : "audio/mp3";
            String instruction = (args != null && args.length > 2 && args[2] instanceof String) ? (String) args[2] : "Describe this audio";

            String systemPrompt = "You are an audio analysis assistant. Please process the provided audio according to the user instruction.";
            Message userMsg = new Message(Role.USER, instruction, null, null, audioBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * 视频处理 Agent，拦截方法并把视频 byte[] 及其 mimeType 封装为 inlineData 发送给大模型。
     */
    public class VideoProcessAgent extends AbstractUnitAgent<Object, String> {

        @Override
        public String execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            byte[] videoBytes = (args != null && args.length > 0 && args[0] instanceof byte[]) ? (byte[]) args[0] : new byte[0];
            String mimeType = (args != null && args.length > 1 && args[1] instanceof String) ? (String) args[1] : "video/mp4";
            String instruction = (args != null && args.length > 2 && args[2] instanceof String) ? (String) args[2] : "Describe this video";

            String systemPrompt = "You are a video analysis assistant. Please process the provided video according to the user instruction.";
            Message userMsg = new Message(Role.USER, instruction, null, null, videoBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * 语音合成 Agent，将文字提示词转换为音频。
     */
    public class TextToSpeechAgent extends AbstractUnitAgent<Object, byte[]> {

        @Override
        public byte[] execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            String text = (args != null && args.length > 0 && args[0] != null) ? args[0].toString() : "";

            String systemPrompt = "You are a text-to-speech assistant. Please convert the user's text into speech audio data.";
            Message userMsg = new Message(Role.USER, text);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            
            // 如果 response.message() 返回了 inlineData，直接作为音频字节返回
            if (response.message().inlineData() != null) {
                return response.message().inlineData();
            }
            
            // 兜底：返回文本内容对应的字节数组
            return response.message().content() != null ? response.message().content().getBytes() : new byte[0];
        }
    }
    ```

    ```java
    package com.aispect.example.spring.agents;

    import com.aispect.agent.core.AbstractUnitAgent;
    import com.aispect.api.AiInvocationContext;
    import com.aispect.common.exception.AiSpectException;
    import com.aispect.core.client.model.Message;
    import com.aispect.core.client.model.PromptContext;
    import com.aispect.core.client.model.Response;
    import com.aispect.core.client.model.Role;

    import java.util.Collections;

    /**
     * 看视频讲故事 Agent。
     * <p>
     * 该 Agent 接收一段视频二进制数据，利用多模态大模型自动分析视频画面与声音，
     * 并生成详细的场景描述与叙事摘要，无需用户提供额外指令。
     * </p>
     */
    public class VideoStoryAgent extends AbstractUnitAgent<Object, String> {

        @Override
        public String execute(AiInvocationContext<Object> ctx) throws AiSpectException {
            Object[] args = ctx.getArgs();
            byte[] videoBytes = (args != null && args.length > 0 && args[0] instanceof byte[]) ? (byte[]) args[0] : new byte[0];
            String mimeType = (args != null && args.length > 1 && args[1] instanceof String) ? (String) args[1] : "video/mp4";

            String systemPrompt = "You are a professional video narrator. "
                    + "Watch the provided video carefully and generate a vivid, structured description: "
                    + "1) Summarize the overall theme or story of the video in one sentence. "
                    + "2) Describe the key scenes or moments chronologically. "
                    + "3) Note any significant audio, dialogue, or sound effects. "
                    + "Answer in the same language as the user's message (default: Chinese).";

            Message userMsg = new Message(Role.USER, "请详细描述这段视频的内容与场景。", null, null, videoBytes, mimeType);
            PromptContext promptCtx = new PromptContext(systemPrompt, Collections.singletonList(userMsg));

            Response response = getAiOperations(ctx).prompt(ctx.getAgentName(), promptCtx);
            return response.message().content();
        }
    }
    ```

- **配置与 DTO**
  - `AiGraphConfig.java`: 用于定义 Agent Graph 工作流的配置类。

    ```java
    package com.aispect.example.spring.config;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.function.RouterFunction;
    import org.springframework.web.servlet.function.RouterFunctions;
    import org.springframework.web.servlet.function.ServerResponse;

    import com.aispect.api.AiOperations;
    import com.aispect.engine.spring.launcher.AiGraphAgentSpringLauncher;

    /**
     * 示例应用中用于手动配置 Graph Agent 相关 Bean 的配置类。
     */
    @Configuration
    public class AiGraphConfig {

        /**
         * 注册 AiGraphAgentSpringLauncher。
         * 负责在 Spring 启动时收集所有的 @AiGraphAgent 并管理生命周期。
         *
         * @param aiOperations 核心的 AI 客户端操作接口，由底层自动装配提供
         * @return 启动器实例
         */
        @Bean
        public AiGraphAgentSpringLauncher aiGraphAgentSpringLauncher(AiOperations aiOperations) {
            return new AiGraphAgentSpringLauncher(aiOperations);
        }

        /**
         * 注册 Graph Agent 的 HTTP 路由。
         * 此处交由业务层自由定制路由路径和请求方法，并将执行任务分发给 Launcher。
         *
         * @param launcher Graph 启动器
         * @return 动态注册的端点路由
         */
        @Bean
        public RouterFunction<ServerResponse> graphAgentRouterFunction(AiGraphAgentSpringLauncher launcher) {
            return RouterFunctions.route()
                    .POST("/ai/graph/{agentName}", request -> {
                        String agentName = request.pathVariable("agentName");
                        try {
                            String body = request.body(String.class);
                            Object result = launcher.executeAgent(agentName, body);
                            return ServerResponse.ok().body(result != null ? result : "Graph execution completed with null result.");
                        } catch (IllegalArgumentException e) {
                            return ServerResponse.notFound().build();
                        } catch (Exception e) {
                            return ServerResponse.status(500).body("Error executing graph agent: " + e.getMessage());
                        }
                    })
                    .build();
        }
    }
    ```

  - `User.java`: 用于演示 POJO 绑定的 DTO。

    ```java
    package com.aispect.example.spring.dto;

    /**
     * 用户数据传输对象 (DTO)
     * 用于测试实体类在 AiSpect 中的传递和处理
     */
    public class User {
        private String name;
        private int age;
        private String status;

        public User() {}

        public User(String name, int age, String status) {
            this.name = name;
            this.age = age;
            this.status = status;
        }

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
        public String getStatus() { return status; }
        public void setStatus(String status) { this.status = status; }
    }
    ```

- **资源与前端页面**
  - `application.properties`: Spring Boot 应用配置文件。

    ```properties
    logging.level.com.aispect=DEBUG

    # AiSpect AI Framework Configuration
    aispect.provider=gemini
    aispect.default-model=gemini-2.5-flash
    # You can set the API key directly here, or inject it via environment variables (e.g., from .env)
    aispect.api-key=${GEMINI_API_KEY:}
    ```

  - `index.html`: 提供一个现代化的暗黑风格交互式 API 测试控制台。

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>AiSpect API Dashboard</title>
        <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
        <link href="/css/style.css" rel="stylesheet">
    </head>
    <body>

        <aside class="sidebar">
            <div class="logo-container">
                <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L2 7l10 5 10-5-10-5z"></path><path d="M2 17l10 5 10-5"></path><path d="M2 12l10 5 10-5"></path></svg>
                AiSpect
            </div>

            <div class="nav-menu-container">
                <div class="nav-section">
                    <div class="nav-title" onclick="toggleSection(this)">
                        <span>Phase Tests</span>
                        <svg class="chevron-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
                    </div>
                    <div id="nav-phase" class="nav-items-wrapper"></div>
                </div>

                <div class="nav-section">
                    <div class="nav-title" onclick="toggleSection(this)">
                        <span>Data Type Tests</span>
                        <svg class="chevron-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
                    </div>
                    <div id="nav-datatype" class="nav-items-wrapper"></div>
                </div>

                <div class="nav-section">
                    <div class="nav-title" onclick="toggleSection(this)">
                        <span>Graph Tests</span>
                        <svg class="chevron-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 12 15 18 9"></polyline></svg>
                    </div>
                    <div id="nav-graph" class="nav-items-wrapper"></div>
                </div>
            </div>
        </aside>

        <main class="main-wrapper">
            <header class="header">
                <h1 id="page-title">Select a Test</h1>
                <p id="page-desc">Choose an API endpoint from the sidebar to begin testing.</p>
            </header>

            <div class="content-grid" id="main-content" style="display: none;">
                <!-- Request Card -->
                <div class="card fade-in">
                    <h3>
                        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--primary)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
                        Configure Request
                    </h3>
                    <div id="endpoint-info" class="endpoint-badge"></div>
                    <form id="test-form" onsubmit="event.preventDefault(); runTest();">
                        <div id="form-fields"></div>
                        <button type="submit" class="btn btn-primary" id="run-btn">
                            <span>Send Request</span>
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>
                        </button>
                    </form>
                </div>

                <!-- Response Card -->
                <div class="card fade-in" style="animation-delay: 0.1s;">
                    <h3 style="display: flex; justify-content: space-between;">
                        <span style="display: flex; align-items: center; gap: 8px;">
                            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="var(--secondary)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="21 8 21 21 3 21 3 8"></polyline><rect x="1" y="3" width="22" height="5"></rect><line x1="10" y1="12" x2="14" y2="12"></line></svg>
                            Response
                        </span>
                        <span id="response-status" style="font-size: 13px; font-weight: normal; color: var(--text-muted);"></span>
                    </h3>
                    <div class="result-container" id="result-box">
                        <div class="empty-state" id="empty-state">
                            <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" style="opacity: 0.5; margin-bottom: 16px;"><circle cx="12" cy="12" r="10"></circle><polyline points="12 16 16 12 12 8"></polyline><line x1="8" y1="12" x2="16" y2="12"></line></svg>
                            Waiting for execution...
                        </div>
                        <div class="empty-state" id="loading-state" style="display: none;">
                            <div class="spinner"></div>
                            <p style="margin-top: 16px; color: white;">Processing request...</p>
                        </div>
                        <pre class="result-output" id="result-text" style="display: none;"></pre>
                        <img class="result-image" id="result-image" />
                        <audio class="result-audio" id="result-audio" controls style="display: none; width: 100%; margin-top: 16px; outline: none;"></audio>
                    </div>
                </div>
            </div>
        </main>
        <script src="/js/app.js"></script>
    </body>
    </html>
    ```

  - `app.js`: 前端逻辑，用于配置请求和预览结果（支持 JSON 格式化和 SSE 流式输出）。

    ```javascript
    /**
     * AiSpect 演示控制面板核心 JavaScript 脚本
     * 负责渲染左侧测试导航栏、动态构建请求配置表单、发起 AOP 交互请求以及解析并渲染各类多模态 AI 响应。
     */

    // 所有 API 测试场景的配置注册表
    const testConfigs = [
        // --- Phase Tests (切面生命周期阶段测试) ---
        { 
            id: 'phase-before', 
            group: 'phase', 
            title: 'Phase: Before', 
            type: 'GET', 
            url: '/test/phase/before', 
            inputs: [{name: 'input', label: 'Input Text', type: 'text', default: 'safe data'}] 
        },
        { 
            id: 'phase-after', 
            group: 'phase', 
            title: 'Phase: After', 
            type: 'GET', 
            url: '/test/phase/after', 
            inputs: [{name: 'input', label: 'Input Text', type: 'text', default: 'dummy'}] 
        },
        { 
            id: 'phase-exception', 
            group: 'phase', 
            title: 'Phase: Exception', 
            type: 'GET', 
            url: '/test/phase/exception', 
            inputs: [{name: 'input', label: 'Input Text', type: 'text', default: 'error'}] 
        },
        { 
            id: 'phase-all', 
            group: 'phase', 
            title: 'Phase: All', 
            type: 'POST', 
            url: '/test/phase/all', 
            isBody: true, 
            inputs: [{name: 'body', label: 'Raw Text', type: 'textarea', default: 'Hello AiSpect'}] 
        },
        
        // --- Data Type Tests (多模态与不同数据类型测试) ---
        { 
            id: 'type-string', 
            group: 'datatype', 
            title: 'Type: String', 
            type: 'POST', 
            url: '/test/datatype/string', 
            isBody: true, 
            inputs: [{name: 'body', label: 'String Body', type: 'textarea', default: 'Testing raw string injection'}] 
        },
        { 
            id: 'type-pojo', 
            group: 'datatype', 
            title: 'Type: POJO', 
            type: 'POST', 
            url: '/test/datatype/pojo', 
            isJson: true, 
            inputs: [{name: 'body', label: 'JSON User Object', type: 'textarea', default: '{\n  "name": "Alice",\n  "age": 25,\n  "status": "NEW"\n}'}] 
        },
        { 
            id: 'type-collection', 
            group: 'datatype', 
            title: 'Type: Collection', 
            type: 'POST', 
            url: '/test/datatype/collection', 
            isJson: true, 
            inputs: [{name: 'body', label: 'JSON Array', type: 'textarea', default: '[\n  "apple",\n  "banana",\n  "carrot"\n]'}] 
        },
        { 
            id: 'type-image', 
            group: 'datatype', 
            title: 'Type: Image', 
            type: 'POST', 
            url: '/test/datatype/image', 
            isFormData: true, 
            inputs: [
                {name: 'image', label: 'Upload Image', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Describe the contents of this image'}
            ], 
            responseType: 'blob' 
        },
        { 
            id: 'type-image-analyze', 
            group: 'datatype', 
            title: 'Type: Image Analyze (Text)', 
            type: 'POST', 
            url: '/test/datatype/image-analyze', 
            isFormData: true, 
            inputs: [
                {name: 'image', label: 'Upload Image', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Perform a detailed scene analysis of this image.'}
            ] 
        },
        { 
            id: 'type-audio', 
            group: 'datatype', 
            title: 'Type: Audio Analysis', 
            type: 'POST', 
            url: '/test/datatype/audio', 
            isFormData: true, 
            inputs: [
                {name: 'audio', label: 'Upload Audio', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Transcribe this audio and explain its background environment.'}
            ] 
        },
        { 
            id: 'type-video', 
            group: 'datatype', 
            title: 'Type: Video Analysis', 
            type: 'POST', 
            url: '/test/datatype/video', 
            isFormData: true, 
            inputs: [
                {name: 'video', label: 'Upload Video', type: 'file'},
                {name: 'instruction', label: 'Instruction', type: 'text', default: 'Summarize the visual and audio actions in this video.'}
            ] 
        },
        { 
            id: 'type-tts', 
            group: 'datatype', 
            title: 'Type: Text to Speech (TTS)', 
            type: 'POST', 
            url: '/test/datatype/tts', 
            isBody: true, 
            inputs: [
                {name: 'body', label: 'Text to Speak', type: 'textarea', default: 'Welcome to AiSpect, the next generation framework for agentic development!'}
            ], 
            responseType: 'audio' 
        },
        { 
            id: 'type-stream', 
            group: 'datatype', 
            title: 'Type: Stream', 
            type: 'GET', 
            url: '/test/datatype/stream', 
            inputs: [{name: 'topic', label: 'Topic', type: 'text', default: 'Cyberpunk City'}], 
            isStream: true 
        },
        
        // --- Graph Tests (拓扑图工作流编排测试) ---
        { 
            id: 'graph-execute', 
            group: 'graph', 
            title: 'Graph: Workflow', 
            type: 'POST', 
            url: '/ai/graph/workflowGraphAgent', 
            isBody: true, 
            inputs: [{name: 'body', label: 'Topic (String)', type: 'textarea', default: 'A lone spaceman exploring a ruined planet.'}] 
        }
    ];

    // 当前选中的测试配置项
    let currentTest = null;
    // 用于取消请求的控制器，防止用户重复提交和并发冲突
    let currentAbortController = null;

    /**
     * 初始化左侧边栏导航项
     * 遍历配置注册表，动态创建按钮元素并绑定点击选择事件，分组挂载到对应的 DOM 容器下。
     */
    function initSidebar() {
        const phaseNav = document.getElementById('nav-phase');
        const datatypeNav = document.getElementById('nav-datatype');
        const graphNav = document.getElementById('nav-graph');

        testConfigs.forEach(test => {
            const el = document.createElement('div');
            el.className = 'nav-item';
            // 使用 SVG Chevron 右箭头图标
            el.innerHTML = `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="9 18 15 12 9 6"></polyline></svg> ${test.title}`;
            el.onclick = () => selectTest(test.id);
            el.id = `nav-${test.id}`;

            // 按分组挂载到相应版块
            if (test.group === 'phase') phaseNav.appendChild(el);
            else if (test.group === 'datatype') datatypeNav.appendChild(el);
            else if (test.group === 'graph') graphNav.appendChild(el);
        });
    }

    /**
     * 选中某一个测试项的事件处理器
     * 激活当前导航状态，重置右侧内容面板、表单字段和之前产生的结果区域。
     * 
     * @param {string} id 测试项的唯一 ID
     */
    function selectTest(id) {
        // 移除所有导航项的 active 样式，并将 active 赋予新选中的项
        document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
        document.getElementById(`nav-${id}`).classList.add('active');

        // 获取并记录当前测试项配置
        currentTest = testConfigs.find(t => t.id === id);
        
        // 更新页面标题和描述信息
        document.getElementById('page-title').textContent = currentTest.title;
        document.getElementById('page-desc').textContent = `Testing endpoint: ${currentTest.url}`;
        
        // 渲染 API 类型标志徽章
        const badge = document.getElementById('endpoint-info');
        badge.textContent = `${currentTest.type} ${currentTest.url}`;
        badge.className = `endpoint-badge ${currentTest.type.toLowerCase()}`;

        // 动态渲染请求参数配置表单
        renderForm();
        // 清空上次请求产生的 Response 数据
        resetResult();
        // 展示右侧的主网格容器
        document.getElementById('main-content').style.display = 'grid';
    }

    /**
     * 根据当前测试配置动态构建和渲染 Request 输入表单
     * 支持渲染文本框 (text)、长文本框 (textarea)、以及文件上传按钮 (file)。
     */
    function renderForm() {
        const container = document.getElementById('form-fields');
        container.innerHTML = ''; // 清空旧表单字段

        currentTest.inputs.forEach(input => {
            const group = document.createElement('div');
            group.className = 'form-group';
            
            const label = document.createElement('label');
            label.className = 'form-label';
            label.textContent = input.label;
            group.appendChild(label);

            // 根据 input 类型选择渲染相应的表单控件
            if (input.type === 'textarea') {
                const el = document.createElement('textarea');
                el.className = 'form-control';
                el.id = `input-${input.name}`;
                el.value = input.default || '';
                group.appendChild(el);
            } else if (input.type === 'file') {
                // 定制化的现代美观文件上传按钮
                const wrapper = document.createElement('div');
                wrapper.className = 'file-input-wrapper';
                wrapper.innerHTML = `
                    <button type="button" class="btn btn-outline" style="pointer-events: none;">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
                        <span id="file-name-${input.name}">Choose File...</span>
                    </button>
                    <input type="file" id="input-${input.name}" onchange="document.getElementById('file-name-${input.name}').textContent = this.files[0] ? this.files[0].name : 'Choose File...'" />
                `;
                group.appendChild(wrapper);
            } else {
                const el = document.createElement('input');
                el.type = input.type;
                el.className = 'form-control';
                el.id = `input-${input.name}`;
                el.value = input.default || '';
                group.appendChild(el);
            }
            
            container.appendChild(group);
        });
    }

    /**
     * 重置 Response 响应面板的状态
     * 隐藏图片和音频播放组件，彻底释放 Blob 缓存以防止内存泄露，并还原空状态占位符。
     */
    function resetResult() {
        document.getElementById('empty-state').style.display = 'flex';
        document.getElementById('loading-state').style.display = 'none';
        document.getElementById('result-text').style.display = 'none';
        document.getElementById('result-image').style.display = 'none';
        document.getElementById('result-audio').style.display = 'none';
        document.getElementById('result-audio').src = ''; // 释放音频流
        document.getElementById('result-text').textContent = '';
        document.getElementById('result-text').style.color = '';
        document.getElementById('response-status').textContent = '';
    }

    /**
     * 切换发送按钮与结果面板的 Loading 等待状态
     * 
     * @param {boolean} isLoading 是否正在发送/处理请求中
     */
    function setLoading(isLoading) {
        const btn = document.getElementById('run-btn');
        if (isLoading) {
            // 按钮禁用，加载 Loading 动画
            btn.disabled = true;
            btn.innerHTML = `<div class="spinner" style="width: 16px; height: 16px; border-width: 2px;"></div> Processing...`;
            document.getElementById('empty-state').style.display = 'none';
            document.getElementById('loading-state').style.display = 'flex';
            document.getElementById('result-text').style.display = 'none';
            document.getElementById('result-image').style.display = 'none';
            document.getElementById('result-audio').style.display = 'none';
            document.getElementById('result-audio').src = '';
            document.getElementById('response-status').textContent = 'Fetching...';
        } else {
            // 还原按钮可用状态与原本的 SVG 发送图标
            btn.disabled = false;
            btn.innerHTML = `<span>Send Request</span><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="22" y1="2" x2="11" y2="13"></line><polygon points="22 2 15 22 11 13 2 9 22 2"></polygon></svg>`;
            document.getElementById('loading-state').style.display = 'none';
        }
    }

    /**
     * 执行测试 API 请求的核心异步方法
     * 提取表单输入，自动封装为 GET Query、Multipart FormData、JSON 或是 Plain Text 格式并发送。
     * 针对大语言模型返回的数据支持：文本流式渲染 (isStream)、图像渲染 (blob)、音频媒体流自动生成与播放 (audio) 等功能。
     */
    async function runTest() {
        if (!currentTest) return;
        
        // 如果有正在执行的请求，则强行中止它（防止多次点击重复请求）
        if (currentAbortController) {
            currentAbortController.abort();
        }
        // 创建新的中断信号控制器
        currentAbortController = new AbortController();

        setLoading(true);

        let url = currentTest.url;
        let options = {
            method: currentTest.type,
            signal: currentAbortController.signal
        };

        try {
            // --- 1. 拼装构建请求参数 ---
            if (currentTest.type === 'GET') {
                // GET 请求：使用 URLSearchParams 进行拼接
                const params = new URLSearchParams();
                currentTest.inputs.forEach(inp => {
                    const val = document.getElementById(`input-${inp.name}`).value;
                    if (val) params.append(inp.name, val);
                });
                if ([...params].length > 0) url += `?${params.toString()}`;
            } else if (currentTest.isFormData) {
                // FormData 请求：处理上传的图片/音视频等多媒体文件与伴随参数
                const formData = new FormData();
                currentTest.inputs.forEach(inp => {
                    const el = document.getElementById(`input-${inp.name}`);
                    if (inp.type === 'file') {
                        if (el.files[0]) formData.append(inp.name, el.files[0]);
                    } else {
                        formData.append(inp.name, el.value);
                    }
                });
                options.body = formData;
            } else if (currentTest.isJson) {
                // JSON 对象请求：设置 Content-Type 头部并执行 JSON 校验与序列化
                options.headers = { 'Content-Type': 'application/json' };
                const rawVal = document.getElementById(`input-body`).value;
                try {
                    JSON.parse(rawVal); // 格式校验
                    options.body = rawVal;
                } catch (e) {
                    throw new Error('Invalid JSON input format');
                }
            } else if (currentTest.isBody) {
                // 纯文本 Body 请求
                options.headers = { 'Content-Type': 'text/plain' };
                options.body = document.getElementById(`input-body`).value;
            }

            // 发送异步网络请求
            const response = await fetch(url, options);
            // 展示接口返回的 HTTP 响应状态码及状态信息
            document.getElementById('response-status').textContent = `${response.status} ${response.statusText}`;

            // --- 2. 接收和解析渲染响应结果 ---
            if (currentTest.isStream) {
                // 【流式响应渲染】：隐藏 Loading，动态读取流式文本流并实时推入文本框并回滚滚动条
                document.getElementById('loading-state').style.display = 'none';
                const textEl = document.getElementById('result-text');
                textEl.style.display = 'block';
                textEl.textContent = '';
                
                const reader = response.body.getReader();
                const decoder = new TextDecoder();
                
                while (true) {
                    const { value, done } = await reader.read();
                    if (done) break;
                    // 追加渲染解密出来的 UTF-8 字符块
                    textEl.textContent += decoder.decode(value, { stream: true });
                    // 容器自动回滚到底部以展现最新输出
                    textEl.parentElement.scrollTop = textEl.parentElement.scrollHeight;
                }
            } else if (currentTest.responseType === 'blob') {
                // 【多模态图像渲染】：将响应直接转换为二进制 Blob，并生成对象 URL 指向并渲染图像组件
                const blob = await response.blob();
                const imgUrl = URL.createObjectURL(blob);
                document.getElementById('result-image').src = imgUrl;
                document.getElementById('result-image').style.display = 'block';
            } else if (currentTest.responseType === 'audio') {
                // 【多模态 TTS 语音播放适配】：将 AI 合成的声音字节流封装为音频 Blob 赋予 HTML5 播放器，并尝试自动播音
                const blob = await response.blob();
                const audioUrl = URL.createObjectURL(blob);
                const audioEl = document.getElementById('result-audio');
                audioEl.src = audioUrl;
                audioEl.style.display = 'block';
                // 执行自动播放，若被现代浏览器安全规则限制则打印警告日志兜底
                audioEl.play().catch(e => console.log('Audio autoplay blocked or failed', e));
            } else {
                // 【文本与 JSON 响应】：尝试对返回的字符串进行美化格式化 JSON 展示，否则直接显示文本
                const text = await response.text();
                const textEl = document.getElementById('result-text');
                textEl.style.display = 'block';
                
                try {
                    const jsonObj = JSON.parse(text);
                    textEl.textContent = JSON.stringify(jsonObj, null, 2);
                } catch {
                    textEl.textContent = text;
                }
            }
        } catch (err) {
            // 捕获用户主动中断的 AbortError 则直接不报错返回，否则向用户提示错误红字
            if (err.name === 'AbortError') return;
            document.getElementById('result-text').style.display = 'block';
            document.getElementById('result-text').textContent = `Error: ${err.message}`;
            document.getElementById('result-text').style.color = '#f87171'; // 错误信息置红
        } finally {
            setLoading(false);
        }
    }

    // Initialize
    document.addEventListener('DOMContentLoaded', initSidebar);

    /**
     * 切换导航栏分组的折叠/展开状态
     * @param {HTMLElement} titleEl 点击的 nav-title 元素
     */
    function toggleSection(titleEl) {
        const sectionEl = titleEl.parentElement;
        sectionEl.classList.toggle('collapsed');
    }
    ```

  - `style.css`: 前端控制台的样式表。

    ```css
    :root {
        --primary: #6366f1;
        --primary-hover: #4f46e5;
        --secondary: #ec4899;
        --bg-color: #f1f5f9;
        --surface: #ffffff;
        --surface-glass: rgba(255, 255, 255, 0.7);
        --text-main: #0f172a;
        --text-muted: #64748b;
        --sidebar-bg: rgba(15, 23, 42, 0.85);
        --border: rgba(226, 232, 240, 0.8);
        --radius-lg: 20px;
        --radius-md: 12px;
        --shadow-sm: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        --shadow-lg: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
    }

    body {
        margin: 0;
        font-family: 'Inter', sans-serif;
        background: var(--bg-color);
        background-image: 
            radial-gradient(at 0% 0%, hsla(253,16%,7%,1) 0, transparent 50%), 
            radial-gradient(at 50% 0%, hsla(225,39%,30%,1) 0, transparent 50%), 
            radial-gradient(at 100% 0%, hsla(339,49%,30%,1) 0, transparent 50%);
        color: var(--text-main);
        display: flex;
        height: 100vh;
        overflow: hidden;
    }

    /* Sidebar Styling */
    .sidebar {
        width: 280px;
        background: var(--sidebar-bg);
        backdrop-filter: blur(24px);
        -webkit-backdrop-filter: blur(24px);
        color: white;
        display: flex;
        flex-direction: column;
        border-right: 1px solid rgba(255, 255, 255, 0.1);
        z-index: 10;
    }

    .logo-container {
        padding: 32px 24px;
        font-size: 24px;
        font-weight: 700;
        letter-spacing: -0.5px;
        display: flex;
        align-items: center;
        gap: 12px;
        background: linear-gradient(135deg, #818cf8 0%, #e879f9 100%);
        -webkit-background-clip: text;
        background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .nav-menu-container {
        flex: 1;
        overflow-y: auto;
        padding-bottom: 24px;
    }

    /* Custom Scrollbar for Nav Menu */
    .nav-menu-container::-webkit-scrollbar {
        width: 6px;
    }
    .nav-menu-container::-webkit-scrollbar-track {
        background: transparent;
    }
    .nav-menu-container::-webkit-scrollbar-thumb {
        background: rgba(255, 255, 255, 0.12);
        border-radius: 3px;
        transition: background 0.2s ease;
    }
    .nav-menu-container::-webkit-scrollbar-thumb:hover {
        background: rgba(255, 255, 255, 0.25);
    }

    .nav-section {
        padding: 0 16px;
        margin-bottom: 20px;
    }

    .nav-title {
        font-size: 12px;
        text-transform: uppercase;
        letter-spacing: 1px;
        color: #94a3b8;
        margin-bottom: 12px;
        padding: 6px 12px;
        font-weight: 600;
        cursor: pointer;
        display: flex;
        justify-content: space-between;
        align-items: center;
        user-select: none;
        border-radius: 6px;
        transition: background 0.2s ease, color 0.2s ease;
    }

    .nav-title:hover {
        background: rgba(255, 255, 255, 0.05);
        color: #f8fafc;
    }

    /* Accordion Fold Logic with CSS transitions */
    .nav-items-wrapper {
        max-height: 800px;
        overflow: hidden;
        transition: max-height 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    }

    .nav-section.collapsed .nav-items-wrapper {
        max-height: 0;
    }

    /* Chevron indicator animation */
    .chevron-icon {
        color: #64748b;
        transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1), color 0.3s ease;
    }

    .nav-title:hover .chevron-icon {
        color: #cbd5e1;
    }

    .nav-section.collapsed .chevron-icon {
        transform: rotate(-90deg);
    }

    .nav-item {
        padding: 12px 16px;
        margin-bottom: 4px;
        border-radius: 8px;
        cursor: pointer;
        transition: all 0.2s ease;
        font-size: 14px;
        font-weight: 500;
        color: #cbd5e1;
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .nav-item:hover {
        background: rgba(255, 255, 255, 0.1);
        color: white;
    }

    .nav-item.active {
        background: var(--primary);
        color: white;
        box-shadow: 0 4px 12px rgba(99, 102, 241, 0.4);
    }

    /* Main Content */
    .main-wrapper {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: 32px;
        overflow-y: auto;
        position: relative;
    }

    .header {
        margin-bottom: 32px;
    }

    .header h1 {
        margin: 0;
        font-size: 28px;
        font-weight: 700;
        color: white;
        text-shadow: 0 2px 4px rgba(0,0,0,0.5);
    }

    .header p {
        margin: 8px 0 0 0;
        color: rgba(255, 255, 255, 0.7);
        font-size: 15px;
    }

    .content-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 24px;
        align-items: start;
    }

    @media (max-width: 1024px) {
        .content-grid {
            grid-template-columns: 1fr;
        }
    }

    /* Cards */
    .card {
        background: var(--surface-glass);
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);
        border: 1px solid var(--border);
        border-radius: var(--radius-lg);
        padding: 24px;
        box-shadow: var(--shadow-lg);
    }

    .card h3 {
        margin: 0 0 20px 0;
        font-size: 18px;
        color: var(--text-main);
        display: flex;
        align-items: center;
        gap: 8px;
    }

    /* Form Elements */
    .form-group {
        margin-bottom: 20px;
    }

    .form-label {
        display: block;
        margin-bottom: 8px;
        font-size: 14px;
        font-weight: 500;
        color: var(--text-muted);
    }

    .form-control {
        width: 100%;
        padding: 12px 16px;
        border: 1px solid var(--border);
        border-radius: var(--radius-md);
        background: rgba(255, 255, 255, 0.9);
        font-family: inherit;
        font-size: 15px;
        color: var(--text-main);
        transition: all 0.2s ease;
        box-sizing: border-box;
    }

    .form-control:focus {
        outline: none;
        border-color: var(--primary);
        box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2);
    }

    textarea.form-control {
        min-height: 120px;
        resize: vertical;
        line-height: 1.5;
        font-family: 'JetBrains Mono', monospace;
        font-size: 13px;
    }

    .file-input-wrapper {
        position: relative;
        overflow: hidden;
        display: inline-block;
        width: 100%;
    }

    .file-input-wrapper input[type=file] {
        font-size: 100px;
        position: absolute;
        left: 0;
        top: 0;
        opacity: 0;
        cursor: pointer;
    }

    .btn {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        gap: 8px;
        padding: 12px 24px;
        border: none;
        border-radius: var(--radius-md);
        font-size: 15px;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        font-family: inherit;
        width: 100%;
    }

    .btn-primary {
        background: linear-gradient(135deg, var(--primary) 0%, #818cf8 100%);
        color: white;
        box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
    }

    .btn-primary:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 16px rgba(99, 102, 241, 0.4);
    }

    .btn-primary:active {
        transform: translateY(0);
    }

    .btn-primary:disabled {
        opacity: 0.7;
        cursor: not-allowed;
        transform: none;
    }

    .btn-outline {
        background: transparent;
        border: 1px solid var(--border);
        color: var(--text-main);
    }

    .btn-outline:hover {
        background: rgba(0,0,0,0.02);
    }

    /* Result Area */
    .result-container {
        position: relative;
        min-height: 200px;
        background: #1e293b;
        border-radius: var(--radius-md);
        padding: 20px;
        overflow-x: auto;
        color: #f8fafc;
        box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);
    }

    .result-output {
        margin: 0;
        font-family: 'JetBrains Mono', Consolas, monospace;
        font-size: 14px;
        line-height: 1.6;
        white-space: pre-wrap;
        word-wrap: break-word;
    }

    .result-image {
        max-width: 100%;
        border-radius: 8px;
        display: none;
        box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    }

    .result-audio {
        width: 100%;
        border-radius: 8px;
        display: none;
        box-shadow: 0 4px 12px rgba(0,0,0,0.2);
    }

    .empty-state {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100%;
        color: #64748b;
        text-align: center;
        padding: 40px 0;
    }

    .spinner {
        width: 24px;
        height: 24px;
        border: 3px solid rgba(255,255,255,0.1);
        border-radius: 50%;
        border-top-color: white;
        animation: spin 1s linear infinite;
    }

    .endpoint-badge {
        display: inline-block;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 11px;
        font-weight: 700;
        font-family: monospace;
        background: #e2e8f0;
        color: #475569;
        margin-bottom: 16px;
    }

    .endpoint-badge.get { background: #dbeafe; color: #1d4ed8; }
    .endpoint-badge.post { background: #dcfce3; color: #15803d; }

    @keyframes spin {
        to { transform: rotate(360deg); }
    }

    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(10px); }
        to { opacity: 1; transform: translateY(0); }
    }

    .fade-in {
        animation: fadeIn 0.4s cubic-bezier(0.4, 0, 0.2, 1) forwards;
    }
    ```

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
