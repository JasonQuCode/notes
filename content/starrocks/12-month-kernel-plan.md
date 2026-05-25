---
title: StarRocks 内核工程师 12 个月成长计划
tags:
  - starrocks
  - 学习计划
  - 数据库内核
date: 2026-05-25
---

# StarRocks 内核工程师 12 个月成长计划

> 制定日期:2026-05-25
> 适用对象:有一定 SQL / 数仓基础、本地有 SR Docker dev 环境、想成为 StarRocks 内核贡献者的工程师
> 总周期:12 个月(达到"独立修 bug + 完成中等 feature"的水平)

---

## ⚙️ 设计原则

1. **理论和实践穿插**:不要先把书读完再写代码 —— 读两章就动手,卡住了回头读
2. **由近及远**:先把"你现在能摸到的"(SR 本身、Docker 集群)吃透,再去补"远处的"(论文、其他系统)
3. **小胜利驱动**:每月必须有看得见的产出(一个 PR、一篇笔记、一次 talk),不然学习容易掉队
4. **横纵结合**:横向先把架构铺开,选好方向后纵向深耕,**不要一上来就钻牛角尖**
5. **真实代价**:1 年达到"独立修 bug + 完成中等 feature"是合理目标,**1 年成为某模块专家不现实** —— 即使最快的人也要 18 个月

---

## 📋 12 个月总览

| 阶段 | 月份 | 主题 | 关键产出 |
|---|---|---|---|
| **Phase 0** | Week 0 | 自检前置 | 列出你的差距清单 |
| **Phase 1** | Month 1-2 | 基础奠基 | DB 理论 + 现代 C++/Java |
| **Phase 2** | Month 3 | 项目入门 | 跑通编译/测试,跟一条 SQL 走全程 |
| **Phase 3** | Month 4 | 第一个 PR | 合入第一个 trivial PR |
| **Phase 4** | Month 5-8 | 选定专精方向深耕 | 修 3-5 个真实 bug |
| **Phase 5** | Month 9-11 | 中等特性 | 独立完成一个 enhancement PR |
| **Phase 6** | Month 12 | 复盘 + 规划 Year 2 | 总结 + 选定 Year 2 专精领域 |

---

# 🔧 Phase 0 — Week 0:自检前置(2-3 天)

**目标**:诚实评估你目前的水平,知道哪些基础要补。

## 自检清单(打勾)

```
【C++(BE 必须)】
□ 能流畅写 std::unique_ptr / shared_ptr,知道何时用 move
□ 看到 template<typename T> 不慌
□ 知道 std::atomic 和 memory_order
□ 看过至少一本现代 C++ 的书(Effective Modern C++ 等)
□ 用过 gdb 调试,不只是 print 大法
□ 写过 CMake,理解 link / include 概念

【Java(FE 必须)】
□ 熟悉 JDK 11+ 新特性(var、Stream、Optional、Records)
□ 知道 G1 GC 的基本工作机制
□ 用过 IntelliJ debugger,会条件断点
□ 写过 Maven 多模块项目
□ 用过 java.util.concurrent(CompletableFuture、Lock 等)

【数据库基础】
□ 写过几百行的复杂 SQL(子查询、窗口函数、CTE)
□ 理解 B+Tree、Hash Index 的区别
□ 知道 OLTP vs OLAP,理解列存为啥适合 OLAP
□ 上过 CMU 15-445 或类似课程(至少看过 lecture)

【分布式系统】
□ 知道 CAP / BASE 是啥
□ 至少看过一篇分布式系统论文(Spanner / Raft / ...)
□ 理解 MVCC、Two Phase Commit 概念
```

打勾不到一半的项,就是 Phase 1 要补的。打 80% 以上,你可以缩短 Phase 1。

## 工具环境检查

```bash
# 验证本地集群
docker exec starrocks-dev-compile mysql -h127.0.0.1 -P9030 -uroot -e "SELECT 1;"
docker exec starrocks-dev-compile ls /workspace/output/fe/bin
docker exec starrocks-dev-compile ls /workspace/output/be/bin

# 同时:
# - IDE 装好:IntelliJ Ultimate(FE)+ CLion 或 VSCode + clangd(BE)
# - GitHub fork 一份 StarRocks 仓库,熟悉 PR 流程
# - 本地也 git clone 一份(不只在 Docker 里),IDE 才能索引
```

