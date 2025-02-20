# 🚀 DeepSeek-Ollama Bridge：让你的 AI 对话更快、更稳、更省心！

## 😫 你是否遇到过这些烦恼？

-   DeepSeek 模型本地部署后重复计算相同问题，算力资源严重浪费
-   高并发场景下系统不稳定，响应延迟大幅波动
-   模型输出夹杂思考标签，影响对话体验
-   服务器资源告急，性能调优无从下手

## 🎯 解决方案来了！

DeepSeek-Ollama Bridge 是一款专为 DeepSeek 模型打造的高性能代理（Proxy）服务，让您的 AI 应用如虎添翼！

### 🎁 核心特性

#### 1️ 智能多层缓存系统

-   高性能内存热点缓存（1024 条），极速响应
-   磁盘持久化存储，不受内存大小限制，支持百万级缓存
-   对话上下文感知，智能匹配历史应答
-   自动化清理机制，无需人工维护
-   灵活的缓存参数配置，轻松应对各类场景
-   注意：流式输出模式下从 v0.1.15 版本开始支持

#### 2️ 成熟的流量控制

-   令牌桶限流保护，防止系统过载
-   Prometheus 指标监控，运行状态一目了然

#### 3️ 容器部署支持

-   支持 Kubernetes 集群部署
-   优雅启停机制
-   跨平台兼容性支持
-   完整的监控指标

#### 4️ 智能思考标签过滤

-   专为 DeepSeek 蒸馏模型优化的思考标签过滤系统
-   自动识别并移除模型输出中的思考过程标记
-   保持输出内容的专业性和连贯性
-   零延迟处理，不影响模型响应速度

#### 5️ 企业级共享模式

-   API Key 集中托管与管理

### 💪 为什么选择 DeepSeek-Ollama Bridge ？

-   为 DeepSeek 模型优化，同时兼容其他 OpenAI API 规范的模型
-   开箱即用，可以零配置启动
-   显著提升响应速度，降低计算成本
-   自动过滤思考标签（专门针对 DeepSeek 蒸馏模型），输出更清晰专业

### 🎬 典型应用场景

#### 1️ 高频对话场景

-   智能客服系统
-   教育问答平台
-   API 集成服务

#### 2️ 资源受限环境

-   个人开发环境
-   边缘计算设备
-   共享计算集群

#### 3️ 企业级应用

-   大规模 AI 服务部署
-   多租户并发访问
-   成本敏感型业务
-   Key 托管

### 📚 支持文档

可以阅读 `docs` 目录下的文档，了解 DeepSeek-Ollama Bridge 的详细使用方法。

### 📁 实用方案

可以阅读 `practical_schemes` 目录下的文档，了解 DeepSeek-Ollama Bridge 的实用方案。

### 📝 版本更新

可以阅读 `CHANGLOG.md` 文件，了解 DeepSeek-Ollama Bridge 的版本更新历史。

### 🤖 默认接口

-   **/health** : 健康检查接口
-   **/metrics** : Prometheus 指标监控接口

### 📊 效果展示

#### 1. 缓存效果 (stream=false)

![缓存效果](./images/cache.png)

#### 2. 缓存效果 (stream=true)

![缓存效果](./images/cache-stream.png)

#### 3. 流量控制

![流量控制](./images/rate-limit.png)

## 🎁 快速开始

只需一行命令，即可启动企业级 AI 加速服务：

```bash
deepseek-ollama-bridge --enable-cache --cache-dir ./cache
```

更多高级配置选项请使用 `-h` 参数查看帮助文档。

_注：实际性能提升因使用场景和配置而异。欢迎留言反馈问题和改进建议。_

## 💡 代码示例

### cURL 示例

```bash
curl http://127.0.0.1:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxx" \
  -d '{
    "model": "deepseek-coder",
    "messages": [
      {
        "role": "user",
        "content": "写一个冒泡排序算法"
      }
    ],
    "temperature": 0.7
  }'
```

### Python 示例

