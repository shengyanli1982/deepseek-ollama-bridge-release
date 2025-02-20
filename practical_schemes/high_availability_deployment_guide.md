# DeepSeek-Ollama Bridge 多实例部署实践指南

## 写在前面

欢迎阅读 DeepSeek-Ollama Bridge 的多实例部署指南！本文将为您介绍如何搭建一个既简单又可靠的分布式模型服务。无论您是想在本地运行多个实例，还是计划使用云服务，这份指南都能帮您轻松实现目标。

## 方案设计

### 1. 整体架构

#### 1.1 一图胜千言

```ascii
                                    外部请求
                                       ⬇
                            +--------------------+
                            |       Nginx        |
                            |   负载均衡 + SSL    |
                            +--------------------+
                                ↙      ↓      ↘
                    +----------+  +----------+  +----------+
                    | Bridge#1 |  | Bridge#2 |  | Bridge#3 |
                    |  实例1   |  |  实例2    |  |  实例3   |
                    +----------+  +----------+  +----------+
                         ↓             ↓             ↓
                    +----------+  +----------+  +----------+
                    |  Cache1  |  |  Cache2  |  |  Cache3  |
                    +----------+  +----------+  +----------+
                         ↙              ↓              ↘
            +----------------+  +----------------+  +----------------+
            |   本地部署      |  |    阿里云      |  |    腾讯云       |
            | [DeepSeek模型]  |  | [Llama2模型]   |  | [Mistral模型]   |
            | 192.168.1.100  |  | xxx.aliyun.com |  | xxx.tencentyun |
            +----------------+  +----------------+  +----------------+
                      ↑                   ↑                   ↑
                +-----------+        +-----------+      +------------+
                | GPU:V100  |        |  GPU:A100 |      |  GPU:A100  |
                +-----------+        +-----------+      +------------+
```

#### 1.2 核心组件介绍

1. **智能负载均衡器**

    - 采用业界流行的 Nginx 作为流量"指挥官"
    - 灵活的分发策略：可以根据需求选择轮询、权重或最小连接
    - 内置健康检查：自动发现并规避故障节点
    - 专业的 SSL 证书管理：保障通信安全

2. **Bridge 服务集群**

    - 多个 Bridge 实例协同工作，互为备份
    - 每个实例都有独立的缓存空间，避免互相干扰
    - 支持动态扩容，轻松应对访问高峰
    - 实例间自动同步状态，保持数据一致

3. **模型服务后端**
    - 支持对接多家模型服务，灵活切换
    - 智能混合部署，兼顾性能和成本
    - 自动故障转移，确保服务持续可用
    - 资源智能调度，提升使用效率

### 2. 部署实战指南

#### 2.1 配置负载均衡

1. **Nginx 配置详解**

    ```nginx
    # nginx.conf - 您的"流量调度中心"
    stream {
        upstream bridge_cluster {
            # 主力实例（承担主要流量）
            server 127.0.0.1:3000 weight=2 max_fails=2 fail_timeout=30s;
            server 127.0.0.1:3001 weight=2 max_fails=2 fail_timeout=30s;

            # 备用实例（随时待命）
            server 127.0.0.1:3002 weight=1 max_fails=3 fail_timeout=60s backup;
            server 127.0.0.1:3003 weight=1 max_fails=3 fail_timeout=60s backup;

            least_conn;  # 采用最小连接数算法，让请求更均匀
            keepalive 32;  # 保持连接活跃，提升性能
        }
    }

    http {
        # 健康检查设置
        check {
            interval=3000;     # 每3秒巡检一次
            rise=2;           # 连续2次正常就上线
            fall=5;           # 连续5次异常就下线
            timeout=1000;     # 检查超时时间
            type=http;        # 检查方式
            port=3000;        # 检查端口
        }

        # 性能加速设置
        gzip on;                    # 启用压缩
        gzip_min_length 1k;         # 最小压缩体积
        gzip_types text/plain application/json;  # 压缩类型

        # 流量保护设置
        limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

        server {
            listen 80;
            server_name api.yourdomain.com;

            location / {
                # 智能请求分发
                proxy_pass http://bridge_cluster;

                # 连接优化
                proxy_http_version 1.1;
                proxy_set_header Connection "";

                # 故障转移设置
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
                proxy_next_upstream_tries 3;
                proxy_read_timeout 300s;

                # 实时健康检查
                check_http_send "HEAD / HTTP/1.0\r\n\r\n";
                check_http_expect_alive http_2xx http_3xx;
            }
        }
    }
    ```

#### 2.2 实例管理与监控