---

# 📚 Phase 1 — Month 1-2:基础奠基(2 个月)

**核心问题**:在动 SR 代码之前,你脑子里要先有 DB 的"心智模型"和工程能力。

## Month 1:数据库系统基础理论

### Week 1-2:总览课程(强制)

**主任务**:跟 CMU 15-445(本科 DB 系统课)走一遍

- 网址:https://15445.courses.cs.cmu.edu/fall2023/(选最新一年)
- 教授 Andy Pavlo,讲课节奏快、信息密度高、是世界最好的 DB 系统课没有之一
- **每周 2 节课视频,每节 80 分钟**,2 周看完核心 10 节

**重点章节**(按你的 OLAP 目标取舍):

| Lecture | 重点级 | 为啥 |
|---|---|---|
| L01-L02 Relational Model | ⭐ | 复习 |
| L03 Storage I (page) | ⭐⭐ | 看就行 |
| L04-L05 Storage II/III (column store) | ⭐⭐⭐ | **重点!** OLAP 基础 |
| L06 Hash Tables | ⭐⭐⭐ | BE Hash Join 基础 |
| L07-L08 Trees Index | ⭐⭐ | B+Tree(SR 不主要用,但要懂) |
| L11 Joins | ⭐⭐⭐ | **重点!** Hash Join / SMJ / NLJ |
| L12 Sort/Aggregate | ⭐⭐⭐ | **重点!** External Sort |
| L13 Query Execution | ⭐⭐⭐ | **重点!** Volcano / Vectorized |
| L14 Query Planning | ⭐⭐⭐ | **重点!** 优化器 |
| L15 Concurrency Control | ⭐⭐ | 看就行 |
| L21 Distributed OLAP | ⭐⭐⭐ | **重点!** SR 直接相关 |

**作业要不要做**:**做** Project 1(Buffer Pool)和 Project 2(B+Tree)。Project 3-4 跟 OLAP 关系不大可以跳。

### Week 3-4:进阶课程

**CMU 15-721 高级 DB 系统**(Andy Pavlo 的研究生课,直接讲 OLAP):
- 网址:https://15721.courses.cs.cmu.edu/spring2024/
- **每节课配 1-2 篇论文必读**,这就是 Phase 1 的论文阅读骨架

**这门课的论文清单就是你的"论文阅读月历"**,跟着它走:

| 周 | 主题 | 必读论文 | 为啥读 |
|---|---|---|---|
| W3-1 | Storage Model | **C-Store** (Stonebraker 2005) | 列存的祖师爷,SR 列存设计源头 |
| W3-2 | Compression | **Integrating Compression and Execution** (Abadi 2006) | 列存为啥能在压缩态直接计算 |
| W3-3 | Vectorization | **MonetDB/X100** (Boncz 2005) | 向量化执行的开山之作,BE 灵感 |
| W3-4 | Compilation | **HyPer/Hyper** (Neumann 2011) | JIT 代码生成,SR 也在做 |
| W4-1 | Optimizer | **Volcano** (Graefe 1990) | 优化器 + 执行框架的基础 |
| W4-2 | Optimizer | **Cascades** (Graefe 1995) | **SR FE 优化器的直接祖师** |
| W4-3 | Optimizer | **Orca** (Soliman et al 2014, Greenplum) | 现代 Cascades 实现参考 |
| W4-4 | Sketches | **HyperLogLog** (Flajolet 2007) | SR 的 HLL 函数底层算法 |

**论文怎么读**(每篇 1-2 小时):

```
1. 读 Abstract + Introduction(10 min)— 抓住核心问题
2. 跳到 Conclusion 和 Future Work(5 min)— 看结论
3. 看 Figures 和 Tables(15 min)— 算法和实验数据
4. 回头细读核心算法章节(30 min)
5. 写 200 字摘要(30 min)— 用自己的话讲明白
```

**强烈建议**:做 PaperList.md(自己建一个文档),每篇论文记录:
- 一句话总结
- 核心 insight(为什么这个 idea 重要)
- 你看到 SR 里哪部分用了这个

## Month 2:语言 + 工程基础

### 选你的方向(FE / BE 二选一,Month 2 集中补)

#### 路径 A:BE / C++ 方向

