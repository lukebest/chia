# 上下文简报(供规划 / 写作 / 验证子代理使用)

## 作者意图(用户原话要点)

写一篇中文技术文章,讲 AI agent 在「自动架构探索」与「RTL agentic 生成」上的思考。

作者核心观点:

1. 自动架构探索与 RTL 生成应拆成**两个独立课题**。
2. 二者中间用**周期精确(cycle-accurate)模型**作为桥接。
3. 理由一:自动架构探索的**产出**本身就应当是周期精确模型。
4. 理由二:RTL 生成应直接以周期精确模型作为**信息对齐的主要手段**,因为**代码的歧义性比自然语言小**。
5. 可引用 CHIA paper 的理论与案例、`docs/信息论.pdf` 的信息论框架,并加入学术界类似课题的深度洞察。
6. 发布场景:公司内部技术交流社区;目标是打造作者个人技术品牌,体现深度思考与洞察力(不是综述,不是产品宣传,要有鲜明立场、可被反驳的论点、有信息增量)。

## 信息论框架(摘自 docs/信息论.pdf《智能体辅助编程的信息论价值模型》,逐条核对过原文)

- S:目标软件的完整规约,H(S) 为其熵(描述 S 所需最小信息量)。
- K = K_pretrain ∪ K_RAG ∪ K_context:人与智能体的共享先验(预训练 / 检索注入 / 会话上下文)。
- 复杂性分解:H(S) = H(S_e) + H(S_a | S_e),S_e 本质复杂性(业务逻辑、独特算法、领域边界条件),S_a 偶然复杂性(样板、胶水、类型声明、错误处理模式)。
- E_manual ∝ H(S);E_agent ∝ H(S | K) = H(S) − I(S;K)。
- **定理 2.1**:智能体价值 V = E_manual − E_agent ∝ I(S;K)。需求完全原创、先验无法覆盖时 I(S;K)→0,价值趋零。
- **命题 3.1**:训练充分的 agent 使 H(S_a | S_e, K) ≈ 0,人类工作量收缩为 H(S_e | K)。
- **歧义开销因子 α ≥ 1**:自然语言相对形式化代码的歧义开销;真实工作量 E_real = α·H(S|K)。
- **定理 4.1(有效性条件)**:α·H(S|K) < H(S) ⇔ I(S;K)/H(S) > 1 − 1/α。α=2 时先验必须覆盖至少 50% 信息量。
- **推论 4.2(悖论区间)**:I(S;K)→0 时条件不满足,意图对齐成本 ≥ 直接编码成本。
- **迭代交互**:T 轮修正 m_t,H(m_t) = H(S|K,m_1..m_{t-1}) − H(S|K,m_1..m_t),求和伸缩得 Σ H(m_t) = H(S|K) − H(S|K,m_1..m_T);收敛时总信息量 = H(S|K),与一次性表达无异;迭代的好处是认知负荷分散、反馈引导、隐式确认。
- **工程策略表**:更好的 RAG(K↑)、领域微调(K_pretrain 与 S 相关性↑)、**形式化 DSL(α→1)**、示例/草图/引用(低熵通道)、持久化规则(Cursor Rules / AGENTS.md)。

### 与作者观点的天然接口(供规划者深化,需自行严谨推导,勿生搬)

- 「用周期精确模型做对齐」≈ 把 α 压向 1 的形式化 DSL 策略;周期精确模型是 *可执行* 的形式化规约。
- 架构探索阶段的目标是在 H(S) 巨大的空间里找到 S 的「本质复杂性」S_e(微架构决策);RTL 阶段的大部分是 S_a(流水线寄存器、握手、胶水)—— 一旦周期精确模型确定了 S_e,RTL 生成近似是 H(S_a | S_e, K) 问题。
- 周期精确模型同时是:探索阶段的评估器(fitness)、探索阶段的产出(artifact)、RTL 阶段的规约(spec)与 oracle(golden)。

## CHIA 相关事实(来自仓库 docs/,均已核对原文)

