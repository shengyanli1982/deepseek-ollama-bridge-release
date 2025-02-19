# Kubernetes 部署指南

欢迎使用 DeepSeek-Ollama Bridge 的 Kubernetes 部署指南！本文档将为你详细介绍如何在 Kubernetes 集群中部署和运维 DeepSeek-Ollama Bridge 服务。我们将使用 GitHub Container Registry (ghcr.io) 作为容器镜像仓库，并为你提供完整的部署流程。

## 环境准备

在开始部署之前，请确保你的环境满足以下要求：

-   一个功能正常的 Kubernetes 集群（版本需要 >= 1.16）
-   已正确安装并配置 kubectl 命令行工具
-   集群中有充足的计算资源配额
-   已完成 ghcr.io 的访问授权配置

## 部署流程

### 1. 配置镜像仓库访问凭证

首先，我们需要创建一个用于拉取容器镜像的访问凭证 Secret（你可以根据实际使用的镜像仓库来调整配置）：

```bash
# 创建 ghcr.io 的访问凭证 Secret
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_TOKEN \
  --docker-email=YOUR_EMAIL \
  -n your-namespace
```

### 2. 设置持久化存储

如果你需要启用缓存功能，请创建以下持久化存储声明文件 `deepseek-ollama-bridge-pvc.yaml`：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: deepseek-ollama-bridge-cache
    namespace: your-namespace
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
            storage: 10Gi
    storageClassName: standard # 根据你的集群配置修改
```

### 3. 部署服务组件

创建服务部署配置文件 `deepseek-ollama-bridge-deployment.yaml`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: deepseek-ollama-bridge
    namespace: your-namespace
spec:
    replicas: 2 # 根据需求调整副本数
    selector:
        matchLabels:
            app: deepseek-ollama-bridge
    template:
        metadata:
            labels:
                app: deepseek-ollama-bridge
        spec:
            containers:
                - name: deepseek-ollama-bridge
                  image: ghcr.io/shengyanli1982/deepseek-ollama-bridge:v0.1.15
                  args:
                      - "-l"
                      - "0.0.0.0:3000"
                      - "-o"
                      - "http://your-ollama-service:11434"
                      - "-t"
                      - "300"
                      - "--enable-cache"
                      - "--cache-dir"
                      - "/app/cache"
                      - "--cache-ttl"
                      - "300"
                      - "--cache-max-entries"
                      - "65535"
                      - "--enable-shared-mode"
                      - "--api-key"
                      - "your-api-key"
                      - "--req-rate-limit"
                      - "100"
                      - "--debug"
                  ports:
                      - name: web
                        containerPort: 3000
                        protocol: TCP
                  livenessProbe:
                      httpGet:
                          path: /health
                          port: web
                      initialDelaySeconds: 15
                      periodSeconds: 10
                      timeoutSeconds: 5
                      failureThreshold: 3
                  readinessProbe:
                      httpGet:
                          path: /health
                          port: web
                      initialDelaySeconds: 5
                      periodSeconds: 10
                      timeoutSeconds: 5
                      failureThreshold: 3
                  volumeMounts:
                      - name: cache-volume
                        mountPath: /app/cache
                  resources:
                      requests:
                          memory: "128Mi"
                          cpu: "128m"
                      limits:
                          memory: "4Gi" # 根据需求调整
                          cpu: "2" # 根据需求调整
            volumes:
                - name: cache-volume
                  persistentVolumeClaim:
                      claimName: deepseek-ollama-bridge-cache
            imagePullSecrets:
                - name: ghcr-secret
---
apiVersion: v1
kind: Service
metadata:
    name: deepseek-ollama-bridge
    namespace: your-namespace
spec:
    selector:
        app: deepseek-ollama-bridge
    ports:
        - name: web
          port: 80
          targetPort: web
          protocol: TCP
    type: ClusterIP # 根据需求可改为 LoadBalancer 或 NodePort
```

### 4. 执行部署

按照以下步骤依次执行部署命令：