**必读书**:
1. **《Effective Modern C++》** Scott Meyers(只读 Item 1-25 即可)
   - 重点:move、forward、unique_ptr/shared_ptr、constexpr
2. **《C++ Concurrency in Action》** Anthony Williams(读 Ch1-7)
   - 重点:atomic、memory order、lock-free queue 思想

**必练**:
- LeetCode C++ 刷 30 题(熟手感)
- 写一个 thread-safe lock-free queue,理解 memory_order
- 用 gdb 调试一个 segfault 程序到底
- 用 perf record + flame graph 分析一个 CPU 密集程序

**关键技能点**:
```
□ 看懂 SR 里 std::shared_ptr<Column> 这种代码
□ 看懂 template<typename T> 的 column 抽象
□ 看懂 SIMD intrinsic(SSE/AVX)
□ 看懂 brpc / bthread 的协程模型(SR 内部 RPC 用 brpc)
```

#### 路径 B:FE / Java 方向

**必读书**:
1. **《Effective Java (3rd Ed)》** Joshua Bloch(全书必读,经典)
2. **《Java Concurrency in Practice》** Brian Goetz(Ch1-10)
3. **《Optimizing Java》** Benjamin Evans(JVM 性能,选读)

**必练**:
- 用 IntelliJ debugger 完整调一遍一个 Spring/Hibernate 项目
- 用 jstack/jmap 分析一个 OOM 的 dump
- 用 async-profiler 给一个 Java 程序生成火焰图
- 学 ANTLR4(SR Parser 用),写一个简单计算器 grammar

**关键技能点**:
```
□ 看懂 ANTLR4 的 .g4 文件(SR Parser 直接读这个)
□ 看懂 Visitor 模式(SR AST 遍历到处都是)
□ 看懂 CompletableFuture 异步链
□ 理解 ThreadLocal + ConnectionContext
```

### Phase 1 期末自检(Month 2 末)

你应该能 **不参考资料** 答出来:

```
1. 列存比行存适合 OLAP 的根本原因是什么?(至少 3 个理由)
2. Volcano 执行模型为啥被向量化模型取代?瓶颈在哪?
3. Hash Join 和 Sort Merge Join 各自适用什么场景?
4. Cascades 和 Volcano 优化器框架的本质区别?
5. LSM-Tree 解决了 B+Tree 的什么问题?代价是什么?
6. C++ 里 std::shared_ptr 在多线程下安全吗?哪些操作安全哪些不?
7. Java G1 GC 怎么避免长时间 STW?
```

答不上的回去补对应章节。

---

# 🚀 Phase 2 — Month 3:项目入门(1 个月)

**核心问题**:把 SR 代码从"陌生"变成"你能在里面闲逛"。

## Week 9:跑通完整开发流程

### Day 1-2:深度熟悉本地环境

```bash
# 1. 完整重编译一次(感受流程)
docker exec -u devuser starrocks-dev-compile bash -c "cd /workspace && ./build.sh --fe"
docker exec -u devuser starrocks-dev-compile bash -c "cd /workspace && ./build.sh --be"

# 2. 跑全套单测,看通过率
./run-fe-ut.sh
./run-be-ut.sh

# 3. 跑 SQL 集成测试
cd test && python3 run.py -v -d sql/query_test
```

### Day 3-5:IDE 索引 + 阅读规范

```
1. IntelliJ 打开 fe/ 模块,等索引完成(可能要 30 分钟)
2. CLion 打开 be/ 模块,配 CMake 到 build_Release/
3. 读完:
   - handbook/index.md(已读)
   - handbook/domains/backend.md
   - handbook/domains/frontend.md
   - be/AGENTS.md
   - fe/AGENTS.md
4. 标注疑问点,Month 后期回来回答
```

## Week 10:跟一条 SQL 走完全链路(本月核心练习)

**目标**:选一条最简单的 SQL(`SELECT 1+1`),手工跟踪它在 SR 内部经过的每个文件、每个函数。

### 推荐路径