- CHIA:敏捷、有原则的硬件/软件协同设计研究框架;把 workflow 抽象为图(loops + clusters),基于 Ray;LLM/agent 是一等节点;已有 Chipyard、FireSim、Hammer、CIRCT、gem5、ChampSim、Verilator 节点,以及 Claude Code / Codex / Copilot 等 agent CLI 节点。论文:arXiv 2606.27350(docs 中给出的链接 https://arxiv.org/abs/2606.27350)。
- **架构探索案例**(`case-studies/architectural-discovery-basic`):ArchAgent 式进化编码 agent(AlphaEvolve / AdaEvolve / OpenEvolve / EvoX / GEPA,经 SkyDiscover)为 **ChampSim**(微架构模拟器)发现数据预取器。每轮:LLM 提出 C++ 候选 → 编译 → 并行跑 trace → geomean IPC 作为 fitness。三函数评估器接口 build_fn / run_fn / result_mapper_fn,可换模拟器。引用 ArchAgent: Gupta et al., "ArchAgent: Agentic AI-driven Computer Architecture Discovery", arXiv 2602.22425;该工作用进化 agent 数日内发现优于人类 SOTA 的 LLC 替换策略。简化示例:3 条 SPEC CPU 2017 trace、5 个 worker、10 次迭代、1 小时内。构建失败得分 0.0,搜索继续。
- **gem5 对齐案例**(`case-studies/gem5-alignment`):LLM-in-the-loop 修改 gem5 配置与 C++ 源码,使其周期数匹配 BOOM RTL(Verilator golden)。每轮:从最优结果采样父代 → LLM 编辑 → 重建 gem5 → 跑基准 → 与 Verilator golden cycle 对比得 %diff。给 agent 的诊断信号:O3PipeView 流水线 trace(对比 gem5 retire cycle 与 Verilator commit cycle,按 PC 对齐)+ BOOM TMA 性能计数器(divider_active、stq_full、dcache_miss、br_mispredict …)对照 gem5 stats.txt。**结果:medium BOOM 上运行 10.5 天,gem5 核与配置周期数平均误差 < 3%**。文中指出:对齐「即便对大团队也是巨大工作量,对小团队与学术界几乎不可能,通常 simulator 不对齐或从未对齐」。
- **RISC-V 扩展案例**(`case-studies/riscv-extensions`):LLM 在 BOOM(Chisel)中实现 ISA 扩展,以 Spike 为 golden oracle 做逐指令 lockstep 协同仿真(cospike),directed test(每指令一个程序,覆盖率 by construction 100%)+ base-ISA 回归 + riscv-dv 随机压力测试;分歧交给 debug LLM 修 RTL;最后综合测 PPA。**结果:MegaBOOM 上实现并验证 Bitmanip(Zba/Zbb/Zbc/Zbs)、标量密码(Zbk*/Zkn*)、Zicond;约 5.6% 与 3.5% 加速(OpenSSL 密码最高 10×),无时序回退,Sky130 面积开销小**。关键设计:「没有手写 expected output —— golden model 就是检查」;「divergence 在发生的那个周期被抓到,而非从最终结果推断」。
- **时序优化案例**(`case-studies/timing-optimization`):LLM 改 BOOM Chisel 缩短关键路径;迭代:读时序报告 → 改 Chisel → 重建 → 综合 BoomTile + Verilator 并行(综合量 slack,Verilator 把关正确性并估 IPC 影响)→ 子分支入 SQLite 变体树。Iron Law(time/program = instr × CPI × cycle time)。**结果:15 次迭代频率提升 > 2×,IPC 仅损失 3.3%,净 Iron-Law 性能 1.97×(Sky130)**。提供 timing_experiment 子模块 A/B 综合工具,让 agent 低成本验证想法再决定是否投入整块综合;三种 prompt 变体控制 IPC-neutral vs IPC-trading。
- **CIRCT issue 案例**(`case-studies/circt-issue-solving`):assess → reproduce → fix → **deterministic verify(无 LLM)** → regression repair → writeup。assess 阶段枚举"合理的解决方式",若多于一种且无依据则返回 UNCLEAR 而不是自信地给错误补丁(这就是「歧义」被显式建模并拒绝的案例)。结果:评估 16 个 issue,5 个可复现且方案明确,全部解决,3 个 PR 被上游接受。
- CHIA 的原则:Fault tolerance、caching & bypass、profiling;「好科学」;以及「人始终在回路」(LLM 不直接写 GitHub)。

## 作者尚未明确、规划者需要判断与取舍的问题

- 「周期精确模型」的精确定义:cycle-accurate vs cycle-approximate vs 事件驱动 vs 可执行的 Chisel/SystemC/gem5 模型?它与「RTL」的边界在哪里(RTL 本身也是周期精确的)?作者的主张其实是:周期精确模型 = *行为/时序层面的可执行规约*,不要求可综合,不承担物理实现细节(时序收敛、面积、功耗、布局布线)。规划者需要把这个边界讲清,并预判读者最容易提出的反驳。
- 如何把「周期精确模型作为探索产出」与 CHIA 案例中的 ChampSim/gem5 对接,同时诚实说明 gem5 对齐案例证明了「模型与 RTL 对齐很贵」这一风险(也是本文立场的潜在最强反例,须正面回应)。
