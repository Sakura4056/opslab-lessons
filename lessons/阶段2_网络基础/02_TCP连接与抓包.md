---
类型: 学习讲义
阶段: 2
标签: [学习讲义, 网络, TCP, tcpdump]
---

# TCP 连接与抓包

> 阶段笔记：[[阶段2_网络基础]] ｜ 上一讲：[[阶段2_网络基础/01_IP地址与路由|IP地址与路由]] ｜ 下一讲：[[阶段2_网络基础/03_DNS解析|DNS解析]]
> 面试之王：**三次握手 / 四次挥手 / TIME_WAIT** —— 本课用抓包把它变成「亲眼见过」。

## 学习目标

- 讲清三次握手、四次挥手的全过程与每一步的标志位
- 会用 tcpdump 抓一次完整的 TCP 连接
- 会看 ss 里的连接状态（LISTEN/ESTAB/TIME_WAIT）

## 核心概念

### 三次握手（建立连接）

```
客户端                    服务端
  │ —— SYN (seq=x) ——→     │  ① 我要连你
  │ ←— SYN+ACK (y, x+1) —  │  ② 收到，你也确认下
  │ —— ACK (y+1) ——→       │  ③ 确认，连接建立
```

**为什么三次**：双方都要确认「我能发、你能收、我还能收」。两次无法防止历史重复连接。

### 四次挥手（断开连接）

```
  │ —— FIN ——→             │  ① 我发完了
  │ ←— ACK —               │  ② 知道了（服务端可能还有数据要发）
  │ ←— FIN —               │  ③ 我也发完了
  │ —— ACK ——→             │  ④ 确认；客户端进入 TIME_WAIT 等 2MSL
```

**TIME_WAIT**：主动关闭方停留 ~60 秒，确保最后的 ACK 丢失可重传。高并发服务大量 TIME_WAIT 是常见现象（先优化长连接，别急着 tcp_tw_recycle）。

| ss 状态 | 含义 |
|---|---|
| LISTEN | 服务端监听中 |
| ESTAB | 连接已建立 |
| TIME_WAIT | 主动关闭方的收尾等待 |
| CLOSE_WAIT | 被动关闭方没调 close()（通常是代码 bug） |

## 命令实操

```bash
# 看连接状态
ss -tan                      # 所有 TCP 连接（含状态列）
ss -tan | awk '{print $1}' | sort | uniq -c    # 各状态数量统计
ss -tlnp | grep 22           # 谁在监听 22

# tcpdump 抓包（WSL 需 sudo；-i eth0 为 WSL 网卡）
sudo tcpdump -i eth0 tcp port 80 -nn -c 20
#      -nn 不解析域名/端口名  -c 抓 20 个包就停

# 终端 A 抓包，终端 B 发起连接（亲眼看三次握手）
sudo tcpdump -i eth0 host baidu.com and tcp port 80 -nn
curl -s http://baidu.com > /dev/null
# 观察：Flags [S] → [S.] → [.]，结束 [F.] → [.] → [F.] → [.]
```

tcpdump 输出关键列：`时间戳 IP源.端口 > IP目的.端口: Flags [标志位], seq, ack, win, length`。

## 动手实验

- [ ] 双终端复现「抓包 + curl」，把 SYN/SYN.ACK/ACK 三行抄进实验日志并标注握手阶段
- [ ] `ss -tan | awk '{print $1}' | sort | uniq -c` 记录本机各状态连接数
- [ ] 抓包观察：`curl` 一个 https 站点（如 bing.com）与 http 站点的差异（443 看到的是密文）
- [ ] 思考题演练：为什么服务端关闭时少见 TIME_WAIT 而客户端多？

## 常见坑

- **tcpdump 忘记 -nn**：做 DNS 反解拖慢输出还刷屏；
- **抓错网卡**：WSL 里抓 `any` 或 `eth0`，抓 lo 只有本机回环；
- **CLOSE_WAIT 堆积**：不是网络问题，是**程序没关闭连接**——报障要指给开发；
- **半连接队列溢出**：SYN 洪水场景看 `netstat -s | grep -i listen` 的溢出计数。

## 自测

1. 画三次握手并标注 SYN/ACK 与 seq/ack 变化。
2. TIME_WAIT 为什么存在？过量的第一优化方向？
3. CLOSE_WAIT 大量堆积说明什么？
4. 写出「抓 eth0 上与 8.8.8.8 的 53 端口通信」的 tcpdump 命令。

## 延伸

- 快照：[[小林coding_快照索引|《图解网络》TCP 三次握手/四次挥手/大量 TIME_WAIT 全套]]（面试重点）
- 下一讲：连接建立前先要「把域名变成 IP」→ [[阶段2_网络基础/03_DNS解析|DNS解析]]

---
*相邻：[[阶段2_网络基础/01_IP地址与路由|← 上一讲]] ｜ [[阶段2_网络基础/03_DNS解析|下一讲 →]]*
