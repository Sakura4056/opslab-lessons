---
类型: 学习讲义
阶段: 5
标签: [学习讲义, Ansible, 自动化]
---

# Ansible 入门

> 阶段笔记：[[阶段5_Docker与自动化]] ｜ 上一讲：[[阶段5_Docker与自动化/03_Python运维脚本|Python运维脚本]] ｜ 下一讲：[[阶段5_Docker与自动化/05_监控与CI_CD概念|监控与CI/CD概念]]
> 一台机器用脚本，**一批机器用 Ansible**——无代理、SSH 直连、幂等执行。

## 学习目标

- 理解 Ansible 三要素：inventory / ad-hoc / playbook
- 理解「幂等」为什么是配置管理的灵魂
- 写一个「批量装 Nginx 并启动」的 playbook

## 核心概念

| 概念 | 说明 |
|---|---|
| inventory | 主机清单（分组），默认 /etc/ansible/hosts |
| ad-hoc | 单条命令批量执行：`ansible all -m ping` |
| playbook | YAML 编排的任务集（生产形态） |
| 幂等 | 跑一遍和跑十遍结果一致（模块自动判断状态） |
| 模块 | 内置能力单元：apt/copy/service/template... |

## 命令实操

### 环境与 inventory（本机+WSL 双节点演练）

```bash
sudo apt install -y ansible
mkdir -p ~/ansible && vim ~/ansible/hosts
```

```ini
# ~/ansible/hosts — inventory
[webservers]
local ansible_connection=local        # 本机
wsl   ansible_host=127.0.0.1 ansible_port=2222 ansible_user=ops
# 前提：本机已 sshd 且 ops 能密钥登录（阶段 4 SSH 加固的成果）
```

### ad-hoc 快速验证

```bash
cd ~/ansible
ansible webservers -i hosts -m ping            # 连通性
ansible webservers -i hosts -m command -a "uptime"
ansible webservers -i hosts -m apt -a "name=nginx state=present" -b   # -b 提权
```

### playbook：批量装 Nginx

```yaml
# ~/ansible/nginx.yml
- hosts: webservers
  become: yes
  tasks:
    - name: 安装 nginx
      apt: { name: nginx, state: present, update_cache: yes }
    - name: 部署站点配置
      copy:
        content: |
          server { listen 8080; root /var/www/html; }
        dest: /etc/nginx/conf.d/ansible-site.conf
      notify: reload nginx
    - name: 确保服务运行
      service: { name: nginx, state: started, enabled: yes }
  handlers:
    - name: reload nginx
      service: { name: nginx, state: reloaded }
```

```bash
ansible-playbook -i hosts nginx.yml      # 再跑一遍：全部 ok(幂等) 而非 changed
curl 127.0.0.1:8080                      # 验证站点
```

## 动手实验

- [ ] inventory 配两节点，`-m ping` 全绿
- [ ] 跑 nginx.yml，第二次重跑观察「幂等」（changed → ok）
- [ ] 故意改坏 copy 内容再跑，体会 notify + handler 的按需触发
- [ ] playbook 与脚本入库 `03_Scripts\ansible\`

## 常见坑

- **inventory 用户/密钥不对**：连不通先 `ansible -m ping` 排 SSH；
- **忘了 become**：apt/service 类任务失败，报权限错误；
- **YAML 缩进**：两个空格，别用 Tab；
- **把 Ansible 当脚本跑**：它管「状态」不是「步骤」——写期望状态，模块自己算差异。

## 自测

1. inventory / ad-hoc / playbook 各是什么？
2. 幂等性举例说明？为什么重要？
3. handler 什么时候执行？
4. `-b` 和 `become: yes` 分别用在哪？

## 延伸

- 快照：[[Ansible中文指南|Ansible 中文指南（playbook 细节）]]
- 下一讲：监控与 CI/CD 概念（2026 岗位硬要求）→ [[阶段5_Docker与自动化/05_监控与CI_CD概念|监控与CI/CD概念]]

---
*相邻：[[阶段5_Docker与自动化/03_Python运维脚本|← 上一讲]] ｜ [[阶段5_Docker与自动化/05_监控与CI_CD概念|下一讲 →]]*
