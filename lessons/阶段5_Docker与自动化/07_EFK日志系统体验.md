---
类型: 学习讲义
阶段: 5
标签: [学习讲义, EFK, 日志, Grafana]
---

# EFK 日志系统体验

> 阶段笔记：[[阶段5_Docker与自动化]] ｜ 上一讲：[[阶段5_Docker与自动化/06_K8s概念与本地体验|K8s概念与本地体验]] ｜ 下一讲：[[阶段6_求职冲刺/01_面试八股速览|面试八股速览]]
> JD 常写「熟悉 ELK/EFK」。本课先用**轻量版（Loki+Promtail+Grafana）**做一次完整体验，再讲清 EFK 全量的架构与分工。

## 学习目标

- 讲清日志集中化的架构：采集 → 传输 → 存储 → 查询
- 用 Docker Compose 跑通一套日志系统并查到容器日志
- 能对 ELK 与 EFK/Loki 的取舍说出所以然

## 核心概念

```
EFK：Filebeat(采集) → Elasticsearch(存储+全文检索) → Kibana(查询看板)
ELK：Logstash(采集+加工) 替代 Filebeat
EFK/Loki（轻量替代）：Promtail(采集) → Loki(仅存标签+索引，不全文索引) → Grafana(查询)
```

| 方案 | 资源 | 全文检索 | 适用 |
|---|---|---|---|
| ELK/EFK | 重（ES 数 GB 内存） | 强 | 日志量大、要复杂检索 |
| Loki | 轻（只索引标签） | 弱（按标签查） | 容器/K8s 场景，配合监控 |

## 命令实操

### 轻量版：Loki + Promtail + Grafana

```yaml
# ~/logs/docker-compose.yml
services:
  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]
    command: -config.file=/etc/loki/local-config.yaml
  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command: -config.file=/etc/promtail/config.yml
  grafana:
    image: grafana/grafana:latest
    ports: ["3001:3000"]
```

```bash
docker compose up -d
# Grafana: http://localhost:3001（admin/admin）
# → Connections → Data sources → 添加 Loki（URL: http://loki:3100）
# → Explore → Label 选 container → 查询某容器的日志流
# LogQL 体验：{container="web"} |= "error"（按标签过滤+行内关键词）
```

### 常用查询（LogQL）

```logql
{container="app"} |= "ERROR"              # 含 ERROR 的行
{container="app"} |= "GET" != "healthz"   # 排除健康检查噪音
sum_rate({container="app"}[5m])           # 日志速率（突然暴涨=异常）
```

### EFK 全量版的架构要点（认知级）

- Filebeat 以 DaemonSet/容器方式跟随日志文件，输出到 ES；
- ES 按 **索引** 分天存储（`logstash-2026.11.19`），查询用 Kibana 的 KQL；
- 运维职责：索引生命周期管理（ILM，7 天热/30 天冷后删除，防磁盘满）、分片与副本规划。

## 动手实验

- [ ] Compose 起三件套，Grafana 数据源与 Explore 查询跑通
- [ ] 找到 nginx 容器的日志流，用 LogQL 过滤出 404 的请求行
- [ ] 对比：`docker logs`、`docker compose logs`、Grafana 查询三者的适用场景
- [ ] 写「日志系统选型建议」段落入项目②文档（何时上 ELK、何时 Loki）

## 常见坑

- **Promtail 采不到日志**：挂载路径必须是宿主 docker containers 目录，且容器要 json-file 驱动；
- **ES 内存**：生产 ES 至少 4GB+ 堆，本地体验别直接上；
- **日志时间线**：容器时区 UTC，查询时注意时间范围；
- **只查不删**：无 ILM 时磁盘被日志吃满——保留期必须配置。

## 自测

1. 画出 EFK 的四层组件流。
2. Loki 为什么比 ES 轻？代价是什么？
3. LogQL 查「app 容器最近 5 分钟含 ERROR 的日志速率」？
4. 日志保留期与磁盘管理的关系？

## 延伸

- 讲义：[[阶段5_Docker与自动化/05_监控与CI_CD概念|监控与CI/CD概念]]（日志+指标=可观测性双支柱）
- 阶段 5 全部完成 → [[阶段6_求职冲刺/01_面试八股速览|阶段 6：面试八股速览]]

---
*相邻：[[阶段5_Docker与自动化/06_K8s概念与本地体验|← 上一讲]] ｜ [[阶段6_求职冲刺/01_面试八股速览|下一讲 →]]*