```
Step 1 [FE Protocol]
└─ MysqlChannel 接收 client 包
   文件:fe/fe-core/src/main/java/com/starrocks/mysql/MysqlChannel.java

Step 2 [FE Parser]
└─ ANTLR4 grammar 解析 SQL
   文件:fe/fe-core/src/main/java/com/starrocks/sql/parser/StarRocks.g4
   入口:SqlParser.parseStatement(...)
   产物:AST(StatementBase 树)

Step 3 [FE Analyzer]
└─ 语义分析、绑定 schema
   入口:Analyzer.analyze(StatementBase, ConnectContext)
   关键:每种 Statement 的 visit 方法

Step 4 [FE Optimizer]
└─ 进入 Cascades
   入口:StatementPlanner.plan(...)
   关键类:Optimizer / Memo / Group / TaskScheduler
   产物:OptExpression(物理计划)

Step 5 [FE Planner]
└─ 转 ExecPlan,切 PlanFragment
   类:PlanFragmentBuilder

Step 6 [FE Coordinator]
└─ Coordinator 把 fragment 分发给 BE
   类:DefaultCoordinator
   通过 Thrift RPC 调 BE

Step 7 [BE]
└─ FragmentMgr 接收
   文件:be/src/runtime/fragment_mgr.cpp
   启动 Pipeline 执行

Step 8 [BE Pipeline]
└─ 创建 driver、执行算子
   文件:be/src/exec/pipeline/pipeline_driver.cpp

Step 9 [回程]
└─ 结果通过 RPC 回到 FE Coordinator
   FE 通过 MysqlChannel 返给 client
```

**练习方式**:
1. 用 IDE 设断点,真跑一次,看每步实际经过哪些函数
2. 写一份"SR SQL 执行路径笔记"(给未来的自己看)
3. 画一张时序图

**这是整个 12 个月里最值得花时间的练习**。它把你从"看 SR 代码"变成"理解 SR 架构"。

## Week 11-12:模块小巡游

每个模块花 1-2 天浏览,知道它在哪、做什么,不求精通:

| 模块 | 时间 | 任务 |
|---|---|---|
| `fe/.../sql/parser` | 半天 | 看 grammar 怎么写,试着加一个简单语法 |
| `fe/.../catalog` | 半天 | 看 Database / Table / Column 怎么组织 |
| `fe/.../qe/Coordinator.java` | 1 天 | 看协调器逻辑,跑一遍 EXPLAIN |
| `fe/.../sql/optimizer` | 2 天 | 看 Memo / Group / Rule 接口 |
| `be/src/exec/pipeline` | 1 天 | 看 Pipeline / Driver / Operator 基类 |
| `be/src/storage` | 1 天 | 看 Tablet / Rowset / Segment 关系 |
| `be/src/column` | 半天 | 看 Column / ColumnFactory |
| `gensrc/proto` | 半天 | 看几个核心 proto 文件(`Types.proto`,`InternalService.proto`) |

### Phase 2 期末自检

你应该能 **画图** 回答:

```
1. 一条 SELECT 从 client 到结果返回的完整时序
2. FE 内部哪些模块,各自接口是什么
3. BE 一个 PlanFragment 怎么启动起来的
4. 编译 + 测试 + 启动集群的完整 dev workflow
```

---

# 🎯 Phase 3 — Month 4:第一个 PR(1 个月)

**核心问题**:从"会读代码"到"能改代码"的跃迁。

## Week 13-14:找一个好的 first issue

**别自己找 bug**,去 SR GitHub 找:

```
https://github.com/StarRocks/starrocks/issues
filter: label:"good first issue"  OR  label:"help wanted"
```

候选类型(按难度递增):

| 类型 | 难度 | 推荐度 |
|---|---|---|
| 文档错误 / 注释 | ⭐ | 不推荐(学不到东西) |
| 错误信息不友好 | ⭐⭐ | 推荐(改 FE 简单字符串) |
| 加一个 SQL 函数 | ⭐⭐⭐ | **强烈推荐** |
| 修一个简单 NPE / segfault | ⭐⭐⭐ | 推荐 |
| 给一个算子加单测 | ⭐⭐ | 推荐(熟悉测试体系) |

### 建议:第一个 PR 选 "添加一个标量函数"

例子:实现一个 `lpad_chars(str, length, char)` 函数(假设社区还没有)。

为啥推荐:
- 改动小、影响面可控
- 同时碰 FE(函数注册 + 类型推导)和 BE(函数实现)
- 必须写 UT(强制学测试体系)
- 必须写 FT(强制学 SQL 集成测试)

## Week 15:实际写代码

跟着已合入的类似 PR 改:

