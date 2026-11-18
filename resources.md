---
类型: 资料总表
标签: [运维学习, 资料清单]
tags: [运维学习, 资料清单]
---

# 入门资料清单（零基础 → 初级运维）

> 整理日期：2026-11-07。原则：**免费优先、中文优先、一套主线资料 + 一本工具书**，不贪多。
> 使用方式见 [[2026-11-07_学习规划_零基础到初级运维]]；每份资料的详细卡片（链接/用法/双链）在 `资料卡片\` 子目录，从 [[学习主页]] 进入图谱最直观。

## 1. Linux 基础（阶段 1 主线）

| 资料 | 类型 | 说明 |
|---|---|---|
| [[韩顺平Linux教程|韩顺平《Linux 零基础入门》]] | B 站视频 | 讲解细致、节奏慢，最适合纯新手；B 站搜「韩顺平 Linux」 |
| [[黑马Linux教程|黑马《新版 Linux 零基础快速入门到精通》]] | B 站视频 | 播放量 800 万+，偏实战就业向；与韩顺平二选一跟完 |
| [[鸟哥的Linux私房菜]] | 书（有免费在线版） | 公认经典，当工具书查阅：http://linux.vbird.org |
| [[菜鸟教程]] | 在线速查 | 免费中文：https://www.runoob.com/linux/linux-command-manual.html |

## 2. 网络基础（阶段 2 主线）

| 资料 | 类型 | 说明 |
|---|---|---|
| [[图解TCPIP]] | 书 | 零基础入门首选，全图解讲分层模型 |
| [[小林coding]] | 免费在线 | 《图解网络》500 张图讲 HTTP/TCP/IP，**面试重点** |
| Wireshark | 工具 | 抓包实验用；Nmap 已在 `07_Tools\nmap` |

## 3. Shell 与 Python（阶段 3 / 5）

| 资料 | 类型 | 说明 |
|---|---|---|
| [[菜鸟教程|菜鸟教程 Shell 章节]] | 免费在线 | https://www.runoob.com/linux/linux-shell.html ，主线入门 |
| [[Linux命令行与Shell脚本编程大全]] | 书 | 系统进阶，选读常用章节 |
| [[explainshell]] | 在线工具 | https://explainshell.com ，粘贴命令逐段解释 |
| [[廖雪峰Python教程]] | 免费在线 | 阶段 5 运维脚本用 |

## 4. 服务实战（阶段 4）

| 资料 | 类型 | 说明 |
|---|---|---|
| Nginx 官方文档 | 官方 | https://nginx.org/en/docs/（配合中文博客食用） |
| MySQL 8.0 官方参考 | 官方 | https://dev.mysql.com/doc/ ；[[小林coding]]《图解 MySQL》补面试 |
| [[小林coding|小林coding《图解Redis》]] | 免费在线 | RDB/AOF 等高频考点 |
| [阿里云《运维工程师学习路线》](https://developer.aliyun.com/article/1150909) | 路线参考 | 对照查漏 |

## 5. Docker 与自动化（阶段 5 主线）

| 资料 | 类型 | 说明 |
|---|---|---|
| [[Docker从入门到实践]] | 免费在线书 | https://docker-practice.github.io/zh-cn ，主线教程 |
| [[Kubernetes官方文档]] | 官方 | https://kubernetes.io/zh-cn/ ，阶段 5 只看概念篇 |
| [[Ansible中文指南]] | 免费在线 | https://ansible-tran.readthedocs.io ，入门 playbook |

## 6. 求职面试（阶段 6）

| 资料                                                         | 类型   | 说明                                                            |
| ---------------------------------------------------------- | ---- | ------------------------------------------------------------- |
| [[牛客运维面经]]                                                 | 面经库  | https://www.nowcoder.com/discuss/experience?tagId=643 ，一手面经最多 |
| [运维常见 40 道面试题（知乎）](https://zhuanlan.zhihu.com/p/405769585) | 题库   | 基础概念题                                                         |
| [[小林coding|小林coding《图解系统》]]                               | 免费在线 | 进程/内存等 OS 八股                                                  |

## 7. 学习路线总参考（怎么串起来）

- [2025 大厂运维工程师终极指南（CSDN）](https://blog.csdn.net/hxsyyds49/article/details/147653281)
- [Linux 运维学习路线：零基础到精通（codefather）](https://www.codefather.cn/post/1640586295529324545)
- [运维工程师学习路线·详细版（阿里云开发者社区）](https://developer.aliyun.com/article/1150909)
- [GitHub: Tender-Liu/LinuxDevOps](https://github.com/Tender-Liu/LinuxDevOps)

## 8. 本地快照一览（2026-11-07 抓取并整理，约 1033 篇）

> 统一简体中文 + 中文标题 + 标准元数据；均已接入图谱。快照仅自用学习，版权归原作者，以在线版为准。

| 快照 | 篇数 | 位置 | 索引 |
|---|---|---|---|
| linux-command 命令速查（官方 11 分类 213 条 + 未归类 408 条） | 621 | `速查手册\01_linux命令速查\` | [[linux命令速查_索引]]（浏览：[[linux命令分类导航]]） |
| Docker 从入门到实践（全书含配图） | 202 | `官方文档\02_Docker从入门到实践\` | [[Docker实践_快照索引]] |
| 小林 coding 图解网络（网络篇） | 41 | `官方文档\01_小林coding\` | [[小林coding_快照索引]] |
| 小林 coding 图解 MySQL（MySQL 篇） | 21 | `官方文档\01_小林coding\` | [[小林coding_MySQL与Redis_快照索引]] |
| 小林 coding 图解 Redis（Redis 篇） | 15 | `官方文档\01_小林coding\` | [[小林coding_Redis_快照索引]] |
| 小林 coding 图解系统（系统篇） | 18 | `官方文档\01_小林coding\` | [[小林coding_系统篇_快照索引]] |
| 小林 coding 全系列导览（站点地图） | 1 | `官方文档\01_小林coding\` | （同上） |
| 鸟哥私房菜·基础篇 CentOS7（第零~二十四章 + 目录） | 26 | `官方文档\05_鸟哥的Linux私房菜\` | [[鸟哥私房菜_快照索引]] |
| 菜鸟教程 Linux/Shell 教程（4 主题子目录） | 21 | `速查手册\02_菜鸟教程Linux\` | [[菜鸟教程Linux_快照索引]] |
| Ansible 中文指南（01_核心教程 / 02_附录与杂项） | 14 | `官方文档\04_Ansible中文指南\` | [[Ansible指南_快照索引]] |
| Kubernetes 概念篇（01~09，含 kubectl 速查） | 9 | `官方文档\03_Kubernetes概念篇\` | [[K8s概念篇_快照索引]] |
| 廖雪峰 Python 教程（4 阶段子目录） | 14 | `官方文档\06_廖雪峰Python教程\` | [[廖雪峰Python_快照索引]] |

## 使用守则

1. **每个阶段只跟一套主线**，其他资料仅作查阅，不做"收藏家"；
2. 视频 1.25~1.5 倍速看概念，命令演示部分原速跟敲；
3. 所有实验结论写进 `02_Experiments\问题复盘\`，踩坑即资产；
4. 官方文档链接随用随存快照/摘录到 `05_References\02_官方文档\`；
5. 新增资料时先建资料卡片（`资料卡片\`），再回本清单登记一行，保持图谱连通。

---
*关联：[[2026-11-07_学习规划_零基础到初级运维]] ｜ [[学习主页]] ｜ [[00_README]]*
