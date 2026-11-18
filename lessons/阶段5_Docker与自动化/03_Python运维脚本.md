---
类型: 学习讲义
阶段: 5
标签: [学习讲义, Python, 自动化]
---

# Python 运维脚本

> 阶段笔记：[[阶段5_Docker与自动化]] ｜ 上一讲：[[阶段5_Docker与自动化/02_Dockerfile与Compose|Dockerfile与Compose]] ｜ 下一讲：[[阶段5_Docker与自动化/04_Ansible入门|Ansible入门]]
> 定位：Shell 之外的「重火力」——批量操作、API 调用、复杂逻辑。

## 学习目标

- 会用 requests 调 API 做健康检查，用 subprocess 调本地命令
- 掌握 paramiko 远程批量执行（跳板机/批量巡检场景）
- 建立「脚本有异常处理、有日志、可传参」的工程习惯

## 核心概念

| 库 | 场景 | 要点 |
|---|---|---|
| requests | HTTP API（健康检查/调云接口） | Session 复用、timeout 必设 |
| subprocess | 调本地命令（复用 shell 工具） | `capture_output=True, text=True` |
| paramiko | SSH 远程执行/传文件 | 密钥优先于密码 |
| 自建 | 日志 + 异常 + 参数 | try/except + logging |

**工具选型**：胶水/循环/解析用 Shell；多主机、API、复杂判断用 Python。

## 命令实操

### 健康检查脚本（requests）

```python
#!/usr/bin/env python3
# health_check.py — 批量健康检查并汇总
# 用法: python3 health_check.py
import requests, sys

TARGETS = [
    "http://127.0.0.1/",
    "http://127.0.0.1/api/visit",
    "https://www.baidu.com",
]

def check(url):
    try:
        r = requests.get(url, timeout=5)
        return r.status_code, r.elapsed.total_seconds()
    except requests.RequestException as e:
        return None, str(e)[:60]

fails = 0
for url in TARGETS:
    code, extra = check(url)
    ok = code == 200
    fails += not ok
    print(f"[{'OK ' if ok else 'FAIL'}] {code or 'ERR':>5} {extra or ''} {url}")
sys.exit(1 if fails else 0)        # 退出码供监控/cron 判断
```

### 本地命令与远程执行

```python
# subprocess：复用你已会的 shell 工具
import subprocess
r = subprocess.run(["df", "-h"], capture_output=True, text=True)
print(r.stdout.splitlines()[1])       # 解析输出

# paramiko：批量执行（pip install paramiko）
import paramiko
HOSTS = [("10.0.0.11", "ops"), ("10.0.0.12", "ops")]
for host, user in HOSTS:
    c = paramiko.SSHClient()
    c.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    c.connect(host, username=user, key_filename="~/.ssh/id_rsa")
    _, out, err = c.exec_command("uptime && df -h /")
    print(f"== {host} ==\n{out.read().decode()}")
    c.close()
```

## 动手实验

- [ ] 跑 health_check.py，把其中一个目标改成坏地址观察 FAIL 与退出码
- [ ] 写 `disk_report.py`：用 subprocess 采集 `df -h`，输出 >80% 的挂载点（结合阶段 1 的 awk 思路）
- [ ] 用 paramiko 对 WSL 本机 sshd 执行 `uptime`（先在本机装并启动 sshd）
- [ ] 把三个脚本入库 `03_Scripts\python\`，头部注释按规范补全

## 常见坑

- **requests 没 timeout**：卡死线程——永远 `timeout=5`；
- **异常裸奔**：网络操作必须 try/except，脚本崩了比慢更糟；
- **subprocess 用 shell=True 拼字符串**：注入风险+难维护，用列表形式；
- **paramiko 密码硬编码**：用密钥 + 配置文件分离。

## 自测

1. Shell 和 Python 各适合什么场景？两个例子。
2. requests 为什么必须设 timeout？
3. paramiko 用什么认证方式最安全？
4. 监控系统如何利用脚本的退出码？

## 延伸

- 快照：[[廖雪峰Python教程|廖雪峰 Python 基础（语法回顾）]]
- 下一讲：规模化配置管理 → [[阶段5_Docker与自动化/04_Ansible入门|Ansible入门]]

---
*相邻：[[阶段5_Docker与自动化/02_Dockerfile与Compose|← 上一讲]] ｜ [[阶段5_Docker与自动化/04_Ansible入门|下一讲 →]]*
