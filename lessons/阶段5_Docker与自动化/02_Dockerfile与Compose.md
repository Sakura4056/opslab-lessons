---
类型: 学习讲义
阶段: 5
标签: [学习讲义, Docker, Dockerfile, Compose]
---

# Dockerfile 与 Compose

> 阶段笔记：[[阶段5_Docker与自动化]] ｜ 上一讲：[[阶段5_Docker与自动化/01_Docker核心概念与命令|Docker核心概念与命令]] ｜ 下一讲：[[阶段5_Docker与自动化/03_Python运维脚本|Python运维脚本]]
> 简历项目②核心：**LNMP 容器化改造**——一条 `docker compose up -d` 拉起全套。

## 学习目标

- 掌握 Dockerfile 十个高频指令与分层缓存原理
- 为阶段 4 的 Flask 应用写出生产级 Dockerfile
- 用 Compose 编排 nginx+app+mysql+redis 四服务并互相联通

## 核心概念

### Dockerfile 分层缓存

每条指令 = 一层。**变动少的指令写前面**（缓存复用），依赖安装在前、代码拷贝在后。

```dockerfile
# Dockerfile（放 /opt/app/）
FROM python:3.12-slim              # 基础镜像：slim 减体积
WORKDIR /app
COPY requirements.txt .            # 先拷依赖清单（不常变 → 缓存命中）
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .                      # 再拷代码（常变 → 只有这层重建）
EXPOSE 8000
USER www-data                      # 非 root 运行（安全基线）
CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:8000", "app:app"]
# CMD 容器默认启动命令；ENTRYPOINT 固定入口（参数可附加）
```

### Compose（多容器编排）

```yaml
# /opt/lnmp/docker-compose.yml
services:
  web:
    image: nginx:1.27-alpine
    ports: ["80:80"]
    volumes: ["./nginx.conf:/etc/nginx/conf.d/app.conf:ro"]
    depends_on: [app]
  app:
    build: ./app                   # 用上面的 Dockerfile 构建
    environment: [DB_HOST=db, REDIS_HOST=cache]
    depends_on: [db, cache]
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpw
      MYSQL_DATABASE: appdb
    volumes: ["dbdata:/var/lib/mysql"]
  cache:
    image: redis:7-alpine
volumes:
  dbdata:
```

服务间用**服务名做主机名**互访（app 连 `db:3306`、`cache:6379`）——Compose 自建网络。

## 命令实操

```bash
cd /opt/lnmp
docker compose build               # 构建镜像
docker compose up -d               # ⭐ 一条命令拉起全套（替代阶段 4 手工部署）
docker compose ps                  # 四个服务 Up
docker compose logs -f app         # 跟某服务日志
docker compose exec app sh         # 进容器
docker compose down                # 停止并移除（-v 连卷一起删！）
docker compose up -d --build       # 改代码后重建部署
```

## 动手实验（项目②主任务）

- [ ] 给阶段 4 的 Flask 应用写 Dockerfile 并 `docker build`，`docker run` 单容器跑通
- [ ] 写 Compose 编排四服务，`up -d` 后逐个验证：nginx 首页、/api/visit、redis 计数
- [ ] 改 app.py 一行代码，`up -d --build` 体会缓存复用（前面的层不重建）
- [ ] `down -v` 再 `up -d`：数据卷生命周期对比（有 dbdata vs 无）
- [ ] 项目②文档 `06_Projects\lnmp容器化\`：Dockerfile/Compose 讲解 + 与项目①部署方式对比，git tag `v2.0-容器化`

## 常见坑

- **COPY 顺序错误**：代码层在前 → 每次改代码全量重装依赖；
- **Compose 里 build 的相对路径**：`build: ./app` 相对于 yml 所在目录；
- **depends_on 只管启动顺序**：不等服务「就绪」，MySQL 刚起 app 就连会报错——应用加重试；
- **down 忘了卷还在/被删**：`down` 留卷，`down -v` 删卷——删前想清楚；
- **容器里连 localhost 连不上 MySQL**：要用服务名 `db`。

## 自测

1. Dockerfile 分层缓存怎么利用？COPY 顺序为什么重要？
2. CMD 和 ENTRYPOINT 的区别？
3. Compose 服务间如何互相访问？端口映射是给谁用的？
4. `docker compose down` 与 `down -v` 的差异？

## 延伸

- 快照：[[Docker实践_快照索引|第 7 章 Dockerfile / 第 11 章 Compose]]、[[Kubernetes官方文档|K8s 概念篇（了解定位）]]
- 下一讲：自动化的另一种手 → [[阶段5_Docker与自动化/03_Python运维脚本|Python运维脚本]]

---
*相邻：[[阶段5_Docker与自动化/01_Docker核心概念与命令|← 上一讲]] ｜ [[阶段5_Docker与自动化/03_Python运维脚本|下一讲 →]]*
