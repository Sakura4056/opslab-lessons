---
类型: 学习讲义
阶段: 2
标签: [学习讲义, 网络, HTTP, HTTPS]
---

# HTTP 与 HTTPS

> 阶段笔记：[[阶段2_网络基础]] ｜ 上一讲：[[阶段2_网络基础/03_DNS解析|DNS解析]] ｜ 下一讲：[[阶段2_网络基础/05_网络排错工具箱|网络排错工具箱]]
> 运维视角的 HTTP：**会看请求响应、懂状态码、知道 HTTPS 多了什么**。

## 学习目标

- 掌握 HTTP 请求/响应结构与方法、常见状态码
- 用 curl 的 -v 模式解剖一次完整请求
- 说清 HTTPS = TLS 握手 + 加密传输，证书为什么重要

## 核心概念

### 请求与响应

```
请求：  方法 + URL + 版本
        请求头（Host / User-Agent / Content-Type / Cookie...）
        空行
        请求体（POST 才有）

响应：  版本 + 状态码 + 短语
        响应头（Server / Content-Type / Set-Cookie / Cache-Control...）
        空行
        响应体（HTML/JSON...）
```

### 状态码速查（背熟这 12 个）

| 码 | 含义 | 运维联想 |
|---|---|---|
| 200 | 成功 | — |
| 301/302 | 永久/临时重定向 | 配置跳转 http→https 用 301 |
| 304 | 缓存有效 | Not Modified |
| 400 | 请求语法错 | 客户端参数问题 |
| 401/403 | 未认证/无权限 | 鉴权配置 |
| 404 | 资源不存在 | 路由/路径错误 |
| 413 | 请求体过大 | nginx client_max_body_size |
| 499 | 客户端提前断开（nginx 定义） | 后端太慢用户跑了 |
| 500 | 服务端错误 | 看 error.log |
| 502 | 网关收到无效响应 | **upstream 挂了** |
| 503 | 服务不可用 | 过载/维护/健康检查失败 |
| 504 | 网关超时 | upstream 处理超时 |

### HTTPS = HTTP + TLS

TLS 握手（简化）：TCP 连接 → ClientHello（支持的算法）→ ServerHello + **证书**（含公钥，CA 签发）→ 密钥协商 → 对称加密通信。证书校验三要素：**域名匹配、有效期、可信 CA 链**——浏览器「不安全」警告基本就是这三样之一。

## 命令实操

```bash
# curl 解剖模式（运维最常用）
curl -v http://baidu.com                 # -v 显示完整请求响应头
curl -sL -o /dev/null -w "%{http_code} %{url_effective}\n" http://baidu.com
#      -sL 跟随重定向  -w 自定义输出：状态码+最终URL

curl -I https://www.baidu.com            # 只看响应头（HEAD 请求）
curl -IL https://www.baidu.com           # 跟随重定向看每一跳的状态码
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"test"}' http://localhost:8000/api     # POST JSON
curl --resolve test.local:443:127.0.0.1 https://test.local   # 临时解析（免改 hosts）

# TLS/证书相关
curl -vI https://expired.badssl.com/     # 看证书校验失败的表现
openssl s_client -connect www.baidu.com:443 -brief < /dev/null 2>&1 | head -8
```

## 动手实验

- [ ] `curl -v http://baidu.com` 抄录：请求行、Host 头、响应状态与 Location 头
- [ ] `curl -IL https://baidu.com` 记录重定向链（每跳状态码）
- [ ] 用 `openssl s_client` 查 baidu.com 证书的有效期
- [ ] 本地起 `python3 -m http.server 8000`，用 curl 分别 GET 一个存在/不存在的文件，观察 200 与 404

## 常见坑

- **502 vs 504**：502=后端回了但回的是坏的（进程挂/崩溃），504=后端干脆没在超时内回——报障时先分清；
- **curl 加了 -s 就看不见错误**：排错时用 `-sS`（保留报错）；
- **https 页面里 http 资源**：混合内容被浏览器拦截；
- **证书续期忘记**：SSL 证书到期是最经典的「周日故障」——用脚本监控有效期。

## 自测

1. 502 和 504 的区别？分别指向什么根因？
2. HTTPS 握手里证书的作用是什么？校验哪三件事？
3. 写出「只输出状态码并跟随重定向」的 curl 命令。
4. 301 和 302 对 SEO/缓存的影响差异？

## 延伸

- 快照：[[小林coding_快照索引|《图解网络》HTTP/HTTPS/HTTP2/HTTP3 全套]]
- 下一讲：把所有工具串成一套排错方法论 → [[阶段2_网络基础/05_网络排错工具箱|网络排错工具箱]]

---
*相邻：[[阶段2_网络基础/03_DNS解析|← 上一讲]] ｜ [[阶段2_网络基础/05_网络排错工具箱|下一讲 →]]*
