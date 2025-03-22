-   **0.1.16-rc3 版本**：

    -   增加请求后端服务 Retry 功能，支持次数、间隔等配置。
    -   增加请求后端服务 Retry 相关 Prometheus 指标。

-   **0.1.16-rc2 版本**：

    -   更新 Luajit 版本为 Openresty 的 luajit2（v210.5.12+a4f56a4）
    -   增加 luajit-package-path 和 luajit-package-cpath 配置项
    -   支持 Luajit 挂载外部 dll 和 so 文件和 lua 文件

-   **0.1.16-rc1 版本**：

    -   增加 Luajit 运行时，支持 LuaJIT 脚本运行。
    -   增加 Luajit 运行相关 Prometheus 指标。

-   **0.1.15 版本**：

    -   优化代码执行效率
    -   少量 BUG 修复

-   **0.1.15-rc3 版本**：

    -   优化缓存工作逻辑，提升缓存命中率。
    -   磁盘缓存能够更加主动地进行命中。

-   **0.1.15-rc2 版本**：

    -   内存热点缓冲的 TTL 最大不超过 5 分钟。
    -   内存热点缓冲的 TTL 默认为磁盘缓存 TTL 的 1/2。
    -   修复内存热点缓冲与磁盘缓存交互时存在的操作逻辑问题。

-   **0.1.15-rc1 版本**：

    -   支持流式输出模式（stream=true）缓存。
    -   支持企业级共享模式（API Key 集中托管）。

-   **0.1.14 版本**：

    -   支持非流模式（stream=false）缓存。
    -   支持容器部署。
    -   支持 Prometheus 指标监控。
    -   支持 Kubernetes 部署。
