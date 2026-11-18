---
类型: 学习讲义
阶段: 4
标签: [学习讲义, MySQL, 数据库]
---

# MySQL 安装与日常运维

> 阶段笔记：[[阶段4_服务实战]] ｜ 上一讲：[[阶段4_服务实战/01_Nginx安装与配置|Nginx安装与配置]] ｜ 下一讲：[[阶段4_服务实战/03_Redis入门与持久化|Redis入门与持久化]]
> 运维不写业务 SQL，但**装库、建账号、备份、看慢查询**全是活。

## 学习目标

- 安装 MySQL 8.x 并完成安全初始化
- 会建库/建用户/授权（最小权限原则）
- 会 mysqldump 备份与恢复，会开慢查询日志

## 核心概念

| 概念 | 说明 |
|---|---|
| 端口 | 默认 3306；my.cnf 可改 |
| root 密码 | 安装后初始化；生产用强密码+密钥管理 |
| 权限模型 | 用户@主机 粒度授权（`dev@'10.0.0.%'`） |
| my.cnf | /etc/mysql/mysql.conf.d/mysqld.cnf（Ubuntu） |
| 慢查询日志 | slow_query_log=ON，long_query_time 阈值 |

**权限最小权限原则**：应用账号只给需要的库+需要的操作（SELECT,INSERT,UPDATE,DELETE），绝不给 ALL。

## 命令实操

### 安装与初始化

```bash
sudo apt install -y mysql-server
sudo systemctl enable --now mysql
sudo mysql_secure_installation        # 交互式安全加固（删匿名用户、禁远程root）
mysql --version
sudo systemctl status mysql
```

### 建库建用户（最常用三板斧）

```sql
-- sudo mysql 进入（Ubuntu root 走 socket 免密）
CREATE DATABASE appdb DEFAULT CHARACTER SET utf8mb4;
CREATE USER 'app'@'10.0.0.%' IDENTIFIED BY '强密码Aa123!';
GRANT SELECT,INSERT,UPDATE,DELETE ON appdb.* TO 'app'@'10.0.0.%';
FLUSH PRIVILEGES;
SHOW GRANTS FOR 'app'@'10.0.0.%';     -- 复核权限
```

### 备份与恢复（运维核心职责）

```bash
# 备份整库
mysqldump -u root -p --single-transaction appdb > appdb_$(date +%F).sql
# --single-transaction：InnoDB 一致性快照，不锁表（生产必加）

# 恢复
mysql -u root -p appdb < appdb_2026-01-01.sql
# 验证：备份文件能 restore 才叫备份成功（同阶段 3 恢复演练原则）
```

### 慢查询日志

```bash
sudo tee /etc/mysql/mysql.conf.d/slow.cnf <<'CONF'
[mysqld]
slow_query_log = ON
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1
CONF
sudo systemctl restart mysql
# 之后用 awk/工具分析 slow.log，问"为什么慢"交给开发 + EXPLAIN
```

## 动手实验

- [ ] 安装 MySQL，完成 secure_installation，记录端口与数据目录（`/var/lib/mysql`）
- [ ] 建库 appdb + 用户 app@'localhost'，只授 CRUD；用 app 登录验证：能读、不能 DROP
- [ ] 建一张表插 3 行数据 → mysqldump 备份 → DROP DATABASE → 恢复 → 验证数据回来
- [ ] 开慢查询日志，执行 `SELECT SLEEP(2);`，在 slow.log 里找到这条记录

## 常见坑

- **root 远程开放**：secure_installation 里已禁；万不得已也用 SSH 隧道；
- **备份没含 --single-transaction**：锁全表，业务抖动；
- **字符集**：建库显式 utf8mb4，否则中文/emoji 乱码返工；
- **密码硬编码在脚本里**：用 `~/.my.cnf`（chmod 600）或环境变量传递；
- **磁盘满**：binlog 会吃磁盘，定期检查（阶段 5 的监控会覆盖）。

## 自测

1. 授权 `app@'10.0.0.%'` 里的主机部分是什么含义？
2. mysqldump 的 `--single-transaction` 解决什么问题？
3. 慢查询日志怎么开？定位到慢 SQL 后给开发什么信息？
4. 为什么建库要显式指定 utf8mb4？

## 延伸

- 快照：[[小林coding_快照索引|《图解MySQL》索引/事务/锁]]（面试重点）、[[2026-11-07_入门资料清单|MySQL 官方文档]]
- 下一讲：缓存层 → [[阶段4_服务实战/03_Redis入门与持久化|Redis入门与持久化]]

---
*相邻：[[阶段4_服务实战/01_Nginx安装与配置|← 上一讲]] ｜ [[阶段4_服务实战/03_Redis入门与持久化|下一讲 →]]*
