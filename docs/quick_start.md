# 快速入门

欢迎使用 DeepSeek-Ollama Bridge！这是一款专为 DeepSeek 模型量身打造的高性能代理服务，它为您提供以下核心特性：

-   智能多层缓存系统：结合内存与磁盘缓存，实现最优性能
-   高效流量控制：基于令牌桶算法，确保稳定可靠
-   智能思考标签过滤：专门优化 DeepSeek 蒸馏模型的输出质量
-   企业级共享模式：提供 API Key 的集中化托管与管理

#### 广泛的模型兼容性

DeepSeek-Ollama Bridge 采用标准的 OpenAI API 协议，支持与各类主流大语言模型平台无缝对接：

**本地部署型平台**

-   Ollama：轻量级的本地模型部署方案
-   LM Studio：一站式的模型管理与推理平台

**云服务型平台**

-   Gemini：Google 新一代 AI 模型服务
-   Claude：Anthropic 开发的高性能助手
-   GPT-4：OpenAI 最新一代大语言模型

此外，任何支持 OpenAI API 协议的模型服务都可以通过本代理进行访问和管理。这种广泛的兼容性让您能够灵活选择最适合的 AI 模型，同时享受统一的接口体验和性能优化。

## 安装指南

### 1. 下载软件包

使用以下命令下载最新版本：

```bash
curl -L https://github.com/shengyanli1982/deepseek-ollama-bridge-release/releases/download/v0.1.15/deepseek-ollama-bridge-Linux-x64-v0.1.15.zip -o deepseek-ollama-bridge.zip
```

### 2. 解压文件

执行解压命令：

```bash
unzip deepseek-ollama-bridge.zip
```

### 3. 启动服务

运行以下命令启动服务：

```bash
./deepseek-ollama-bridge-linux-x64
```

### 3.1 查看可用参数

你可以通过以下命令查看所有可用的配置参数：

```bash
./deepseek-ollama-bridge-linux-x64 -h

Usage: deepseek-ollama-bridge.exe [OPTIONS]

Options:
  -l, --listen <LISTEN>
          Network address and port for the bridge service (http) to listen on (format: host:port) [default: 127.0.0.1:3000]
  -o, --ollama <OLLAMA>
          Ollama API endpoint URL for request forwarding (format: http(s)://host:port) [default: http://127.0.0.1:11434]
  -t, --timeout <TIMEOUT>
          Request timeout threshold in seconds for Ollama API communication (range: 1-86400) [default: 300]
  -d, --debug
          Enable detailed debug logging for request tracing and diagnostics
      --enable-cache
          Enable disk-based caching for LLM responses to improve performance and reduce API calls
      --cache-dir <CACHE_DIR>
          Directory path for storing cached LLM responses (format: path/to/cache) [default: ./cache]
      --cache-ttl <CACHE_TTL>
          Cache entry time-to-live in seconds for automatic expiration (range: 30-86400) [default: 300]
      --cache-max-entries <CACHE_MAX_ENTRIES>
          Maximum number of cache entries to maintain in storage (range: 100-1000000) [default: 65535]
      --disable-think-filter
          Disable think-tag filtering for DeepSeek R1 responses (default: enabled)
      --req-rate-limit <REQ_RATE_LIMIT>
          Maximum number of requests to Ollama API per second (0 for unlimited, range: 0-65535) [default: 0]
      --enable-shared-mode
          Enable shared mode for centralized API key management and authentication
      --api-key <API_KEY>
          API key for authentication in shared mode (required when shared mode is enabled)
  -h, --help
          Print help
  -V, --version
          Print version

```

### 3.2 配置参数详解

DeepSeek-Ollama Bridge 提供了一套完整的配置参数，让你能够根据实际需求灵活调整服务性能。以下是详细说明：

#### 基础配置参数

1. **监听地址设置 (-l, --listen)**

    - 参数格式：`host:port`
    - 默认配置：`127.0.0.1:3000`
    - 功能说明：配置服务监听的网络地址和端口。如需允许外部访问，请设置为 `0.0.0.0:3000`
    - 使用示例：`--listen 0.0.0.0:3000`

