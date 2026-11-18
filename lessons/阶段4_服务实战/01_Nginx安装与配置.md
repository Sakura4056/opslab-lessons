---
类型: 学习讲义
阶段: 4
标签: [学习讲义, Nginx, 服务实战]
---

# Nginx 安装与配置

> 阶段笔记：[[阶段4_服务实战]] ｜ 上一讲：[[阶段3_Shell脚本/04_综合实战_备份与巡检脚本|阶段3综合实战]] ｜ 下一讲：[[阶段4_服务实战/02_MySQL安装与日常运维|MySQL安装与日常运维]]
> Nginx 是运维简历出现频率最高的软件，没有之一。

## 学习目标

- 安装 Nginx 并理解配置文件的组织结构
- 掌握 server/location 两级结构，配出静态站点
- 会配反向代理与日志格式，看懂 access/error 日志

## 核心概念

### 配置结构（主→站→路径）

```
/etc/nginx/
├── nginx.conf                 # 主配置：全局参数 + include 各站点
└── conf.d/*.conf              # 站点配置放这里（http{} 级）
    └── sites-available/ sites-enabled/  （Debian 风格，软链启用）
```

```nginx
server {                              # 一个 server = 一个虚拟主机（靠 server_name 区分）
    listen 80;
    server_name app.local;            # 域名
    root /var/www/app;                # 静态根目录
    index index.html;

    location /api/ {                  # 前缀匹配：转给后端
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location ~* \.(jpg|png|css|js)$ { # 正则匹配：静态资源缓存
        expires 7d;
    }
}
```

**请求匹配顺序**：`server_name` 选虚拟主机 → location 按「精确 → 前缀(最长) → 正则(顺序)」。

## 命令实操

```bash
sudo apt install -y nginx
nginx -t                              # ⭐ 每次改配置后必跑：语法检查
sudo systemctl reload nginx           # 平滑加载（不断连接）

# 静态站点
sudo mkdir -p /var/www/app && echo '<h1>hello nginx</h1>' | sudo tee /var/www/app/index.html
sudo vim /etc/nginx/conf.d/app.conf   # 写入上面的 server 配置
curl -H "Host: app.local" http://127.0.0.1/     # 用 Host 头模拟域名访问

# 反向代理验证（后端用 python 模拟）
python3 -m http.server 8000 &         # 简易后端
curl http://127.0.0.1/api/            # 经 nginx → 后端

# 日志
sudo tail -f /var/log/nginx/access.log    # 看访问：IP 状态码 路径 UA
sudo tail -f /var/log/nginx/error.log     # 看错误：配置错/后端不通都在这
```

## 动手实验

- [ ] 安装 nginx，`curl 127.0.0.1` 看默认页
- [ ] 建静态站点：conf.d/app.conf + index.html，`nginx -t && reload`，curl 验证 200
- [ ] 配 `/api/` 反代到 python http.server，curl 验证返回内容来自后端
- [ ] 故意写错配置（如缺分号）体验 `nginx -t` 报错与 reload 拒绝
- [ ] access.log 里找到自己的请求，说出每一列含义

## 常见坑

- **403 Forbidden**：root 目录权限（nginx worker 无 x 权限）或 index 文件缺失；
- **改配置只 reload 不 -t**：语法错时 reload 拒绝加载，你以为生效了其实没有；
- **proxy_pass 末尾斜杠**：`/api/` 与 `http://backend/api`、`http://backend` 拼接行为不同——改完用 curl 实测；
- **server_name 冲突**：同端口同 server_name 只生效第一个；用 `nginx -T`（大写）导出全部生效配置查重。

## 自测

1. `nginx -t` 和 `systemctl reload` 分别做什么？
2. 写出「把 /api/ 转发到 127.0.0.1:8000 并保留真实 IP」的 location 配置。
3. 403 排查的两个方向？
4. 502 出现时先看哪个日志？

## 延伸

- 快照：[[2026-11-07_入门资料清单|Nginx 官方文档]]、[[小林coding_快照索引|HTTP 相关]]
- 下一讲：数据层 → [[阶段4_服务实战/02_MySQL安装与日常运维|MySQL安装与日常运维]]

---
*相邻：[[阶段3_Shell脚本/04_综合实战_备份与巡检脚本|← 上一讲]] ｜ [[阶段4_服务实战/02_MySQL安装与日常运维|下一讲 →]]*