```
1. git log --oneline | grep -i "Add.*function"
   找最近合入的"加函数"PR
2. 看那个 PR 的 diff,理解都改了哪几个文件
3. 模仿改动
4. 跑 UT + FT
5. 提 PR
```

**Tips**:
- Commit message 必须 imperative,英文,符合 PR 标题规范
- PR description 填完整(题目对应 issue、test plan、screenshots)
- 不要在 PR 里堆砌无关改动

## Week 16:Review 循环

```
- Reviewer comment → 改 → push
- 不懂的 comment 直接问,不要装懂
- 如果 CI 挂,定位日志、修
- 平均 SR 社区 PR 合入要 1-3 周
- 这期间继续看代码,不要干等
```

### Phase 3 期末自检

```
□ 你的第一个 PR 已合入(或在合入中,不卡你)
□ 你完整理解了 SR 的 commit / review / CI 流程
□ 你写过至少一个 UT(C++ Google Test 或 Java JUnit)
□ 你写过至少一个 FT(test/sql 下的 SQL 测试)
```

---

# 🔬 Phase 4 — Month 5-8:专精方向深耕(4 个月)

**核心问题**:从泛泛了解到某模块的"在内行"。

## Month 5:选定方向

四个候选方向(各自的"看到第一份代码会不会兴奋"的判断标准):

| 方向 | 兴奋点 | 不适合 |
|---|---|---|
| 🅰️ **优化器** | 喜欢"规则匹配"、"代价模型"、"逻辑推理" | 怕 Java,不喜欢抽象 |
| 🅱️ **执行引擎** | 喜欢"性能"、"SIMD"、"底层" | 怕 C++,不喜欢深 profile |
| 🅲 **存储引擎** | 喜欢"持久化"、"LSM"、"IO" | 不喜欢琢磨边界条件 |
| 🅳 **数据湖 / 外表** | 喜欢"开放格式"、"生态对接" | 不喜欢 RPC 协议、JNI |

**选不出来的话**:**优化器**是最有迁移价值的(SR、Doris、ClickHouse、Spark、Calcite 全部相关),也最能锻炼 CS 综合能力。**默认推荐选优化器**。

## 详细规划:🅰️ 优化器方向(默认推荐)

### Month 5:优化器框架理解

**论文必读**(按顺序):

1. **Volcano** (Graefe 1990) — 重读一次,这次带着 SR 代码读
2. **Cascades** (Graefe 1995) — 重点理解 Memo / Group / Expression 概念
3. **Orca** (Soliman et al 2014, SIGMOD) — Greenplum 的现代 Cascades 实现
4. **Calcite** (Begoli et al 2018) — Apache Calcite 设计,可对比 SR

**代码精读**(每天 2 小时):

```
Week 17:
- fe-core/.../sql/optimizer/Optimizer.java(入口)
- fe-core/.../sql/optimizer/Memo.java(核心数据结构)
- fe-core/.../sql/optimizer/Group.java
- fe-core/.../sql/optimizer/OptExpression.java

Week 18:
- fe-core/.../sql/optimizer/task/(任务调度)
  - OptimizeGroupTask.java
  - ExploreGroupTask.java
  - ApplyRuleTask.java
- fe-core/.../sql/optimizer/rule/RuleSet.java(所有规则列表)

Week 19:
- 挑 5 个简单规则源码读
  - EliminateProjectRule
  - PushDownPredicateRule
  - JoinReorderRule

Week 20:
- fe-core/.../sql/optimizer/statistics/(代价 + 统计)
- fe-core/.../sql/optimizer/cost/
```

**配套实战**:
- 用 `TRACE TIMES OPTIMIZER` 看每条规则的耗时
- 用 `TRACE LOGS OPTIMIZER` 看决策日志
- 改一个规则,加日志,看它什么时候被触发

### Month 6:MV 改写专题

**论文必读**:

1. **Materialized Views: A Survey** (Roy et al 1999)
2. **Optimizing Queries Using Materialized Views: A Practical, Scalable Solution** (Goldstein & Larson 2001, SIGMOD)
   - **这是 SR MV 改写的直接理论基础!**
3. **Foundations of Automatic View Selection** (Theodoratos & Sellis 1997)
4. **View-Based Query Containment** (Halevy 2001) — 改写正确性证明