2. **Ollama 服务端点 (-o, --ollama)**

    - 参数格式：`http(s)://host:port`
    - 默认配置：`http://127.0.0.1:11434`
    - 功能说明：设置 Ollama API 服务地址，支持 HTTP 和 HTTPS 协议
    - 使用示例：`--ollama https://api.deepseek.com`

3. **请求超时设置 (-t, --timeout)**

    - 可选范围：1-86400 秒
    - 默认配置：300 秒
    - 功能说明：控制与 Ollama API 通信的最大等待时间，建议根据网络状况适当调整
    - 使用示例：`--timeout 600`

4. **调试模式开关 (-d, --debug)**
    - 默认状态：关闭
    - 功能说明：开启详细的调试日志，帮助你追踪请求流程和诊断问题
    - 使用示例：`--debug`

#### 缓存系统配置

5. **缓存功能开关 (--enable-cache)**

    - 默认状态：关闭
    - 功能说明：启用磁盘缓存机制，显著提升响应速度并降低 API 调用频率
    - 使用示例：`--enable-cache`

6. **缓存存储路径 (--cache-dir)**

    - 参数格式：`path/to/cache`
    - 默认配置：`./cache`
    - 功能说明：指定缓存文件的存储位置，请确保目录具备读写权限
    - 使用示例：`--cache-dir /data/llm/cache`

7. **缓存有效期设置 (--cache-ttl)**

    - 可选范围：30-86400 秒
    - 默认配置：300 秒
    - 功能说明：设定缓存数据的有效期限，超期数据将自动失效
    - 使用示例：`--cache-ttl 3600`

8. **缓存容量限制 (--cache-max-entries)**
    - 可选范围：100-1000000 条
    - 默认配置：65535 条
    - 功能说明：限定缓存条目的最大数量，超出限制时将启用 LRU 清理机制
    - 使用示例：`--cache-max-entries 100000`

#### 高级功能配置

9. **思考标签过滤器 (--disable-think-filter)**

    - 默认状态：启用
    - 功能说明：控制是否过滤 DeepSeek R1 模型输出中的思考标签
    - 使用示例：`--disable-think-filter`

10. **请求速率控制 (--req-rate-limit)**
    - 可选范围：0-65535（0 代表不限制）
    - 默认配置：0
    - 功能说明：控制每秒发送至 Ollama API 的最大请求数，用于保护服务稳定性
    - 使用示例：`--req-rate-limit 100`

#### 共享模式配置

11. **共享模式开关 (--enable-shared-mode)**

    -   默认状态：关闭
    -   功能说明：启用企业级的 API 密钥集中管理功能
    -   使用示例：`--enable-shared-mode`

12. **API 密钥配置 (--api-key)**
    -   参数格式：字符串
    -   功能说明：配置共享模式下的认证密钥
    -   使用示例：`--api-key your_api_key`

#### 使用建议

-   时间类参数均采用秒作为单位，便于统一管理
-   启用缓存可大幅提升性能，但请预留足够的磁盘空间
-   生产环境建议开启共享模式，实现 API 密钥的统一管理
-   调试模式建议仅在开发测试阶段使用，避免日志过多
-   请求速率限制要与服务器性能和业务需求相匹配

### 3.3 配置示例

以下是一个访问 DeepSeek 官方服务的完整配置示例：

```bash
./deepseek-ollama-bridge -l 127.0.0.1:3000 -o https://api.deepseek.com -t 300 --enable-cache --cache-dir ./cache --cache-ttl 30 --enable-shared-mode --api-key your_api_key
```

### 4. 服务调用

**重要提示：** 请将示例中的 `http://127.0.0.1:3000` 替换为你实际配置的服务地址和端口。

```bash
$ curl -N http://127.0.0.1:3000/v1/chat/completions -H "Content-Type: application/json" -d '{
  "model": "deepseek-r1",
  "messages": [{"role": "user", "content": "your message1"}],
  "stream": true
}'
```

**返回示例：**

