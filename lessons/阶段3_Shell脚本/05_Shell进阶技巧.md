---
类型: 学习讲义
阶段: 3
标签: [学习讲义, Shell, 进阶]
---

# Shell 进阶技巧

> 阶段笔记：[[阶段3_Shell脚本]] ｜ 上一讲：[[阶段3_Shell脚本/04_综合实战_备份与巡检脚本|备份与巡检实战]] ｜ 下一讲：[[阶段4_服务实战/01_Nginx安装与配置|Nginx安装与配置]]
> 生产级脚本与练手脚本的区别就在这五件事：信号处理、调试、参数规范、锁与幂等。

## 学习目标

- 会用 trap 做「退出清理」和「临时文件保障」
- 会用 `bash -x` / shellcheck 调试与静态检查
- 会用 getopts 规范参数、用锁文件防任务重叠
- 会用 select/管道陷阱等进阶技巧收尾

## 核心概念与实操

### 1. trap：信号处理（退出时自动清理）

```bash
#!/bin/bash
TMP=$(mktemp)
trap 'rm -f "$TMP"; echo "清理完成"' EXIT    # 无论正常/异常退出都执行
trap 'echo "收到中断，安全退出"; exit 130' INT TERM
sleep 100
```

### 2. 调试与静态检查

```bash
bash -x script.sh            # 逐行回显执行过程（变量展开可见）
bash -n script.sh            # 只做语法检查不执行
shellcheck script.sh         # ⭐ 静态检查神器（sudo apt install shellcheck）
                             #   能查出未引号变量、无效循环、永久循环等 90% 低级错误
```

### 3. getopts：规范参数（对比手工 $1$2）

```bash
#!/bin/bash
# 用法: deploy.sh -e prod -v 1.2 [-f]
while getopts "e:v:f" opt; do
  case $opt in
    e) ENV=$OPTARG ;;        # -e 后带值
    v) VER=$OPTARG ;;
    f) FORCE=1 ;;            # 开关型参数
    *) echo "用法: $0 -e 环境 -v 版本 [-f]"; exit 1 ;;
  esac
done
echo "环境=${ENV:-dev} 版本=${VER:-latest} 强制=${FORCE:-0}"
```

### 4. 锁文件：防止 cron 任务重叠执行

```bash
LOCK=/var/run/backup.lock
if [ -e "$LOCK" ] && kill -0 $(cat "$LOCK") 2>/dev/null; then
  echo "上一任务仍在运行，退出"; exit 1
fi
echo $$ > "$LOCK"             # 写入自己的 PID
trap 'rm -f "$LOCK"' EXIT     # 退出时释放
# ……任务主体……
```

### 5. 其他高频技巧

```bash
some_cmd | tee log.txt        # 输出同时进屏幕和文件
diff <(cmd1) <(cmd2)          # 进程替换：比较两条命令的输出
timeout 30 long_task.sh       # 超时强杀（防挂死）
xargs -P 4 -n 1 curl -s       # 并行 4 路（批量下载/检查提速）
```

## 动手实验

- [ ] 给阶段 3 的 backup.sh 加 trap 清理 + 锁文件；开两个终端同时跑验证互斥
- [ ] `shellcheck` 检查自己写过的所有脚本，清零所有 warning
- [ ] 把 backup.sh 的手工参数改成 getopts：`-s 源目录 -k 保留天数`
- [ ] 用 `xargs -P` 并行探测 10 个端口的连通性并计时对比串行

## 常见坑

- **trap 里引用未初始化变量**：定义 trap 前先定义变量；
- **锁文件残留**：进程被 kill -9 后锁未释放——用 `kill -0 PID` 判断进程真死了才跳过；
- **shellcheck 全绿≠正确**：它查不出业务逻辑错误，只兜底语法/惯用法；
- **xargs 并行打爆对端**：-P 适度（4~8），对外部服务加限速。

## 自测

1. trap '…' EXIT 在什么情况下会执行？（正常/出错/Ctrl+C 都算吗）
2. 锁文件方案为什么要检查 PID 存活而不是只判文件存在？
3. getopts 相比手工 `$1 $2` 的两个优势？
4. `bash -x` 输出里 `+` 开头的行代表什么？

## 延伸

- 讲义：[[阶段3_Shell脚本/04_综合实战_备份与巡检脚本|备份与巡检实战]]
- 快照：[[explainshell]]、[[菜鸟教程|菜鸟 Shell 教程]]

---
*相邻：[[阶段3_Shell脚本/04_综合实战_备份与巡检脚本|← 上一讲]] ｜ [[阶段4_服务实战/01_Nginx安装与配置|下一讲 →]]*
