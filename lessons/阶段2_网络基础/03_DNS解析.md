---
类型: 学习讲义
阶段: 2
标签: [学习讲义, 网络, DNS]
---

# DNS 解析

> 阶段笔记：[[阶段2_网络基础]] ｜ 上一讲：[[阶段2_网络基础/02_TCP连接与抓包|TCP连接与抓包]] ｜ 下一讲：[[阶段2_网络基础/04_HTTP与HTTPS|HTTP与HTTPS]]
> 「网站打不开」一半的案子是 DNS —— 本课让你能一条命令定位。

## 学习目标

- 讲清 DNS 递归解析全链路（浏览器缓存 → hosts → 本地 DNS → 根/顶级/权威）
- 会用 dig/nslookup 查询并读懂答案（A 记录、TTL、CNAME）
- 会改 hosts、会排查「解析不对」类故障

## 核心概念

### 解析链路

```
浏览器缓存 → OS 缓存 → /etc/hosts → 本地 DNS(运营商/223.5.5.5)
   → 根域名服务器(.) → 顶级域(.com) → 权威 DNS(域名商) → 返回 A 记录
```

### 常见记录类型

| 类型 | 作用 | 例 |
|---|---|---|
| A | 域名 → IPv4 | baidu.com → 110.242.68.66 |
| AAAA | 域名 → IPv6 | — |
| CNAME | 域名 → 另一个域名 | www.shop.com → shop.example.cdn.com（CDN 常见） |
| MX | 邮件路由 | — |
| NS | 该域名由哪台权威 DNS 解析 | — |
| PTR | IP → 域名（反向解析） | — |

**TTL**：记录缓存秒数。改解析后「全球生效慢」就是旧缓存按 TTL 过期。

## 命令实操

```bash
# dig（信息最全，推荐）
dig baidu.com                       # ANSWER 段：A 记录与 TTL
dig +short baidu.com                # 只要结果
dig www.jd.com                      # 观察 CNAME 链
dig @223.5.5.5 baidu.com            # 指定用阿里 DNS 查
dig baidu.com ANY +noall +answer    # 只看答案段

# nslookup（Windows 同款）
nslookup baidu.com
nslookup baidu.com 8.8.8.8

# 本机解析链路的关键文件
cat /etc/hosts                      # 本地优先级最高的静态映射
cat /etc/resolv.conf                # 本机使用的 DNS 服务器
resolvectl status | head -20        # systemd 管的 DNS 配置（WSL2）
```

**hosts 实战**：`echo "10.10.10.10 test.internal" | sudo tee -a /etc/hosts` —— 内网服务/本地开发劫持域名必会。

## 动手实验

- [ ] `dig +short baidu.com` 与 `dig +short www.jd.com`，记录后者 CNAME 链
- [ ] 用 hosts 把 `mydev.local` 指向 127.0.0.1，`ping mydev.local` 验证，然后还原
- [ ] `dig @223.5.5.5 baidu.com` 与默认 DNS 对比 TTL 是否一致
- [ ] 记录 /etc/resolv.conf 里的 DNS 服务器地址（WSL 默认指向 Windows）

## 常见坑

- **hosts 不生效**：浏览器有 DNS 缓存/DoH（浏览器内置加密 DNS 会绕过 hosts）；
- **改了域名解析立刻验证**：本机缓存没过期——`dig` 时加 @新 DNS 先验证权威答案；
- **WSL resolv.conf 被自动覆盖**：WSL 用 Windows 的 DNS；要固定需在 `/etc/wsl.conf` 关闭 `generateResolvConf`；
- **公司内网域名公网查不到**：那是内网权威 DNS，`dig @内网DNS 内网域名`。

## 自测

1. 画出完整 DNS 递归解析链路。
2. CNAME 和 A 记录的区别？CDN 为什么用 CNAME？
3. 「ping 域名解析到了旧 IP」按什么顺序排查？
4. /etc/hosts 的优先级在本地 DNS 之前还是之后？

## 延伸

- 快照：[[小林coding_快照索引|《图解网络》DNS 篇]]
- 下一讲：拿到 IP 就能连上 443 了 → [[阶段2_网络基础/04_HTTP与HTTPS|HTTP与HTTPS]]

---
*相邻：[[阶段2_网络基础/02_TCP连接与抓包|← 上一讲]] ｜ [[阶段2_网络基础/04_HTTP与HTTPS|下一讲 →]]*
