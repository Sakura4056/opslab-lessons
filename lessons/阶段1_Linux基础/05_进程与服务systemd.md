---
类型: 学习讲义
阶段: 1
标签: [学习讲义, Linux, 进程, systemd]
---

# 进程与服务 systemd

> 阶段笔记：[[阶段1_Linux基础]] ｜ 上一讲：[[阶段1_Linux基础/04_软件包与环境变量|软件包与环境变量]] ｜ 下一讲：[[阶段1_Linux基础/06_文本三剑客与管道|文本三剑客与管道]]
> 排障第一现场：**CPU 飙了看 top，服务挂了看 systemctl，端口被占看 ss+lsof**。

## 学习目标

- 理解进程概念：PID、前台/后台、父子进程
- 熟练用 ps/top/kill 管理进程，会用 nohup/& 后台运行
- 掌握 systemctl 管理 systemd 服务的全套动作，会写最简单的 unit 文件

## 核心概念

| 概念 | 说明 |
|---|---|
| PID | 进程号，系统给每个进程的唯一编号 |
| 父/子进程 | 进程由另一进程启动形成父子树（PPID） |
| 信号 | 进程间通知机制；`kill -9`=SIGKILL 强杀，`kill -15`=SIGTERM 优雅终止 |
| 守护进程(daemon) | 后台长期运行的服务进程，如 nginx、sshd |
| systemd | 现代 Linux 的 1 号进程，统一管理服务（unit） |

**服务状态机**：`inactive → active(running) / failed`；开机自启由 `enable/disable` 控制。

## 命令实操

### 进程三板斧

```bash
ps aux                     # 快照式全量进程（BSD 风格）
ps aux | grep nginx        # 找特定进程
ps -ef --forest            # 树状显示父子关系
top                        # 交互式实时监控：P 按CPU排 M 按内存排 q 退出
kill -15 1234              # 优雅终止（先试这个）
kill -9 1234               # 强杀（15 无效再用）
pkill -f "python app.py"   # 按命令行匹配杀进程
jobs / fg / bg             # 前后台作业：Ctrl+Z 暂停后 bg 放后台
nohup python app.py &      # 退出终端也继续跑（输出到 nohup.out）
```

### systemd 服务管理（必背八条）

```bash
systemctl status nginx     # 看状态（active/failed、PID、日志片段）
sudo systemctl start nginx     # 启动
sudo systemctl stop nginx      # 停止
sudo systemctl restart nginx   # 重启（配置改完）
sudo systemctl reload nginx    # 重载配置（不断连接，nginx 支持）
sudo systemctl enable nginx    # 开机自启
sudo systemctl disable nginx   # 取消自启
systemctl list-units --type=service --state=running   # 列运行中的服务
journalctl -u nginx -n 50 --no-pager   # 看某服务最近 50 行日志
```

### 写一个自己的服务（面试加分）

```bash
sudo tee /etc/systemd/system/myapp.service > /dev/null <<'UNIT'
[Unit]
Description=My Test App
After=network.target

[Service]
ExecStart=/bin/bash -c 'while true; do echo running; sleep 10; done'
Restart=on-failure

[Install]
WantedBy=multi-user.target
UNIT
sudo systemctl daemon-reload      # 改过 unit 文件必须 reload
sudo systemctl enable --now myapp # 开机自启并立即启动
```

## 动手实验

- [ ] 用上面的 unit 文件创建 myapp 服务，status 验证 running；stop 后看状态变化
- [ ] `top` 里找出 CPU 占用最高的进程名与 PID，记录下来
- [ ] `nohup sleep 600 &` 后台运行，`ps -ef | grep sleep` 找到并 kill
- [ ] `journalctl -u myapp -n 20` 查看自己服务的日志

## 常见坑

- **kill -9 成瘾**：数据库/写入型进程先 -15，强杀可能丢数据；
- **改 unit 后忘 daemon-reload**：systemctl start 报错或行为不变；
- **enable 了但没 start**：enable 只管开机自启，当前状态不变（用 `enable --now`）；
- **WSL 里服务开机自启**：需 systemd=true（阶段 0 已配），且 WSL 启动才拉起。

## 自测

1. `kill -15` 与 `kill -9` 的区别？应该先用哪个？
2. status 输出里 active(running) 和 enabled 分别什么意思？
3. 写服务 unit 文件的三个区块名？改完要执行什么？
4. 怎么让脚本在退出 SSH 后继续运行？

## 延伸

- 快照：[[鸟哥私房菜_快照索引|鸟哥第十六章 进程管理、第十七章 系统服务]]、[[菜鸟教程Linux_快照索引]]
- 下一课：进程产生的文本怎么处理 → [[阶段1_Linux基础/06_文本三剑客与管道|文本三剑客与管道]]

---
*相邻：[[阶段1_Linux基础/04_软件包与环境变量|← 上一讲]] ｜ [[阶段1_Linux基础/06_文本三剑客与管道|下一讲 →]]*
