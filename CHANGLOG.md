# 更新日志

## 0.1.16 版本

### 0.1.16-rc5

-   增加异步请求支持
-   增加异步请求的幂等性支持
-   增加异步请求的缓存支持
-   增加异步请求的限流支持

### 0.1.16-rc4

-   修正 `ollama_request_errors_total` 和 `ollama_request_retry_attempts_total` 的计算方法
-   统一计算时间和单位
-   完善不同时间检验和验证方法

### 0.1.16-rc3

-   增加请求后端服务的重试功能，支持次数、间隔等配置
-   增加请求后端服务重试相关的 Prometheus 指标

### 0.1.16-rc2

-   更新 Luajit 版本为 Openresty 的 luajit2（v210.5.12+a4f56a4）
-   增加 `luajit-package-path` 和 `luajit-package-cpath` 配置项
-   支持 Luajit 挂载外部 dll、so 文件和 lua 文件

### 0.1.16-rc1

-   增加 Luajit 运行时，支持 LuaJIT 脚本运行
-   增加 Luajit 运行相关的 Prometheus 指标

## 0.1.15 版本

### 0.1.15

-   优化代码执行效率
-   少量 BUG 修复

### 0.1.15-rc3

-   优化缓存工作逻辑，提升缓存命中率
-   磁盘缓存能够更加主动地进行命中

### 0.1.15-rc2

-   内存热点缓冲的 TTL 最大不超过 5 分钟
-   内存热点缓冲的 TTL 默认为磁盘缓存 TTL 的 1/2
-   修复内存热点缓冲与磁盘缓存交互时存在的操作逻辑问题

### 0.1.15-rc1

-   支持流式输出模式（stream=true）缓存
-   支持企业级共享模式（API Key 集中托管）

## 0.1.14 版本

-   支持非流模式（stream=false）缓存
-   支持容器部署
-   支持 Prometheus 指标监控
-   支持 Kubernetes 部署
