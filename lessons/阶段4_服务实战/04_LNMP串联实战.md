---
类型: 学习讲义
阶段: 4
标签: [学习讲义, 实战, LNMP]
---

# LNMP 串联实战

> 阶段笔记：[[阶段4_服务实战]] ｜ 上一讲：[[阶段4_服务实战/03_Redis入门与持久化|Redis入门与持久化]] ｜ 下一讲：[[阶段4_服务实战/05_运维方案_备份监控与安全|运维方案]]
> **简历项目①的主体**：把 Nginx + 应用 + MySQL + Redis 串成一个真实可访问的服务。

## 学习目标

- 完整跑通「Nginx → Flask 应用 → MySQL/Redis」链路
- 用 systemd 管理应用进程（生产形态）
- 形成可复现的部署文档（这是项目①的交付物核心）

## 架构图

```
浏览器 ──→ Nginx :80 ──┬── /          → 静态页
                       └── /api/*     → Flask :8000 ──→ MySQL :3306
                                                    └─→ Redis :6379
```

## 命令实操

### 1. 准备应用（Flask + Redis + MySQL）

```bash
sudo apt install -y python3-pip python3-venv
python3 -m venv /opt/app/venv
/opt/app/venv/bin/pip install flask redis mysql-connector-python

mkdir -p /opt/app && vim /opt/app/app.py
```

```python
# /opt/app/app.py — 最小可用应用：计数器走 Redis，数据落 MySQL
from flask import Flask, jsonify
import redis, mysql.connector

app = Flask(__name__)
r = redis.Redis(host="127.0.0.1", port=6379, decode_responses=True)
db = mysql.connector.connect(host="127.0.0.1", user="app", password="密码",
                             database="appdb")

@app.route("/")
def index():
    return "<h1>LNMP 实验站</h1><a href='/api/visit'>/api/visit</a>"

@app.route("/api/visit")
def visit():
    visits = r.incr("visits")            # Redis 计数
    cur = db.cursor()
    cur.execute("INSERT INTO visits(ts) VALUES (NOW())")
    db.commit()
    return jsonify({"visits": visits, "stored": True})
```

### 2. systemd 托管应用（生产形态）

```bash
sudo tee /etc/systemd/system/flaskapp.service > /dev/null <<'UNIT'
[Unit]
Description=Flask LNMP Demo
After=network.target mysql.service redis-server.service

[Service]
User=www-data
WorkingDirectory=/opt/app
ExecStart=/opt/app/venv/bin/gunicorn -w 2 -b 127.0.0.1:8000 app:app
Restart=on-failure

[Install]
WantedBy=multi-user.target
UNIT
/opt/app/venv/bin/pip install gunicorn   # 生产 WSGI 服务器（替代内置 dev server）
sudo systemctl daemon-reload
sudo systemctl enable --now flaskapp
```

### 3. Nginx 反代与验证

```nginx
# /etc/nginx/conf.d/app.conf 的 location /api/ 指向 127.0.0.1:8000（阶段 4 第 1 讲）
```

```bash
curl http://127.0.0.1/api/visit     # 反复调用，visits 递增
redis-cli GET visits                # 验证 Redis 计数
```

## 动手实验（项目①主任务）

- [ ] 按 1~3 全步骤搭通，`curl /api/visit` 成功且数据双写（Redis+MySQL）
- [ ] 建表：`CREATE TABLE visits(id INT AUTO_INCREMENT PRIMARY KEY, ts DATETIME);`
- [ ] 压测雏形：`for i in $(seq 1 50); do curl -s http://127.0.0.1/api/visit > /dev/null; done` 后核对计数
- [ ] 断链实验：`systemctl stop redis-server` 再 curl，观察 500 → 记录 error.log 表现 → 恢复
- [ ] **写部署文档** `06_Projects\lnmp\部署文档.md`：架构图、每步命令、验证方法、故障记录

## 常见坑

- **Flask 自带 server 别用于生产**：单线程调试用，生产用 gunicorn（如上）；
- **权限混乱**：应用文件 owner 给 www-data，别全 777；
- **连接信息硬编码**：用环境变量/systemd Environment= 传递；
- **改了代码不重启**：gunicorn 不会热加载，`systemctl restart flaskapp`。

## 自测

1. 这套链路里每一跳的作用与端口？
2. 为什么用 gunicorn 替代 flask 内置服务器？
3. Redis 挂了应用什么表现？你会怎么排（提示：阶段 2 排错法 + error.log）？

## 延伸

- 项目产出目录：`06_Projects\lnmp\`（阶段 4 结束时打 git tag）
- 下一讲：给这套系统配齐「备份/监控/安全」→ [[阶段4_服务实战/05_运维方案_备份监控与安全|运维方案]]

---
*相邻：[[阶段4_服务实战/03_Redis入门与持久化|← 上一讲]] ｜ [[阶段4_服务实战/05_运维方案_备份监控与安全|下一讲 →]]*
