---
类型: 学习讲义
阶段: 4
标签: [学习讲义, Nginx, HTTPS, 证书]
---

# Nginx HTTPS 与证书

> 阶段笔记：[[阶段4_服务实战]] ｜ 上一讲：[[阶段4_服务实战/05_运维方案_备份监控与安全|运维方案]] ｜ 下一讲：[[阶段5_Docker与自动化/01_Docker核心概念与命令|Docker核心概念与命令]]
> 运维高频任务：**给站点上 HTTPS**——自签用于内网/练习，Let's Encrypt 用于公网。

## 学习目标

- 用 openssl 自签证书配出可用 HTTPS 站点（本地练习）
- 理解证书链与浏览器校验失败的原因
- 掌握 HTTP→HTTPS 强制跳转与 HSTS；了解 Let's Encrypt/certbot 流程

## 核心概念

```
证书内容：域名(SAN) + 有效期 + 公钥 + 签发者(CA) 签名
信任链：  根CA → 中级CA → 服务器证书（浏览器内置根CA 列表）
自签证书：自己当 CA——浏览器不信任（测内网可用，公网用 Let's Encrypt）
```

| 方案 | 适用 | 有效期 |
|---|---|---|
| openssl 自签 | 内网/练习/测试环境 | 自定 |
| Let's Encrypt (certbot) | 公网域名（需 DNS 可控） | 90 天，自动续期 |

## 命令实操

### 1. 自签证书（一条命令生成 key+证书）

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/app.key \
  -out /etc/nginx/ssl/app.crt \
  -subj "/CN=app.local"
```

### 2. HTTPS 站点 + 强制跳转

```nginx
# /etc/nginx/conf.d/app.conf
server {                          # HTTP → HTTPS 跳转
    listen 80;
    server_name app.local;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl;
    server_name app.local;
    root /var/www/app;
    ssl_certificate     /etc/nginx/ssl/app.crt;
    ssl_certificate_key /etc/nginx/ssl/app.key;
    ssl_protocols       TLSv1.2 TLSv1.3;     # 禁用老旧协议
    ssl_ciphers         HIGH:!aNULL:!MD5;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
curl -k https://127.0.0.1/            # -k 忽略自签证书告警（仅测试用）
curl -sI http://127.0.0.1/ | head -3  # 应看到 301 → https
```

### 3. 证书检查（巡检脚本素材）

```bash
openssl x509 -in /etc/nginx/ssl/app.crt -noout -dates -subject
# 远程站点（含剩余天数）：
echo | openssl s_client -connect target.com:443 2>/dev/null \
  | openssl x509 -noout -enddate
```

### 4. Let's Encrypt（公网流程认知）

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com        # 自动改 nginx 配置 + 申请
sudo certbot renew --dry-run                  # 续期演练（certbot 自动挂了续期任务）
```

## 动手实验

- [ ] 自签证书 + HTTPS server 配置，`curl -k https://127.0.0.1/` 返回 200
- [ ] 不加 `-k` 看报错信息（自签不被信任，理解证书链校验）
- [ ] HTTP 访问验证 301 跳转到 HTTPS
- [ ] 用 openssl 查自己证书的 SAN/有效期；写一条「证书剩余天数」巡检命令入库

## 常见坑

- **证书域名不匹配**：浏览器报 NET::ERR_CERT_COMMON_NAME_INVALID——SAN 里必须有访问域名；
- **443 未放行**：ufw/安全组记得 allow 443；
- **私钥权限**：app.key 应 600 且属主 root，泄漏=可伪造你的站点；
- **certbot 申请失败**：80 端口需可达 + DNS 已指向本机（公网流程的前提）。

## 自测

1. 自签证书和 Let's Encrypt 证书的本质区别？
2. HTTPS server 块中三行 ssl 配置各是什么？
3. 如何用一条命令查远程站点证书的到期时间？
4. 为什么 80 端口要保留 301 跳转而不是直接关掉？

## 延伸

- 讲义：[[阶段4_服务实战/01_Nginx安装与配置|Nginx安装与配置]]、[[阶段2_网络基础/04_HTTP与HTTPS|HTTP与HTTPS]]
- 快照：[[小林coding_快照索引|《图解网络》HTTPS/TLS 握手篇]]

---
*相邻：[[阶段4_服务实战/05_运维方案_备份监控与安全|← 上一讲]] ｜ [[阶段5_Docker与自动化/01_Docker核心概念与命令|下一讲 →]]*