**代码精读**:

```
fe-core/.../sql/optimizer/rule/transformation/materialization/
├── MaterializedViewRewriter.java         ← 主入口
├── MaterializedViewRule.java
├── MvRewriteContext.java
├── EquationContext.java                  ← 等价类
├── PredicateSplit.java                   ← 谓词分解
├── ColumnRefMapping.java
├── compensator/                          ← 补偿计算
└── (一堆具体规则文件)
```

**配套实战**:
- 把 demo 扩展:故意造 5 个改写**不命中**的 case,用 TRACE LOGS MV 找出原因
- 写一篇技术博客 / 笔记:《SR MV 改写的 5 种失败模式》
- 选一个目前不支持的 MV 改写场景,提一个 issue 讨论

### Month 7:Cost 模型 + 统计信息

**论文必读**:

1. **Selectivity Estimation Without the Attribute Value Independence Assumption** (Poosala 1997)
2. **Sampling-Based Selectivity Estimation** — 系列论文
3. **Cardinality Estimation Done Right** (Leis et al 2015)
4. **DeepDB** (Hilprecht et al 2020) — 神经网络估计基数,前沿

**代码精读**:

```
fe-core/.../sql/optimizer/statistics/
├── StatisticsCalculator.java        ← 给每个 plan 估算 stats
├── ColumnStatistic.java
└── HistogramStatistic.java

fe-core/.../sql/optimizer/cost/
├── CostModel.java                   ← 各种算子的 cost 公式
└── StatisticsEstimateUtils.java
```

**配套实战**:
- 在本地造一张表,跑 `ANALYZE`,看统计信息
- 对比 EXPLAIN 估算行数 vs 实际行数,理解 Cost 误差
- 改一下某个算子的 cost 公式,看 plan 变化

### Month 8:Join Reorder + 复杂场景

**论文必读**:

1. **Selecting the Optimal Order of Joining Records** (Ibaraki & Kameda 1984) — 经典
2. **Dynamic Programming Strikes Back** (Moerkotte & Neumann 2008)
3. **Adaptive Optimization** (Babu et al) — 自适应优化

**代码精读**:

```
fe-core/.../sql/optimizer/rule/transformation/
├── JoinReorderGreedy.java         ← 贪心(快但不一定最优)
├── JoinReorderLeftDeep.java       ← 动态规划(精确但慢)
└── JoinReorderExhaust.java        ← 穷举(只 ≤4 表用)
```

**Month 8 末交付**:
- 5-8 个已合入的小 PR(每个 1-3 天工作量)
- 一份 1 万字的"SR 优化器内部架构"笔记
- 给团队/朋友做一次 30 分钟分享

---

## 详细规划:🅱️ 执行引擎方向(简化版)

**Month 5**:Pipeline 框架
- 论文:Morsel-driven parallelism (Leis 2014), Photon (Databricks 2022)
- 代码:`be/src/exec/pipeline/` 全部

**Month 6**:向量化 + 表达式
- 论文:MonetDB/X100 (Boncz 2005, 重读), Vectorwise (Zukowski 2008)
- 代码:`be/src/exprs/` + `be/src/column/`

**Month 7**:Join + Aggregate 内部实现
- 论文:Hash Join 优化系列论文,Radix Cluster (Manegold 2002)
- 代码:`be/src/exec/hash_joiner*`, `be/src/exec/aggregator*`

**Month 8**:JIT 编译 + SIMD
- 论文:HyPer LLVM (Neumann 2011)
- 代码:`be/src/exprs/jit/`

---

## 详细规划:🅲 存储引擎方向(简化版)

