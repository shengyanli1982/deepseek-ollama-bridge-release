# 🚀 DeepSeek-Ollama Bridge：让你的 AI 对话更快、更稳、更省心！

## 😫 你是否遇到过这些烦恼？

-   DeepSeek 模型本地部署后重复计算相同问题，算力资源严重浪费
-   高并发场景下系统不稳定，响应延迟大幅波动
-   模型输出夹杂思考标签，影响对话体验
-   服务器资源告急，性能调优无从下手

## 🎯 完美解决方案来了！

DeepSeek-Ollama Bridge 是一款专为 DeepSeek 模型打造的高性能桥接服务，让您的 AI 应用如虎添翼！

### 🎁 核心特性

#### 1️⃣ 智能多层缓存系统

-   🚄 高性能内存热点缓存（1024 条），极速响应
-   💾 磁盘持久化存储，不受内存大小限制，支持百万级缓存
-   🧠 对话上下文感知，智能匹配历史应答
-   🧹 自动化清理机制，无需人工维护
-   📦 灵活的缓存参数配置，轻松应对各类场景

#### 2️⃣ 成熟的流量控制

-   🚦 令牌桶限流保护，防止系统过载
-   📊 Prometheus 指标监控，运行状态一目了然

#### 3️⃣ 容器部署支持

-   🎯 支持 Kubernetes 集群部署
-   🔄 优雅启停机制
-   🌐 跨平台兼容性支持
-   📈 完整的监控指标

### 💪 为什么选择 DeepSeek-Ollama Bridge ？

-   🎯 为 DeepSeek 模型优化，同时兼容其他 OpenAI API 规范的模型
-   🛠️ 开箱即用，可以零配置启动
-   📈 显著提升响应速度，降低计算成本
-   🔄 自动过滤思考标签（专门针对 DeepSeek 蒸馏模型），输出更清晰专业

### 🎬 典型应用场景

1. **高频对话场景**

    - 智能客服系统
    - 教育问答平台
    - API 集成服务

2. **资源受限环境**

    - 个人开发环境
    - 边缘计算设备
    - 共享计算集群

3. **企业级应用**
    - 大规模 AI 服务部署
    - 多租户并发访问
    - 成本敏感型业务

### 📊 效果展示

#### 1. 缓存效果

![缓存效果](./images/cache.png)

#### 2. 流量控制

![流量控制](./images/rate-limit.png)

## 🎁 快速开始

只需一行命令，即可启动企业级 AI 加速服务：

```bash
deepseek-ollama-bridge --enable-cache --cache-dir ./cache
```

更多高级配置选项请使用 `-h` 参数查看帮助文档。

_注：实际性能提升因使用场景和配置而异。欢迎留言反馈问题和改进建议。_
