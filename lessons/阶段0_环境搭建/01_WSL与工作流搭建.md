---
类型: 学习讲义
阶段: 0
标签: [学习讲义, WSL, Git, 环境]
---

# WSL 与工作流搭建

> 阶段笔记：[[阶段0_环境搭建]] ｜ 上一讲：（起点）｜ 下一讲：[[阶段1_Linux基础/01_文件与目录操作|文件与目录操作]]

## 学习目标

- 理解 WSL2 是什么、和虚拟机/双系统的区别
- 完成 Ubuntu 24.04 的初始化与 systemd 开启
- 建立「Windows 编辑 + Linux 执行」的工作流
- 会用 Git 做最基本的版本管理

## 核心概念

| 概念 | 一句话解释 |
|---|---|
| WSL2 | Windows 里的轻量 Linux 虚拟机，文件互通、秒级启动 |
| 发行版 | Linux 的「发行版本」，我们用 Ubuntu 24.04 LTS（长期支持版） |
| systemd | Linux 的服务管理器，WSL2 默认关闭，要手动打开（后面 systemctl 依赖它） |
| Git | 版本管理工具，记录每次改动，可随时回退 |
| 仓库（repository） | 被 Git 管理的一个目录 |

## 命令实操

### 1. 确认 WSL2 与 Ubuntu 版本

```powershell
# Windows PowerShell 中执行
wsl --status          # 确认默认版本为 2
wsl -l -v             # 列出发行版与版本
```

### 2. Ubuntu 初始化（在 WSL 终端里）

```bash
sudo apt update && sudo apt upgrade -y   # 更新软件源与已装包
sudo apt install -y git vim curl wget    # 装基础工具
```

### 3. 开启 systemd（让 systemctl 可用）

```bash
# 编辑 /etc/wsl.conf，加入以下内容
[sudo] tee /etc/wsl.conf > /dev/null <<'CONF'
[boot]
systemd=true
CONF
```

然后在 Windows 侧执行 `wsl --shutdown`，重新进入 WSL 生效。验证：`systemctl status sshd 2>&1 | head -3` 能看到服务状态即成功。

### 4. Git 最小配置与首次提交

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
cd /mnt/d/OpsLab        # WSL 里 Windows 的 D 盘挂在 /mnt/d
git init                # 初始化仓库（.gitignore 已就位）
git add . && git commit -m "init: 工作区初始化"
```

## 动手实验

- [ ] `wsl -l -v` 截图/记录版本号到 `02_Experiments\操作日志\`
- [ ] 开启 systemd 并用 `systemctl` 验证
- [ ] 完成 git init + 首次提交；再改一个文件，用 `git diff` 看变化并提交第二次
- [ ] 在 VS Code 里 `code .` 打开 `D:\OpsLab`，确认能编辑 WSL/Windows 文件

## 常见坑

- **apt 前忘 update**：报 `Unable to locate package`——先 `apt update` 再装；
- **systemd 没重启就验证**：`wsl.conf` 改完必须 `wsl --shutdown`；
- **git 身份没配**：首次 commit 会报 `Please tell me who you are`；
- **在 /mnt/d 下跑 apt 等命令**：没有意义，Linux 软件装在 WSL 文件系统内。

## 自测

1. WSL2 和传统虚拟机的最大区别是什么？
2. 为什么 systemctl 需要先开启 systemd？怎么开？
3. `git add` 和 `git commit` 分别做了什么？
4. Windows 的 D 盘在 WSL 里的路径是什么？

## 延伸

- 快照：[[Docker实践_快照索引|Docker 书 03_install 章（以后装 Docker 用）]]
- 工作区规则：[[00_README]]

---
*相邻：[[阶段6_求职冲刺/03_简历与面试指南|（课程起点）]] ｜ 下一讲：[[阶段1_Linux基础/01_文件与目录操作|文件与目录操作]]*
