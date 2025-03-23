# 🚀 DeepSeek-Ollama Bridge：让你的 AI 对话更快、更稳、更省心！

## 😫 你是否遇到过这些烦恼？

-   大语言模型本地部署后重复计算相同问题，算力资源严重浪费
-   高并发场景下系统不稳定，响应延迟大幅波动
-   模型输出夹杂思考标签，影响对话体验
-   服务器资源告急，性能调优无从下手

## 🎯 解决方案来了！

DeepSeek-Ollama Bridge 是一款面向多模型场景的高性能代理服务，全面支持 DeepSeek、QWen、GPT4o、Gemini、Claude 等主流大语言模型，助力 AI 应用性能提升。

本服务基于 OpenAI API 规范构建，集成高效缓存系统，提供共享 API Key 管理功能，支持 LuaJIT 运行时环境，并可通过自定义 Lua 脚本实现功能扩展。在安全性、性能表现及易用性等方面，为 AI 应用提供全方位优化支持。

### 🎁 核心特性

#### 1️ 智能多层缓存系统

-   高性能内存热点缓存（1024 条），极速响应
-   磁盘持久化存储，不受内存大小限制，支持百万级缓存
-   对话上下文感知，智能匹配历史应答
-   自动化清理机制，无需人工维护
-   灵活的缓存参数配置，轻松应对各类场景
-   注意：流式输出模式（stream=true）从 v0.1.15 版本开始支持

#### 2️ 成熟的流量控制

-   令牌桶限流保护，防止系统过载
-   Prometheus 指标监控，运行状态一目了然

#### 3️ 容器部署支持

-   支持 Kubernetes 集群部署
-   优雅启停机制
-   跨平台兼容性支持
-   完整的监控指标

#### 4️ 智能思考标签过滤

-   支持 DeepSeek、QWen、LLAMA 等主流模型的思考标签过滤系统
-   自动识别并移除模型输出中的思考过程标记
-   保持输出内容的专业性和连贯性
-   零延迟处理，不影响模型响应速度

#### 5️ 企业级共享模式（从 v0.1.15 版本开始）

-   API Key 集中托管与管理

#### 6️ LuaJIT 运行时支持（从 v0.1.16 版本开始）

-   内置高性能 LuaJIT 运行时环境
-   支持自定义 Lua 脚本扩展系统功能
-   毫秒级响应，适用于实时处理场景

### 💪 为什么选择 DeepSeek-Ollama Bridge ？

-   支持 DeepSeek、QWen、GPT4o、Gemini、Claude 等主流大语言模型
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

#### 4. 企业级共享模式

![企业级共享模式](./images/enterprise-mode.png)

#### 5. LuaJIT 运行时

![LuaJIT 运行时](./images/luajit-runtime.png)

#### 6. 请求后端服务 Retry

![请求后端服务 Retry](./images/http-request-retry.png)

## 🎁 快速开始

只需一行命令，即可启动企业级 AI 加速服务：

```bash
deepseek-ollama-bridge --enable-cache --cache-dir ./cache
```

更多高级配置选项请使用 `-h` 参数查看帮助文档。

_注：实际性能提升因使用场景和配置而异。欢迎留言反馈问题和改进建议。_

## �� 中间件

从 **v0.1.16** 版本开始，支持 LuaJIT 运行时，支持自定义 Lua 脚本扩展系统功能。

-   **HandleServerRequest 函数**：处理客户端发送的请求
-   **HandleServerResponse 函数**：处理服务端返回的响应

> [!NOTE]
>
> 下面提供一个示例，其中 `function HandleServerRequest(request)` 和 `function HandleServerResponse(response)` 函数是必须实现的。要不然会报错，如果没有逻辑可以编写一个空函数。

### plugin.lua 代码示例

```lua
-- 处理请求
-- @param request table 请求对象
--   - request.path string 请求路径 (例如: "/api/users")
--   - request.method string 请求方法 (例如: "GET", "POST")
--   - request.headers table 请求头
--     - 键为小写的header名称 (例如: "content-type", "authorization")
--     - 值为对应的header值
--   - request.body string 请求体内容
-- @return boolean, table 返回两个值:
--   1. boolean:
--      - true: 继续处理请求
--      - false: 拦截请求，返回自定义响应
--   2. table:
--      当第一个返回值为 true 时:
--        - 返回修改后的 request 表
--      当第一个返回值为 false 时:
--        - status_code number HTTP状态码
--        - body string 响应体
--        - headers table 响应头
function HandleServerRequest(request)

    -- 示例：记录请求
    print("Handling request to: " .. request.path)

    -- 示例：修改请求头
    request.headers["x-modified-by"] = "Lua-Middleware-Request"

    -- 示例：拦截特定请求
    if request.path == "/blocked" then
        return false, {
            status_code = 403,
            body = "Access denied by Lua middleware",
            headers = {
                ["content-type"] = "text/plain"
            }
        }
    end

    -- 继续处理请求
    return true, request
end

-- 处理响应
-- @param response table 响应对象
--   - response.status_code number HTTP状态码 (例如: 200, 404)
--   - response.headers table 响应头
--     - 键为小写的header名称 (例如: "content-type")
--     - 值为对应的header值
--   - response.body string 响应体内容
-- @return table 返回修改后的响应对象
--   - status_code number 修改后的HTTP状态码
--   - headers table 修改后的响应头
--   - body string 修改后的响应体内容
function HandleServerResponse(response)

    -- 示例：添加响应头
    response.headers["x-powered-by"] = "Lua-Middleware-Response"

    -- 调试日志：打印Content-Type
    print("Response Content-Type:", response.headers["content-type"])

    -- 示例：修改响应体
    if response.headers["content-type"] == "text/plain" then
        response.body = response.body .. "\n(Modified by Lua)"
    end

    -- 返回修改后的响应对象
    return response
end
```

### 支持第三方 lua 库和 dll 和 so 文件

-   支持挂载外部 lua 库
-   支持挂载外部 dll 文件
-   支持挂载外部 so 文件

```lua
-- plugin.lua - 主 Lua 脚本
-- 使用自定义模块处理请求和响应

-- 加载自定义模块
local cjson = require('cjson')
local utils = require("utils")

-- 调试模式
local DEBUG = true

-- 处理服务器请求的函数
-- 返回值：
-- 1. 布尔值，表示是否继续处理请求（true 表示继续，false 表示拦截）
-- 2. 请求表，如果拦截请求，可以包含自定义响应
function HandleServerRequest(request)
    local jsonData = {
        name = "Alice",
        age = 30,
        address = {
            city = "Shanghai",
            zip = "200000"
        },
        hobbies = {"reading", "traveling", "coding"},
        isStudent = false
    }

    local jsonBody = cjson.encode(jsonData)

    print(jsonBody)

    -- 继续处理请求
    return true, request
end

-- 处理服务器响应的函数
-- 返回值：修改后的响应表
function HandleServerResponse(response)
    return response
end
```

## 💡 访问代码示例

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