1. **Supervisor 进程守护配置**

    ```ini
    # deepseek-bridge.conf - 让您的服务永远在线
    [program:deepseek-bridge-1]
    command=/path/to/deepseek-ollama-bridge -l 127.0.0.1:3000 \
            --ollama http://127.0.0.1:11434 \
            --enable-cache \
            --cache-dir ./cache1 \
            --enable-shared-mode \
            --api-key your_key
    directory=/path/to/working/directory
    user=bridge-user
    autostart=true           # 开机自启
    autorestart=true        # 故障自动重启
    startretries=3          # 重试次数
    redirect_stderr=true    # 错误日志重定向
    stdout_logfile=/var/log/supervisor/deepseek-bridge-1.log
    stdout_logfile_maxbytes=50MB   # 日志大小限制
    stdout_logfile_backups=10      # 日志备份数量
    environment=RUST_LOG=info      # 日志级别设置

    [program:deepseek-bridge-1]
    command=/path/to/deepseek-ollama-bridge -l 127.0.0.1:3000 \
            --ollama http://127.0.0.1:11434 \
            --enable-cache \
            --cache-dir ./cache2 \
            --enable-shared-mode \
            --api-key your_key
    directory=/path/to/working/directory
    user=bridge-user
    autostart=true           # 开机自启
    autorestart=true        # 故障自动重启
    startretries=3          # 重试次数
    redirect_stderr=true    # 错误日志重定向
    stdout_logfile=/var/log/supervisor/deepseek-bridge-1.log
    stdout_logfile_maxbytes=50MB   # 日志大小限制
    stdout_logfile_backups=10      # 日志备份数量
    environment=RUST_LOG=info      # 日志级别设置

    [program:deepseek-bridge-2]
    command=/path/to/deepseek-ollama-bridge -l 127.0.0.1:3001 \
            --ollama https://cloud-provider-1.com/v1 \
            --enable-cache \
            --cache-dir ./cache2 \
            --enable-shared-mode \
            --api-key your_key
    directory=/path/to/working/directory
    user=bridge-user
    autostart=true           # 开机自启
    autorestart=true        # 故障自动重启
    startretries=3          # 重试次数
    redirect_stderr=true    # 错误日志重定向
    stdout_logfile=/var/log/supervisor/deepseek-bridge-2.log
    stdout_logfile_maxbytes=50MB   # 日志大小限制
    stdout_logfile_backups=10      # 日志备份数量
    environment=RUST_LOG=info      # 日志级别设置

    [program:deepseek-bridge-3]
    command=/path/to/deepseek-ollama-bridge -l 127.0.0.1:3002 \
            --ollama https://cloud-provider-2.com/v1 \
            --enable-cache \
            --cache-dir ./cache3 \
            --enable-shared-mode \
            --api-key your_key
    directory=/path/to/working/directory
    user=bridge-user
    autostart=true           # 开机自启
    autorestart=true        # 故障自动重启
    startretries=3          # 重试次数
    redirect_stderr=true    # 错误日志重定向
    stdout_logfile=/var/log/supervisor/deepseek-bridge-3.log
    stdout_logfile_maxbytes=50MB   # 日志大小限制
    stdout_logfile_backups=10      # 日志备份数量
    environment=RUST_LOG=info      # 日志级别设置

    [group:deepseek-bridges]
    programs=deepseek-bridge-1,deepseek-bridge-2,deepseek-bridge-3,deepseek-bridge-4
    priority=999
    ```

2. **Prometheus 监控配置**

    ```yaml
    # prometheus.yml - 为您的服务保驾护航
    global:
        scrape_interval: 15s # 全局采集间隔
        evaluation_interval: 15s # 规则评估间隔

    scrape_configs:
        - job_name: "deepseek-bridge"
          static_configs:
              - targets:
                    - "127.0.0.1:3000" # 实例1
                    - "127.0.0.1:3001" # 实例2
                    - "127.0.0.1:3002" # 实例3
                    - "127.0.0.1:3003" # 实例4
          metrics_path: "/metrics"
          scheme: http
          scrape_interval: 10s # 特定采集间隔
          scrape_timeout: 5s # 采集超时时间

          # 标签美化
          relabel_configs:
              - source_labels: [__address__]
                regex: "127.0.0.1:(.*)"
                target_label: "instance"
                replacement: "bridge-$1"
    ```

#### 2.3 轻松部署指南

1. **启动步骤**

    ```bash
    # 让我们开始吧！

    # 第一步：启动所有 Bridge 实例
    supervisorctl start deepseek-bridge-1
    supervisorctl start deepseek-bridge-2
    supervisorctl start deepseek-bridge-3
    supervisorctl start deepseek-bridge-4

    # 第二步：启动负载均衡器
    systemctl start nginx
    ```

2. **智能容错机制**

    - 单实例故障？无需担心，系统会自动切换到其他正常实例
    - 多实例故障？备用实例随时顶上
    - 负载激增？系统会自动扩展实例数量

3. **性能优化小贴士**
    - 开启 HTTP/2，提升传输效率
    - 合理设置缓存，减少重复计算
    - 设置请求限流，保护系统稳定

## 写在最后

恭喜您！通过本指南的配置，您已经搭建了一个可靠的多实例服务系统。它不仅能够应对大规模访问，还能在出现故障时自动恢复。记住，根据您的实际需求调整配置参数，持续监控和优化，让系统越来越强大！

如果您在部署过程中遇到任何问题，欢迎查阅我们的文档或在社区中寻求帮助。祝您部署顺利！