```python
import openai

# 设置 API 基础地址（默认为本地服务）
openai.api_base = "http://127.0.0.1:3000/v1/chat/completions"
# 设置一个占位 API Key（本地服务不校验）
openai.api_key = "sk-xxx"

# 基础对话示例
def chat_example():
    response = openai.ChatCompletion.create(
        model="deepseek-coder",  # 使用 DeepSeek Coder 模型
        messages=[
            {"role": "user", "content": "写一个 Python 快速排序算法"}
        ],
        temperature=0.7
    )
    print(response.choices[0].message.content)

# 带上下文的对话示例
def context_chat_example():
    messages = [
        {"role": "system", "content": "你是一个专业的编程助手。"},
        {"role": "user", "content": "我想实现一个 REST API。"},
        {"role": "assistant", "content": "我可以帮你使用 FastAPI 框架实现。"},
        {"role": "user", "content": "好的，请给出具体示例。"}
    ]

    response = openai.ChatCompletion.create(
        model="deepseek-coder",
        messages=messages,
        temperature=0.7
    )

    print(response.choices[0].message.content)

if __name__ == "__main__":
    print("基础对话示例：")
    chat_example()

    print("\n带上下文的对话示例：")
    context_chat_example()
```

### Go 示例

```go
package main

import (
    "context"
    "fmt"
    "log"

    openai "github.com/sashabaranov/go-openai"
)

func main() {
    // 创建客户端（使用本地服务地址）
    client := openai.NewClient("sk-xxx")
    client.BaseURL = "http://127.0.0.1:3000/v1/chat/completions"

    // 创建对话请求
    req := openai.ChatCompletionRequest{
        Model: "deepseek-coder",
        Messages: []openai.ChatCompletionMessage{
            {
                Role:    openai.ChatMessageRoleUser,
                Content: "用 Go 实现一个简单的 HTTP 服务器",
            },
        },
        Temperature: 0.7,
    }

    // 发送请求
    resp, err := client.CreateChatCompletion(context.Background(), req)
    if err != nil {
        log.Printf("对话请求失败: %v\n", err)
        return
    }

    // 输出响应
    fmt.Println(resp.Choices[0].Message.Content)

    // 带上下文的对话示例
    contextReq := openai.ChatCompletionRequest{
        Model: "deepseek-coder",
        Messages: []openai.ChatCompletionMessage{
            {
                Role:    openai.ChatMessageRoleSystem,
                Content: "你是一个专业的 Go 开发专家。",
            },
            {
                Role:    openai.ChatMessageRoleUser,
                Content: "解释什么是依赖注入",
            },
        },
        Temperature: 0.7,
    }

    contextResp, err := client.CreateChatCompletion(context.Background(), contextReq)
    if err != nil {
        log.Printf("上下文对话请求失败: %v\n", err)
        return
    }

    fmt.Println("\n带上下文的对话响应:")
    fmt.Println(contextResp.Choices[0].Message.Content)
}
```

### NodeJS 示例

```javascript
const { Configuration, OpenAIApi } = require("openai");

// 配置 OpenAI API
const configuration = new Configuration({
    basePath: "http://127.0.0.1:3000/v1/chat/completions",
    apiKey: "sk-xxx",
});

const openai = new OpenAIApi(configuration);

// 基础对话示例
async function basicChatExample() {
    try {
        const response = await openai.createChatCompletion({
            model: "deepseek-coder",
            messages: [{ role: "user", content: "用 Express 实现一个 RESTful API" }],
            temperature: 0.7,
        });

        console.log("基础对话响应:", response.data.choices[0].message.content);
    } catch (error) {
        console.error("对话请求失败:", error.message);
    }
}

// 带上下文的对话示例
async function contextChatExample() {
    try {
        const response = await openai.createChatCompletion({
            model: "deepseek-coder",
            messages: [
                { role: "system", content: "你是一个专业的 Node.js 开发专家。" },
                { role: "user", content: "如何实现一个 WebSocket 服务器？" },
            ],
            temperature: 0.7,
        });

        console.log("\n带上下文的对话响应:");
        console.log(response.data.choices[0].message.content);
    } catch (error) {
        console.error("对话请求失败:", error.message);
    }
}

// 执行示例
async function main() {
    console.log("=== 基础对话示例 ===");
    await basicChatExample();

    console.log("\n=== 带上下文的对话示例 ===");
    await contextChatExample();
}

main().catch(console.error);
```
