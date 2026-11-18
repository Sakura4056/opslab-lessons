---
类型: 学习讲义
阶段: 5
标签: [学习讲义, 监控, Prometheus, CI/CD]
---

# 监控与 CI/CD 概念

> 阶段笔记：[[阶段5_Docker与自动化]] ｜ 上一讲：[[阶段5_Docker与自动化/04_Ansible入门|Ansible入门]] ｜ 下一阶段：[[阶段6_求职冲刺/01_面试八股速览|面试八股速览]]
> 2026 招聘硬要求：**会用 Prometheus/Grafana + 懂 CI/CD 流程**。本课讲概念+最小体验，深入留给入职。

## 学习目标

- 讲清监控三件套的分工与数据流
- 用 Prometheus + Grafana 监控一台主机（容器化 10 分钟体验）
- 描述一条标准 CI/CD 流水线，动手做一个 GitHub Actions 示例

## 核心概念

### 监控三件套

```
被监控主机(exporter 暴露 /metrics)
   → Prometheus 定时拉取并存储时序数据
   → Grafana 读 Prometheus 画看板
   → Alertmanager 按规则发告警（分级：警告/严重）
日志：Filebeat → Elasticsearch → Kibana（ELK/EFK）
```

| 指标类型 | 例子 | 用途 |
|---|---|---|
| CPU/内存/磁盘 | node_exporter | 主机健康 |
| QPS/延迟/错误率 | 应用或网关 | 服务健康（USE/RED 方法） |
| 告警规则 | `disk_used > 85% 持续 10m` | 先于用户发现问题 |

### CI/CD 流水线

```
代码 push → CI 构建+测试（自动）→ 打镜像/产物 → CD 部署（自动/审批）
工具：GitHub Actions / GitLab CI / Jenkins
```

## 命令实操

### 最小监控体验（Compose 起 Prometheus + Grafana + node_exporter）

```yaml
# ~/monitor/docker-compose.yml
services:
  node:
    image: prom/node-exporter:latest
    pid: host
    volumes: ["/:/host:ro"]
    command: ["--path.rootfs=/host"]
  prom:
    image: prom/prometheus:latest
    volumes: ["./prometheus.yml:/etc/prometheus/prometheus.yml:ro"]
    ports: ["9090:9090"]
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
```

```yaml
# ~/monitor/prometheus.yml
global: { scrape_interval: 15s }
scrape_configs:
  - job_name: node
    static_configs: [{ targets: ["node:9100"] }]
```

```bash
docker compose up -d
# Grafana: http://localhost:3000（admin/admin）→ 添加 Prometheus 数据源(地址 http://prom:9090)
#          → 导入仪表盘 ID 1860（Node Exporter Full）→ 看到本机 CPU/内存曲线即成功
# PromQL 体验: 浏览器开 http://localhost:9090 → 查询 100 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))*100
```

### CI/CD 最小体验（GitHub Actions）

```yaml
# .github/workflows/ci.yml —— 项目②仓库里加
name: ci
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: 构建 Docker 镜像
        run: docker build -t myapp:${{ github.sha }} .
      - name: 冒烟测试
        run: docker run -d --name t -p 8000:8000 myapp:${{ github.sha }} && sleep 3 && curl -sf localhost:8000/
```

push 后到仓库 Actions 页看绿色对勾——这就是 CI 的全部直觉。

## 动手实验

- [ ] Compose 起三件套，Grafana 导入 1860 看板并截图进实验日志
- [ ] PromQL 手查：本机内存使用率、磁盘剩余
- [ ] 给项目②仓库加 ci.yml，push 触发一次成功构建
- [ ] 写 5 条告警规则草稿（磁盘/CPU/内存/服务探活/证书有效期）存入项目②文档

## 常见坑

- **告警风暴**：阈值太敏感+无分级——按「警告/严重」分级并收敛；
- **只监控不排错**：监控答「哪里不对」，日志答「为什么」——两者要能互相跳转；
- **Grafana 数据源地址**：Compose 内用服务名 `http://prom:9090`，不是 localhost；
- **CI 跑在别人的机器**：密钥/凭证用平台的 secrets，绝不写进仓库。

## 自测

1. Prometheus/Grafana/Alertmanager 的分工与数据流？
2. 一条告警规则由哪几部分构成？
3. CI 和 CD 的分界在哪？为什么要有「审批」？
4. USE / RED 方法分别监控什么？

## 延伸

- 快照：[[Kubernetes官方文档|K8s 概念篇]]（监控在 K8s 生态的形态：kube-state-metrics 等）
- 阶段 5 毕业 → [[阶段6_求职冲刺/01_面试八股速览|阶段 6：面试八股速览]]

---
*相邻：[[阶段5_Docker与自动化/04_Ansible入门|← 上一讲]] ｜ [[阶段6_求职冲刺/01_面试八股速览|下一讲 →]]*