```bash
# 创建专用命名空间（如果尚未创建）
kubectl create namespace your-namespace

# 应用配置文件
kubectl apply -f deepseek-ollama-bridge-pvc.yaml
kubectl apply -f deepseek-ollama-bridge-deployment.yaml

# 检查部署结果
kubectl get pods -n your-namespace
kubectl get services -n your-namespace
```

### 5. 验证服务

完成部署后，你可以通过以下方式验证服务是否正常运行：

```bash
# 创建本地端口转发（用于测试）
kubectl port-forward service/deepseek-ollama-bridge 3000:80 -n your-namespace

# 发送测试请求
curl -N http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-r1",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'
```

## 配置参数说明

服务启动时可以使用以下命令行参数进行配置：

| 参数                 | 说明                         | 默认值                 |
| -------------------- | ---------------------------- | ---------------------- |
| -l, --listen         | 服务监听的地址和端口         | 127.0.0.1:3000         |
| -o, --ollama         | Ollama API 服务的地址        | http://127.0.0.1:11434 |
| -t, --timeout        | 请求的超时时间（秒）         | 300                    |
| --enable-cache       | 是否启用磁盘缓存功能         | false                  |
| --cache-dir          | 缓存文件存储路径             | ./cache                |
| --cache-ttl          | 缓存数据的有效期（秒）       | 300                    |
| --cache-max-entries  | 缓存条目的最大数量           | 65535                  |
| --enable-shared-mode | 是否启用共享访问模式         | false                  |
| --api-key            | API 访问密钥（共享模式必需） | -                      |
| --req-rate-limit     | 每秒最大请求处理数           | 0                      |
| --debug              | 是否启用调试日志输出         | false                  |

### 参数配置建议

1. **服务监听配置**

    - 在 Kubernetes 环境中，建议使用 `0.0.0.0:3000` 以允许集群内外部访问

2. **缓存优化配置**

    - 强烈建议启用缓存以提升服务性能
    - 请将缓存目录配置在持久化存储卷上
    - 根据业务需求合理设置缓存有效期和最大条目数

3. **性能调优配置**

    - 通过 `--req-rate-limit` 合理控制请求并发量
    - 根据业务场景适当设置 `--timeout` 值，避免请求长时间挂起

4. **安全防护配置**
    - 建议开启 `--enable-shared-mode` 共享访问模式
    - 在生产环境中必须设置 `--api-key` 并确保其安全性

## 运维管理建议

1. **监控体系建设**

    - 建立完善的资源使用监控
    - 配置规范的日志采集方案
    - 合理设置健康检查机制：
        - `livenessProbe`: 用于检测服务存活状态，失败时触发容器重启
        - `readinessProbe`: 用于检测服务就绪状态，失败时将从负载均衡中剔除
        - 健康检查接口：`/health`
        - 推荐的探针配置参数：
            - 初始延迟：容器启动后 5-15 秒开始检查
            - 检查周期：每 10 秒执行一次
            - 超时设置：单次检查超过 5 秒判定失败
            - 失败阈值：需要连续 3 次失败才触发相应动作

2. **扩展性保障**

    - 根据实际负载情况动态调整副本数量
    - 配置 HPA（Horizontal Pod Autoscaling）实现自动扩缩容
    - 使用 PodDisruptionBudget 确保服务高可用性

3. **安全防护措施**
    - 保持容器镜像的定期更新
    - 实施严格的网络访问策略
    - 建立完善的 RBAC 权限控制体系

## 问题排查指南

如果你在部署过程中遇到问题，可以通过以下命令进行排查：

```bash
# 查看 Pod 详细状态
kubectl describe pod -l app=deepseek-ollama-bridge -n your-namespace

# 查看容器运行日志
kubectl logs -l app=deepseek-ollama-bridge -n your-namespace

# 检查存储卷状态
kubectl get pvc -n your-namespace
```

## 版本更新指南

当需要更新已部署的服务时，你可以使用以下方法：

```bash
# 更新服务镜像版本
kubectl set image deployment/deepseek-ollama-bridge \
  deepseek-ollama-bridge=ghcr.io/shengyanli1982/deepseek-ollama-bridge:new-version \
  -n your-namespace

# 或者重新应用更新后的配置文件
kubectl apply -f deepseek-ollama-bridge-deployment.yaml
```