```bash
data: {"id":"chatcmpl-0","object":"chat.completion.chunk","created":1739943594,"model":"deepseek-r1","system_fingerprint":null,"choices":[{"index":0,"delta":{"role":"assistant","content":"It seems like there might be some confusion in your message. If you're referring to a previous part of a story or a specific request, feel free to clarify or provide more context! For example:\n\n- Are you asking me to continue a story we were working on earlier?  \n- Do you need help brainstorming ideas?  \n- Or is this part of a different task?  \n\nLet me know how I can assist! 😊"},"finish_reason":null}]}

data: {"id":"chatcmpl-0","object":"chat.completion.chunk","created":1739943594,"model":"deepseek-r1","system_fingerprint":null,"choices":[{"index":0,"delta":{"role":"assistant","content":""},"finish_reason":"stop"}]}

data: [DONE]
```

**运行日志分析**

以下是一段典型的运行日志，你可以通过这些信息来了解请求的处理过程：

```bash
2025-02-19T05:40:20.145064Z  INFO deepseek_ollama_bridge::handlers: 51: [req-3] Method: POST, Path: /v1/chat/completions, Protocol: HTTP/1.1
2025-02-19T05:40:20.145135Z  INFO deepseek_ollama_bridge::handlers: 108: [req-3] Forwarding request to: https://api.lkeap.cloud.tencent.com/v1/chat/completions
2025-02-19T05:40:20.145167Z DEBUG deepseek_ollama_bridge::handlers: 109: [req-3] Request headers: {"host": "127.0.0.1:3000", "user-agent": "curl/8.10.1", "accept": "*/*", "content-type": "application/json", "content-length": "108"}
2025-02-19T05:40:20.145199Z  INFO deepseek_ollama_bridge::handlers: 114: [req-3] Request body size: 108 bytes
2025-02-19T05:40:20.145225Z DEBUG deepseek_ollama_bridge::handlers: 115: [req-3] Request body content: {
  "model": "deepseek-r1",
  "messages": [{"role": "user", "content": "your message1"}],
  "stream": true
}
2025-02-19T05:40:20.145277Z  INFO deepseek_ollama_bridge::handlers: 128: [req-3] Private mode is enabled, using custom API key for authorization
2025-02-19T05:40:20.145302Z  INFO deepseek_ollama_bridge::handlers: 131: [req-3] Converted request headers, Content-Type: Some("application/json")
2025-02-19T05:40:20.145331Z DEBUG deepseek_ollama_bridge::handlers: 146: [req-3] Streaming response detected
2025-02-19T05:40:20.145365Z  INFO deepseek_ollama_bridge::cache: 309: Cache Manager - Memory cache hit, Key: c65aad7d2feceedf0498615bad07f1f4, Latency: 2.50µs
2025-02-19T05:40:20.145395Z  INFO deepseek_ollama_bridge::cache: 522: Cache Manager - Context cache hit with prefix length 1 (stream_mode: true)
2025-02-19T05:40:20.145423Z DEBUG deepseek_ollama_bridge::cache: 523: Cache Manager - Context cache hit with prefixs: [user:your message1]
2025-02-19T05:40:20.145450Z DEBUG deepseek_ollama_bridge::handlers: 304: [req-3] Context cache hit
2025-02-19T05:40:20.145479Z  INFO deepseek_ollama_bridge::response: 49: [req-3] Final response status: 200 OK
2025-02-19T05:40:20.145510Z DEBUG deepseek_ollama_bridge::response: 90: [req-3] Final response headers: {"content-type": "text/event-stream", "connection": "keep-alive", "cache-control": "no-cache", "content-length": "836"}
2025-02-19T05:40:20.145538Z  INFO deepseek_ollama_bridge::response: 91: [req-3] Final response body size: 836 bytes
2025-02-19T05:40:20.145566Z  INFO deepseek_ollama_bridge::handlers: 76: [req-3] LLM request completed successfully, Latency: 502.70µs
```

从日志中可以看到 `Cache Manager - Context cache hit with prefixs: [user:your message1]` 的提示，这表明系统成功命中了缓存，有效提升了响应速度。
