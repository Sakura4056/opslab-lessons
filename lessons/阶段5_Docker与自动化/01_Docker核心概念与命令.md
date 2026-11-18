---
类型: 学习讲义
阶段: 5
标签: [学习讲义, Docker, 容器]
---

# Docker 核心概念与命令

> 阶段笔记：[[阶段5_Docker与自动化]] ｜ 上一讲：[[阶段4_服务实战/05_运维方案_备份监控与安全|阶段4运维方案]] ｜ 下一讲：[[阶段5_Docker与自动化/02_Dockerfile与Compose|Dockerfile与Compose]]
> 环境基准：Docker Desktop（WSL2 后端）或 WSL 内原生 Engine；命令一律 **Compose v2 风格**。

## 学习目标

- 说清镜像/容器/仓库三者关系与「容器只是进程」的本质
- 熟练 15 个高频命令覆盖 90% 日常操作
- 掌握数据卷与端口映射，理解容器为什么「默认易失」

## 核心概念

```
镜像(image)：只读模板（分层存储，像类）
容器(container)：镜像的运行实例（像对象；停止 ≠ 删除）
仓库(registry)：存镜像的地方（Docker Hub / 私有 registry）
```

**容器本质** = 被 namespace 隔离、被 cgroup 限制资源的一个**进程**。所以：没有 `-d` 它就是前台进程；删容器数据就没了——持久数据必须挂**卷（volume）**。

## 命令实操

```bash
docker version && docker info        # 环境确认
docker search nginx | head -5        # 搜镜像

# 镜像生命周期
docker pull nginx:1.27-alpine        # 拉取（尽量固定版本+alpine 瘦身）
docker images                        # 本地镜像列表
docker rmi nginx:1.27-alpine         # 删镜像

# 容器生命周期
docker run -d --name web -p 8080:80 nginx:1.27-alpine
#      -d 后台  --name 命名  -p 宿主端口:容器端口
docker ps                            # 运行中容器；-a 含已停止
docker logs -f --tail 20 web         # ⭐ 看容器日志（最常用）
docker exec -it web sh               # 进入容器（alpine 用 sh 不是 bash）
docker stop web && docker start web
docker rm web                        # 删容器（先 stop）；-f 强删
docker inspect web | grep -A5 Mounts # 查容器详情/挂载
docker stats                         # 容器资源占用实时视图

# 数据卷（持久化的关键）
docker volume create appdata
docker run -d --name db -v appdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=x mysql:8
docker run -d --name web2 -p 8081:80 -v "$PWD/site:/usr/share/nginx/html" nginx:1.27-alpine
#      -v 卷名:容器内路径      绑定挂载：宿主目录:容器内路径（开发常用）

# 清理（磁盘管理日常）
docker system df                     # 磁盘占用总览
docker system prune -a               # 清理无用容器/镜像/网络（⚠ 确认后再执行）
```

## 动手实验

- [ ] 拉 nginx 并映射 8080，浏览器访问 `http://localhost:8080`；`logs` 验证访问记录
- [ ] `exec` 进容器查看 `/usr/share/nginx/html`，改掉 index.html（容器内改动，删容器即失）
- [ ] 绑定挂载实验：宿主目录放 index.html，容器内立即生效
- [ ] 卷实验：mysql 容器写数据 → 删容器 → 新容器挂同一卷 → 数据还在
- [ ] `docker system df` 记录清理前后对比

## 常见坑

- **数据没挂卷**：容器一删数据全丢——数据库类容器必须挂卷；
- **端口映射方向**：`-p 8080:80` 是「宿主:容器」，别写反；
- **容器内无 systemd**：容器里跑的是单进程，systemd 那套不适用（一容器一服务）；
- **镜像 tag 用 latest**：今天和明天的 latest 可能不同——生产固定版本号；
- **alpine 无 bash**：进容器用 `sh`。

## 自测

1. 镜像、容器、仓库的关系？容器停止和删除的区别？
2. 为什么数据库容器必须挂卷？两种挂载方式的适用场景？
3. 进入一个 alpine 容器用什么命令？
4. 磁盘被 Docker 吃满，第一条清理命令是什么？

## 延伸

- 快照：[[Docker实践_快照索引|全书第 4~9 章与 11 章 Compose]]（深入阅读）
- 下一讲：把 LNMP 容器化 → [[阶段5_Docker与自动化/02_Dockerfile与Compose|Dockerfile与Compose]]

---
*相邻：[[阶段4_服务实战/05_运维方案_备份监控与安全|← 上一讲]] ｜ [[阶段5_Docker与自动化/02_Dockerfile与Compose|下一讲 →]]*