**Month 5**:LSM 基础 + Rowset/Segment
- 论文:LSM-Tree (O'Neil 1996), bLSM (Sears 2012)
- 代码:`be/src/storage/rowset/`, `segment_v2/`

**Month 6**:Compaction 策略
- 论文:RocksDB compaction papers
- 代码:`be/src/storage/compaction*`

**Month 7**:Primary Key 表 + 主键索引
- 代码:`be/src/storage/primary_index*`, `persistent_index*`

**Month 8**:Shared-Data / Lake
- 论文:Snowflake (2016), Photon (2022)
- 代码:`be/src/storage/lake/`

---

## 详细规划:🅳 数据湖 / 外表方向(简化版)

**Month 5**:Catalog 抽象 + Connector 框架
- 代码:`fe-core/.../catalog/connector/`, `be/src/connector/`

**Month 6**:Iceberg 表格式
- 论文:Iceberg 表格式规范、Delta Lake (Armbrust 2020)
- 代码:`be/src/connector/iceberg_*`

**Month 7**:Parquet / ORC 读取优化
- 论文:Parquet 格式规范
- 代码:`be/src/formats/parquet/`

**Month 8**:JNI 桥接 + Java Extension
- 代码:`java-extensions/`

---

# 🚀 Phase 5 — Month 9-11:中等特性独立完成(3 个月)

**核心问题**:从"修 bug"到"加 feature"的跃迁。

## Month 9:找一个 feature 级任务

来源:
1. GitHub `Enhancement` 标签的 open issue
2. SR 路线图(`handbook/plans/`)里你能 hold 的小项
3. 自己提一个 design discussion,看社区认不认

### 推荐方向(优化器方向举例)

| Feature 类型 | 难度 | 周期 |
|---|---|---|
| 加一个新的优化规则(简单等价改写) | ⭐⭐⭐ | 2-3 周 |
| 支持一种新 MV 改写模式 | ⭐⭐⭐⭐ | 4-6 周 |
| 优化某个特定 query pattern 的 cost 估算 | ⭐⭐⭐⭐ | 4-6 周 |
| 实现一个新的 Join 策略 | ⭐⭐⭐⭐⭐ | 8 周 |

## Month 10:设计 + 实现

```
Week 1     设计文档(写 RFC,贴 GitHub Discussion)
Week 2-3   核心实现
Week 4     测试 + 边界场景
```

**强烈建议**:**先写设计文档再写代码**。哪怕 1 页也行。社区 reviewer 看到"作者想清楚了"会更愿意细看代码。

## Month 11:Review + 合入 + 复盘

可能要改 3-5 轮才能合入。**这是正常的**,大特性 PR 平均 review 时间是 4-8 周。

期间你应该:
- 继续看代码、看其他人 PR(保持手感)
- 帮其他人 review(社区贡献的硬指标)
- 写一篇技术博客复盘这个 feature

---

# 🎓 Phase 6 — Month 12:复盘 + Year 2 规划(1 个月)

## 复盘

```
□ 我合入了几个 PR?哪些是 bug fix,哪些是 feature?
□ 我现在能在 SR 的哪个目录"闭眼走"?
□ 哪些论文我读完真的看懂了,哪些只是"看过"?
□ 我能给别人讲清楚 SR 哪几个概念?
□ 我还有哪些短板?
```

## Year 2 选择题

| 选择 | 适合人 | 路径 |
|---|---|---|
| 继续深耕原方向 | 想成为某模块"作者级" | 接大特性、参与设计 review、影响路线图 |
| 横向扩展到二专 | 想做全栈 SR | 用 Year 2 在另一个方向也达到 Year 1 末水平 |
| 学术化 | 想发论文 / 读 PhD | 做创新点 + 投 SIGMOD/VLDB |
| 跳到相关项目 | 想看更广 | Doris / ClickHouse / DuckDB / DataFusion 都可以 |

---

# 📚 总论文清单(贯穿 12 个月)

## 必读(Tier 1)— 不读就不算入行(共 12 篇)

| # | 论文 | 第几个月 |
|---|---|---|
| 1 | Volcano (Graefe 1990) | M1 |
| 2 | Cascades (Graefe 1995) | M1 |
| 3 | C-Store (Stonebraker 2005) | M1 |
| 4 | MonetDB/X100 (Boncz 2005) | M1 |
| 5 | LSM-Tree (O'Neil 1996) | M1-2 |
| 6 | Hyper (Neumann 2011) | M1-2 |
| 7 | Optimizing Queries Using MVs (Goldstein & Larson 2001) | M6(优化器方向) |
| 8 | Spanner (Corbett 2012) | M2 |
| 9 | Snowflake (Dageville 2016) | M2 |
| 10 | Photon (Databricks 2022) | M5(执行方向) |
| 11 | Orca (Soliman 2014) | M5(优化器方向) |
| 12 | Morsel-driven (Leis 2014) | M5(执行方向) |

## 选读(Tier 2)— 进阶必备

| 论文 | 主题 |
|---|---|
| Vectorwise (Zukowski 2008) | 向量化进阶 |
| Hekaton (Diaconu 2013) | 内存数据库 |
| HyperLogLog (Flajolet 2007) | 基数估计 |
| Calcite (Begoli 2018) | 优化器框架对比 |
| Cardinality Estimation Done Right (Leis 2015) | 统计 |
| F1 Query (Samwel 2018) | Google 分布式查询 |
| Procella (YouTube 2019) | 多负载 OLAP |

## 横向(Tier 3)— 拓宽视野

ClickHouse / DuckDB / DataFusion / Doris 的 paper / design doc 各 1 份

---

# 📖 总书籍清单

| 书 | 啥时候看 | 看哪些 |
|---|---|---|
| 《Designing Data-Intensive Applications》Kleppmann | Month 1 | 全书,反复读 |
| 《Database System Concepts》Silberschatz | Month 1 | 第 12-20 章 |
| 《Effective Modern C++》Meyers | Month 2 (C++ 方向) | Item 1-25 |
| 《C++ Concurrency in Action》Williams | Month 2 (C++ 方向) | Ch1-7 |
| 《Effective Java》Bloch | Month 2 (Java 方向) | 全书 |
| 《Java Concurrency in Practice》Goetz | Month 2 (Java 方向) | Ch1-10 |
| 《数据库系统实现》Garcia-Molina | 全程当工具书 | 按需查 |
| 《Database Internals》Petrov | Month 3-4 | 全书 |
| 《Streaming Systems》Akidau | Year 2 | 选读 |

---

# 🎬 第一周具体做啥(立即可执行)

```
Day 1 (今天):
- 把这份计划保存到一个 Notion / 飞书 / 备忘录
- 打开 https://15445.courses.cs.cmu.edu/fall2023/,看 L01 视频
- 在 GitHub fork StarRocks 仓库

Day 2:
- 看 15-445 L02,做 PaperList.md
- 本地 git clone fork 后的 SR 仓库(不只在 Docker 里)
- IntelliJ 打开 fe/,等索引

Day 3:
- 看 15-445 L03(Storage)
- 读 handbook/index.md(再细读)
- 读 handbook/domains/backend.md

Day 4:
- 看 15-445 L04(Column Store)
- 读 handbook/domains/frontend.md
- 在 GitHub 找 "good first issue" 标签的 issue,看 5 个

Day 5:
- 看 15-445 L05
- 读论文 C-Store(Tier 1 第 3 篇)
- 写 200 字总结到 PaperList.md

Day 6-7 (周末):
- 完整跑一遍 docker exec 启 FE/BE,跑几个 MV 查询
- 设几个 IntelliJ 断点,跟踪一条 SELECT 1 的执行
- 不强求看懂,目标是"建立第一印象"
```

---

# ⚠️ 关于这份计划的几个真相

1. **没有人 100% 按计划走** — 路上你会偏离,这没关系,只要总方向没偏就行
2. **计划是骨架不是枷锁** — 遇到有意思的问题深入下去比赶进度更值
3. **找一个学习伙伴 / mentor** — 一个人学比有伙伴慢 3 倍,真的
4. **GitHub Discussion / StarRocks 社区微信群多潜水** — 看别人怎么讨论问题,比单读代码学得快
5. **不要怕问蠢问题** — SR 中国团队对社区贡献者很友好,有问题直接 issue / 群里问
6. **保护好你的健康** — 不要熬夜刷论文,持续性比强度重要 100 倍

---

# 🔗 重要链接清单

| 资源 | URL |
|---|---|
| StarRocks 主仓库 | https://github.com/StarRocks/starrocks |
| StarRocks 官方文档 | https://docs.starrocks.io/ |
| StarRocks 社区讨论 | https://github.com/StarRocks/starrocks/discussions |
| CMU 15-445 | https://15445.courses.cs.cmu.edu/ |
| CMU 15-721 | https://15721.courses.cs.cmu.edu/ |
| 论文检索 | https://dblp.org/ |
| DB-Engines 排行 | https://db-engines.com/ |

---

> 这份计划是动态的,每个月底回来更新一次:
> - 实际进度 vs 计划
> - 哪些假设错了
> - Year 2 方向是否调整
>
> Good luck. The journey of a thousand miles begins with a single step. 🚀
