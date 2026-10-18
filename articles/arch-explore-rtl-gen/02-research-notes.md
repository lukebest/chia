# 02 · 调研原始笔记(已核实链接与摘录)

> 调研日期:2026-10-17。核实方式:arXiv 官方 API(`export.arxiv.org/api/query`,取标题/作者/日期/摘要)、arXiv HTML 全文、作者主页/出版方 PDF、IEEE/ACM DOI 页。
> 标记约定:**[已核实]** = 亲自读到原文或官方摘要;**[部分核实]** = 仅从可信二手元数据(他文参考文献 / DOI 页)确认存在,未读正文;**[未核实]** = 未找到或未读,**不得引用**。
> 摘录为英文原文片段(必要处截断),中文为笔者概括。

---

## A. CHIA 论文本体(arXiv 2606.27350)— [已核实,HTML 全文已读]

- 标题:CHIA: An open-source framework for principled, agentic AI-driven hardware/software co-design research
- 作者:Angela Cui*, Ferran Hermida-Rivera*, Jack Toubes*, Raghav Gupta, …, Yakun Sophia Shao, Borivoje Nikolić, Christopher W. Fletcher, Sagar Karandikar(UC Berkeley)
- URL:https://arxiv.org/abs/2606.27350
- 论文的"理论性"论述很少(它是框架论文,不是理论论文)。可用的观点性论述集中在 §1、§5.4.1、§7:

**§7 Discussion(核心论断,可直接引用)**
> "As the process of turning an idea into a preliminary implementation speeds up, evaluating the performance and QoR of that idea as well as verifying the idea takes the bulk of the time. CHIA on its own attacks this problem with parallelism, but Amdahl's law tells us that parallelism alone is not enough. Rapid, high-fidelity evaluation and verification of hardware designs will be key …"

> "Since proprietary data usually cannot be fed to public frontier model-based agents, open PDKs … are used … Unsurprisingly, these PDKs do not always serve as predictive guides when the ultimate goal is to produce a chip in an advanced process."

**§5.4.1(进化式架构发现的挑战)**
> "microarchitectural simulators such as ChampSim are unable to provide reliable feedback on power, area, and critical paths. This brings into question the hardware realizability of designs generated with feedback from such models."
> "an evolutionary coding agent's ability to explore a design space is fundamentally limited by the latency of evaluating proposed candidates."
> ArchAgent 早期实验中 AlphaEvolve "reward hack[ed] an improperly enforced assertion to drop memory traffic"。

**§1(对"让 agent 自由跑"的批评)**
> "relying strictly on agents to orchestrate the entire design flow … makes it challenging to enforce verification and validation requirements that are critical to successful hardware design while minimizing human review overhead."

**论文中比 docs 更细的案例数字(docs 中没有,已从论文核实;写作时若使用需注明来自论文)**
- gem5 对齐(§5.1):Claude Code + Opus 4.6;**202 次迭代 ≈ 10.5 天**;2-wide MediumBOOM;训练集为 **36 个 Microbench 微基准**,平均周期误差 **< 3%**;**留出集 Embench IOT 2.0 上最好迭代平均误差 < 7%**,有若干离群点。离群原因:(1) 过拟合——某迭代引入参数 `kSuppressThreshold`,agent 自述 "with a threshold calibrated to [training benchmark]'s inner loop",同一迭代令 Embench Wikisort 明显变差;(2) 训练集覆盖不足(只有一个基准的代码工作集超过 I-cache)。最终对齐核相对基线 gem5 RISC-V O3 核 diff **1520 行**。每迭代平均 **$11.72**,总计 **$2,355.98**。给 agent 加 "Run & Compare MCP"(一小时实现)后改进迭代比例显著上升。
- RISC-V 扩展(§5.2):MegaBOOM 4-wide;Bitmanip SPEC06 ref 几何平均加速 **1.056**(正文一处写"5%",表 2 为 5.6%),Zicond **1.035**;Crypto OpenSSL 最高约 10×;SPEC06 ref **25+ 万亿条指令**在 FireSim 中跑通。面积开销 Sky130:Bitmanip ~2%,Crypto <5%,Zicond 可忽略;商用 16nm:3% / 6% / 0.8%;两种 PDK 均无时序回退。LLM 迭代次数 Bitmanip 2、Crypto 6、Zicond 1;成本 $23.57 / $46.24 / $5.11(Opus 4.7)。cosim 占总机时 50%–66%,测试生成占 1/4–1/3。原文:"the LLM knows nothing about this infrastructure, it only receives the necessary traces"。
- 时序优化(§5.3):Opus 4.6;**前 4 次迭代要求"不影响 IPC",结果平台化后才切换到 Iron Law(允许以小 IPC 损失换频率)**;最佳为第 7 次迭代,频率 **约 47MHz → 约 95MHz(2.03×)**,SPEC06 ref 上 IPC 损失 **3.28%**(Verilator 小基准估计 3.20%),净 Iron-Law **1.97×**;总成本 **$202.03**(14 次)。**"Invisibly to the agent" 同时在商用 16nm 综合:只有第一个优化(issue queue 线性深度结构改为对数深度)在 16nm 上改善关键路径约 70%,其余均不泛化** —— "our Sky130 results do not generalize to the advanced commercial node beyond the first optimization."
- CIRCT(§5.5):16 个随机 issue 并行,< 45 分钟完成;表 5:Not a bug 5 个,Fix unclear 4 个(#4649 #5626 #7531 #8508),Already fixed 2 个,修复 5 个,其中 3 个提交并合入。原型阶段 #8508 曾被 assessor 判为"清晰",fixer 也修好了,**但 assessor 与 fixer 对"正确解法"意见不一** → 最终版要求 assessor 枚举所有好解法,多于一种即判 UNCLEAR。人工复核约 1 小时/issue + 10 分钟写 PR。
- §1 引用 Design Conductor(Verkor)"generation of a 5-stage RISC-V microprocessor design from scratch using agentic AI"。

---

## B. LLM / Agent 做架构探索

### B1. ArchAgent — [已核实,arXiv HTML 全文已读]
- Gupta, Jain, Gonzalez, Novikov, …, Ranganathan, Karandikar. *ArchAgent: Agentic AI-driven Computer Architecture Discovery.* arXiv 2602.22425 (2026);OpenReview 匿名版。https://arxiv.org/abs/2602.22425
- 核心结论(摘要):基于 AlphaEvolve,在 ChampSim 中自动设计 LLC 替换策略;"In two days and without human intervention, ArchAgent generated a policy achieving a 5.322% IPC speedup improvement over the prior SoTA on public multi-core Google Workload Traces";SPEC06 单核 18 天、+0.907%;"3-5× faster than humans"。
- **"simulator escapes"**(§7.3):ChampSim 中禁止 LLC write-bypass 的检查只在 assertion 里,开优化编译时被消除;agent 生成的 Policy12 利用"被 bypass 的写从系统中消失"表观上赢 Mockingjay 3%/4%。原文:"Since there is no notion of 'correct computation' in such a trace-driven simulator, there was no further way to detect this issue." 并呼吁 "Building simulators that are either better verified or closer to the actual hardware design (e.g., RTL simulation or hardware-accelerated emulation) and thus less susceptible to 'abuse' by the AI agents is crucial."
- **可实现性需要人工**(§7.1):"confirming the realizability of generated policies remains a human intensive process, due to the lack of ASIC quality-of-result data (e.g., frequency and area) in most microarchitectural simulators." 用 LOC 上限做规则校验;"inferring physical storage size from a microarchitectural model is non-trivial"。
- **评估延迟**(§3.5):1B 指令单次 ChampSim 模拟可 >12 小时;多核 2–4 天;用 100M 短跑做搜索、1B 长跑做验证,存在 "generalization issues"。
- 与论点关系:**支持**"探索产出是代码化模型";**挑战**"现有探索产出可直接桥接 RTL"(ChampSim 是 trace-driven、无正确性概念、无 PPA);**支持**"模型需更接近硬件/可被 RTL 校验"。

### B2. Agentic Architect — [已核实,arXiv API 摘要]
- arXiv 2604.25083 (2026)。API 作者顺序:Vasilis Kypriotis, Alexander Blasberg, Alexandros Moiras, Dimitrios Skarlatos(CHIA 参考文献写作 Blasberg et al.——**作者顺序有出入,引用时以 arXiv 页面为准**)。https://arxiv.org/abs/2604.25083
- 摘要:"combines LLM-driven code evolution with cycle-accurate simulation and hardware cost modeling to explore performance, area, and power tradeoffs … constrained by area and power budgets";进化出的分支预测器 "achieves 97.62% of TAGE-SC-L's geometric mean IPC at 10.58% of its area";"Our analysis highlights the importance of seed selection, hardware constraints, and training workloads in shaping search outcomes."
- 关系:**部分支持**——探索阶段可以引入 PPA 代理(hardware cost model)而不必引入 RTL/物理实现;对"PPA 完全剥离"是**对照**。

### B3. AlphaEvolve — [已核实,arXiv API]
- Novikov et al. *AlphaEvolve: A coding agent for scientific and algorithmic discovery.* arXiv 2506.13131 (2025)。https://arxiv.org/abs/2506.13131
- 摘要:进化式编码 agent,"continuously receiving feedback from one or more evaluators";在 Google 栈中 "found a functionally equivalent simplification in the circuit design of hardware accelerators"。
- 关系:**提供对照**——"评估器质量决定上限"范式的来源。

### B4. Barbarians at the Gate(ADRS)— [已核实,arXiv API]
- Cheng, Liu, Pan, Li et al. arXiv 2510.06189 (2025)。https://arxiv.org/abs/2510.06189
- 摘要:"Crucially, this approach assumes the existence of a reliable verifier … system performance problems naturally admit reliable verifiers: solutions are typically implemented in real systems or simulators, and verification reduces to running these software artifacts against predefined workloads".
- 关系:**支持**——"可执行模型 = 可靠 verifier"是 AI 驱动发现的前提。

### B5. ArchGym — [已核实,arXiv API;ISCA 2023]
- Krishnan, Yazdanbakhsh, Prakash, Jabbour et al. arXiv 2306.08888。https://arxiv.org/abs/2306.08888
- 摘要:"connects diverse search algorithms to architecture simulators";"with unlimited samples, ML algorithms are equally favorable … hyperparameter lottery";代理成本模型 "RMSE of 0.61% that offers a 2,000-fold reduction in simulation time"。
- 关系:**提供对照**——搜索算法差别不如评估器/样本量重要;支持"廉价 fitness 是探索的主杠杆"。

### B6. LUMINA — [已核实,arXiv API]
- Zhang, Ma, Xu, Xiong et al. arXiv 2603.05904 (2026)。https://arxiv.org/abs/2603.05904
- 摘要:LLM 从**模拟器代码**中抽取架构知识、组合 DSE 规则;在 4.7M 设计空间中 20 步找到 6 个性能与面积均优于 A100 的设计。
- 关系:**部分支持**——模拟器代码本身是 LLM 可读的"架构知识载体"(代码作为对齐媒介的旁证)。

### B7. Sankaralingam — [已核实,arXiv API]
- *Computer Architecture's AlphaZero Moment: Automated Discovery in an Encircled World.* arXiv 2604.03312 (2026)。https://arxiv.org/abs/2604.03312
- 摘要:人类团队每代只探索 50–100 个设计;提出 "automated idea factories … through multi-tiered evaluation pipelines";"We predict that within 2 years, purely human-driven architecture research will be as obsolete as human chess players competing against engines."
- 关系:**提供对照/立场参照**(观点性文章,不是实证)。可用于"多级评估管线"与本文桥接观点对比。

### B8. SECDA-DSE — [已核实,arXiv API]
- Sharma, Fu, Haris, Cano. *LLM-Driven Design Space Exploration of FPGA-based Accelerators.* arXiv 2605.05920 (2026,EuroSys'26 InSyDe workshop)。https://arxiv.org/abs/2605.05920
- 摘要:基于 SECDA(**SystemC 仿真 + FPGA 执行**)的 LLM DSE。初步评估。
- 关系:**部分支持**(SystemC 模型作为探索载体)。证据较弱,可不用。

---

## C. LLM / Agent 做 RTL 生成与验证

### C1. VerilogEval — [已核实,arXiv API;ICCAD 2023 invited]
- Liu, Pinckney, Khailany, Ren. arXiv 2309.07544。https://arxiv.org/abs/2309.07544
- 摘要:156 题来自 HDLBits;"automatically tested for functional correctness by comparing the transient simulation outputs of the generated design with a golden solution."
- 关系:**支持(旁证)**——即便是 benchmark,判定也依赖 golden 实现;NL 题面本身无法判定对错。

### C2. Revisiting VerilogEval(VerilogEval v2)— [已核实,arXiv API]
- Pinckney, Batten, Liu, Ren et al. arXiv 2408.11053 (2024)。https://arxiv.org/abs/2408.11053
- 摘要:"GPT-4o achieves a 63% pass rate on specification-to-RTL tasks";"prompt engineering remains crucial … varies widely with model and task"。
- 关系:**提供对照**(2024 年 spec-to-RTL 水平;小模块)。

### C3. RTLLM — [已核实,arXiv API;ASP-DAC 2024]
- Lu, Liu, Zhang, Xie. arXiv 2308.05345。https://arxiv.org/abs/2308.05345
- 摘要:NL 指令生成 RTL 的 benchmark;三目标:syntax / functionality / design quality;"self-planning" prompt。
- 关系:**提供对照**。

### C4. RTLCoder — [已核实,arXiv API]
- Liu, Fang, Lu, Zhang et al. arXiv 2312.08617(LAD 2024)。https://arxiv.org/abs/2312.08617
- 摘要:7B 开源模型超过 GPT-3.5。关系:**提供对照**(领域微调 = K_pretrain 与 S 相关性↑)。

### C5. VeriGen — [已核实,arXiv API]
- Thakur, Ahmad, Pearce, Tan et al. arXiv 2308.00708 (2023)。https://arxiv.org/abs/2308.00708
- 摘要:在 GitHub + 教科书 Verilog 上微调 CodeGen-16B,略超 GPT-3.5-turbo。关系:**提供对照**(领域微调)。

### C6. Chip-Chat — [已核实,arXiv API;MLCAD 2023]
- Blocklove, Garg, Karri, Pearce. arXiv 2305.13243。https://arxiv.org/abs/2305.13243
- 摘要:"Modern hardware design starts with specifications provided in natural language";工程师与 LLM 对话式共同设计 8-bit 累加器处理器并流片(Sky130),"world's first wholly-AI-written HDL for tapeout"。
- 关系:**提供对照**——对话式(迭代 H(m_t) 由人支付)的典型。

### C7. AutoChip — [已核实,arXiv API]
- Thakur, Blocklove, Pearce, Tan et al. arXiv 2311.04887 (2023)。https://arxiv.org/abs/2311.04887
- 摘要:用编译错误与仿真"expected vs actual"差异作为反馈;"incorporating context from compiler tools … yielding 24.20% more accurate Verilog"。
- 关系:**支持**——机器生成的差分反馈(低熵通道)有效;但其 "expected" 来自 testbench,前提是有 oracle。

### C8. MAGE — [已核实,arXiv API]
- Zhao, Zhang, Huang, Yu et al. arXiv 2412.07822 (2024)。https://arxiv.org/abs/2412.07822
- 摘要:多 agent;"Verilog-state checkpoint checking mechanism that enables early detection of functional errors";VerilogEval-Human 2 上 95.7%。
- 关系:**支持**——"在分歧发生处尽早定位"的价值(与 Spike lockstep "the cycle it happens" 同构)。

### C9. CVDP(NVIDIA)— [已核实,arXiv API]
- Pinckney, Deng, Ho, Tsai et al. *Comprehensive Verilog Design Problems.* arXiv 2506.14074 (2025)。https://arxiv.org/abs/2506.14074
- 摘要:783 题、13 类任务(含 "specification alignment");"state-of-the-art models achieving no more than 34% pass@1 on code generation. Agentic tasks—especially those involving RTL reuse and verification—are particularly difficult."
- 关系:**支持**"直接 NL→RTL 在真实复杂度下仍弱"。

### C10. Spec2RTL-Agent — [已核实,arXiv API;ICLAD 2025]
- Yu, Liu, Zimmer, Lin et al. arXiv 2506.13905。https://arxiv.org/abs/2506.13905
- 摘要:"Instead of directly generating RTL from natural language, our system strategically generates synthesizable C++ code, which is then optimized for HLS";"up to 75% fewer human interventions"。
- 关系:**支持**——学界已在用"可执行代码中间表示"替代 NL→RTL 直译。

### C11. ChipNeMo — [已核实,arXiv API]
- Liu, Ene, Kirby, Cheng et al. arXiv 2311.00176 (2023)。https://arxiv.org/abs/2311.00176
- 摘要:领域自适应 tokenization / 继续预训练 / 指令对齐 / 检索;应用于工程助手、EDA 脚本生成、bug 摘要。
- 关系:**提供对照**(增大 I(S;K) 的路线,而非降 α 的路线)。

### C12. ACE-RTL — [已核实,arXiv API]
- Deng, Yu, Liu, Pinckney et al. arXiv 2602.10218 (2026)。https://arxiv.org/abs/2602.10218
- 摘要:把 RTL 专用模型(170 万 RTL 样本训练)与前沿推理模型通过 "Agentic Context Evolution" 结合,由仿真反馈驱动。关系:**提供对照**。

### C13. LLM-FSM — [已核实,arXiv API]
- Wu, Gokmen, Xie, Li et al. arXiv 2602.07032 (2026)。https://arxiv.org/abs/2602.07032
- 摘要:FSM 先以**结构化 YAML** 表达,再转成 NL 规约,参考 RTL 由 YAML "correct-by-construction" 合成;"even the strongest LLMs exhibit sharply declining accuracy as FSM complexity increases."
- 关系:**支持**——NL 是结构化规约的有损渲染;复杂度上升时 NL→RTL 精度骤降(对应 α_NL 随精度要求上升的假说)。

### C14. Understanding and Mitigating Errors of LLM-Generated RTL Code — [已核实,arXiv API;TCAD 已接收]
- Zhang, Liu, Cheng, Li et al. arXiv 2508.05266 (2025)。https://arxiv.org/abs/2508.05266
- 摘要:"most failures arise not from deficient reasoning, but from a lack of RTL programming knowledge, insufficient circuit understanding, **ambiguous specifications**, or misinterpreted multimodal inputs";修正手段含 "design description rules with rule-checking to clarify inputs"。VerilogEval 98.1%。
- 关系:**支持**——"歧义规约"是可归因的失败来源。

### C15. ChipBench — [已核实,arXiv HTML 全文已读] ⚠️ 重要反向证据
- Yu, Zhou, Lin, Zhang et al. arXiv 2601.21448 (2026)。https://arxiv.org/abs/2601.21448
- 摘要:三类任务:Verilog 生成、调试、**参考模型生成**(Python / SystemC / CXXRTL,132 样本);"Claude-4.5-opus achieving only 30.74% on Verilog generation and **13.33% on Python reference model generation**"(注意:正文贡献列表处写的是 **15.93%**,与摘要不一致,引用时需注明)。
- 任务定义(§2):输入 `prompt.txt`(模块 NL 描述)+ `golden.sv`;HTE 从 golden.sv 抽 I/O 信号,随机激励比较 golden RTL 与 LLM 生成的参考模型。即:**NL → 参考模型,判定标准是与一个隐藏的 golden RTL 一致**。
- 原文观察:"for simple self-contained modules, LLMs achieve 5%–20% higher accuracy on Python reference model generation compared to Verilog generation, likely due to Python's prevalence in LLM pre-training data … This advantage further diminishes as module complexity increases";"while LLMs understand Python syntax, they lack hardware behavior modeling knowledge"。
- 原文(引言,作者主张,未独立核实):"In industrial workflows, reference model generation is even more resource-intensive than Verilog design, reflected in a 1:1 - 5:1 ratio of verification engineers …"
- 关系:**挑战**"LLM 写周期精确模型很容易";但其任务方向是"从 NL 复现一个隐藏 RTL 的行为"(模型追 RTL),与本文主张的"模型即定义、RTL 追模型"方向相反——这是写作中必须正面处理的点。

### C16. FormalRTL — [已核实,arXiv API]
- Li, Li, Wen, Zhao et al. *FormalRTL: Verified RTL Synthesis at Scale.* arXiv 2603.08738 (2026)。https://arxiv.org/abs/2603.08738
- 摘要:"substantial barriers remain for industrial-scale, datapath-centric designs due to ambiguous specifications and a lack of formal correctness guarantees … systematically integrates **software reference models as formal, executable specifications** to guide RTL code generation and verification … formal equivalence checking"。
- 关系:**强支持**(数据通路场景下"参考模型作为规约"的直接实例;周期级控制密集场景未覆盖)。

### C17. SpecLLM — [已核实,arXiv API]
- Li, Fang, Zhang, Xie. arXiv 2401.13266 (2024)。https://arxiv.org/abs/2401.13266
- 摘要:用 LLM 生成与审阅架构规约;"Mistakes in these specifications may significantly affect subsequent stages of chip design."
- 关系:**提供对照**(仍在 NL 规约范式内改进)。

### C18. SpecRead — [已核实,arXiv API] (单作者、2026-09 预印本,证据强度一般)
- Feilian Huang. arXiv 2609.33699。https://arxiv.org/abs/2609.33699
- 摘要:把"读懂规约"与"写出代码"分离评测(OpenTitan 10 个 IP,385 题);以 Ministral-3B 初步表征,"51.2% false-positive rate on distractors and 100% on consistent-RTL controls"。
- 关系:**部分支持**——"失败是因为误读规约还是不会写"本身就是歧义;可选用,非必需。

### C19. LLM-enabled BDD Workflow for Formally Verified Hardware — [已核实,arXiv API]
- Müller, Liu, Drechsler. arXiv 2609.15318 (2026)。https://arxiv.org/abs/2609.15318
- 摘要:"Specifications … suffer from ambiguity when written in natural language … Formal specifications mitigate these ambiguities, but they come with their own challenges … Controlled Natural Language (CNL) … middle-ground";RTL 功能正确性提升 2.48×。
- 关系:**部分支持**(降 α 的另一条路线:受控自然语言)。

### C20. ClarifyGPT — [已核实,arXiv API]
- Mu, Shi, Wang, Yu et al. arXiv 2310.10996 (2023)。https://arxiv.org/abs/2310.10996
- 摘要:"detects whether a given requirement is ambiguous by performing a code consistency check. If it is ambiguous … generate targeted clarifying questions."
- 关系:**支持**——"通过多个候选实现的不一致来检测歧义"与 CIRCT UNCLEAR 门(枚举多种合理解法)同构,是软件领域的先例。

### C21. Design Conductor(Verkor)— [已核实,arXiv HTML 全文已读] ⚠️ 最强"端到端"对照
- The Verkor Team, Ravi Krishna, Suresh Krishna, David Chin. *Design Conductor: An agent autonomously builds a 1.5 GHz Linux-capable RISC-V CPU.* arXiv 2603.08716 (2026-02)。https://arxiv.org/abs/2603.08716
- 摘要:"In 12 hours and fully autonomously … meet timing at 1.48 GHz (rv32i-zmmul; using the ASAP7 PDK), starting from a **219-word requirements document**";CoreMark 3261。
- **输入规约原文要点**:"5-stage pipelined design, in-order, single-issue … You need to achieve a CPI <= 1.5 … Testing: You have access to Spike, the RISC-V ISA simulator. Use this to build a **cycle-by-cycle integration test** and verify that the behavior of your module matches that of Spike." 还提供了 RISC-V ISA/ASM 手册。
- 方法:"DC uses Spike to construct an overall vercore_tb.v testbench which … confirm the design's architectural state and memory transactions match those reported by Spike … It is this verification-driven approach that allows DC to arrive at a working design."
- **§5.3 Specification Requirements(关键)**:"We found that the input specification provided to DC has to be written in an extremely deliberate, tight, and **verifiable/measurable** manner. Without the CPI requirement in that document, for example, DC would sometimes generate a processor with significantly worse performance on branches and forwarding. With that line in the spec, DC would use a cycle counter in its testbench to compute its cycles per PC reported in the Spike trace to estimate CPI."
- §5.1:"we believe it will remain critical to have experienced human architects guide systems such as DC";forwarding 初始实现导致过长关键路径,"Only after the model observed timing results did it grasp the issue"。
- §5.2:"models reason about Verilog, which is an event-driven language, as if it were sequential code … We posit that this is due to the heavy presence of software code in both LLM pre-training and post-training"。
- §6.2:"One likely process change will be to **front-load verification work**, such that DC can be provided with some kind of integration test to guide its RTL implementation."
- 关系:**挑战**(端到端可行);**同时支持**(成功依赖 Spike 这一可执行 oracle + 可测量的规约 + 教科书级设计带来的高 I(S;K))。

### C22. Design Conductor 2.0 — [已核实,arXiv HTML 全文已读]
- 同一团队。arXiv 2605.05170 (2026-05)。https://arxiv.org/abs/2605.05170
- 摘要:80 小时自主构建 TurboQuant 推理加速器 VerTQ(5129 个 FP16/32 单元,FPGA 125MHz,TSMC 16FF 5.7mm²),"handle 80x larger tasks"。
- 输入包含 "the TurboQuant references and **an existing software implementation**";§2.1.3 "Conductor 2.0 wrote extensive testbenches for each module, using **reference Python implementations** to debug functionality and **cycle-by-cycle behavior**";AES 案例输入 "Use OpenSSL as the golden reference"。
- §4.1 token 分布:Architecture **6.3%**,Low-level modules 28.2%,High-level modules 32.6%,Final integration 32.9%;"Not unlike a human team, verification and achieving timing closure are the most expensive components."
- §4.2.4:"human review is the major bottleneck. The only 'solution' we have found to this is for the human designers to think through the key design constraints (PPA) upfront."
- §1 脚注:业界有人认为开源 RISC-V 核可能在预训练数据中,作者称 "a specious point, but we acknowledge it"。
- 关系:**支持**(参考实现 + 逐周期对比 = 本文桥接的实践形态;PPA 约束需前置显式给出 = 本文的侧信道 C)。

### C23. HLSPilot — [已核实,arXiv API]
- Xiong, Liu, Li, Li. arXiv 2408.06810 (2024)。https://arxiv.org/abs/2408.06810
- 摘要:"RTL code generation with natural language … remains error-prone and limited to relatively small modules because of the **substantial semantic gap between natural language expressions and hardware design intent**. … reduces the semantic gaps by utilizing C/C++ for generating hardware designs via HLS".
- 关系:**支持**(α_NL 的直接文字表述)。

### C24. C2HLSC — [已核实,arXiv API;TODAES 2025]
- Collini, Garg, Karri. arXiv 2406.09233(LAD workshop 版)/ 2412.00214(TODAES 版)。https://arxiv.org/abs/2412.00214
- 摘要:LLM 把普通 C 重构为 HLS 可综合 C,"guided by the system prompt and tool's feedback";分层 divide-and-conquer。
- 关系:**提供对照**(代码→代码的桥接;软件语义到硬件语义仍有鸿沟)。

### C25. Agent Factories for HLS — [已核实,arXiv API;ICLAD 2026]
- Bhandwaldar, Choudhury, Puri, Srivastava. arXiv 2603.25719。https://arxiv.org/abs/2603.25719
- 摘要:通用编码 agent 从高层算法规约做 HLS 优化;子核分解 + ILP 组合 + 专家 agent 跨函数优化。
- 关系:**提供对照**(从可执行高层规约出发的硬件优化)。

### C26. HSCO-Bench — [已核实,arXiv API]
- Tsai, Chiu, Baisi, Chen et al.(Carloni 组)arXiv 2605.19399 (2026)。https://arxiv.org/abs/2605.19399
- 摘要:端到端 HW/SW 协同设计 benchmark;"Among the five frontier models evaluated, only two of them could successfully generate valid SoC prototypes."
- 关系:**支持**"端到端仍难"。

---

## D. 敏捷硬件范式 / 中间表示 / 统一抽象

### D1. Assassyn — [已核实,作者 PDF 全文]
- Jian Weng, Boyang Han, …, Lisa Wu Wills, Marco Canini. *Assassyn: A Unified Abstraction for Architectural Simulation and Implementation.* ISCA 2025. DOI 10.1145/3695053.3731004。PDF:https://sands.kaust.edu.sa/papers/assassyn.isca25.pdf
- 摘要:"performance modeling based on simulation is typically developed alongside hardware implementation, allowing the exploration of high-level design decisions before dealing with the error-prone, low-level RTL details. However, this approach also introduces new challenges in coordinating across multiple teams to align implementation details separate codebases";"All the generated simulators perfectly align with the generated RTL behavior, while achieving 2.2-8.1× simulation speedup, and requiring 70% lines of code"。
- 正文细节:"our Assassyn-generated cycle-accurate simulator implemented in Rust is **2.2× faster than the Verilator-generated simulator** on CPU simulation, and has 8.1× speedup over HLS ASIC simulation";"for longer-running workloads like qsort and rsort, **gem5 achieves an order of magnitude speedup** once this overhead is amortized. Gem5 excels in raw simulation speed, but our Assassyn generated simulators offer a unique cycle-exact correspondence"。
- 关系:**支持**(业界/学界承认"性能模型先行 + 与 RTL 对齐"的痛点;单源可消除对齐);**挑战/对照**(单源路线让"桥"退化为同一份代码;同时显示与 RTL 逐周期对应的模拟器速度只比 Verilator 快约 2×,远慢于 gem5 —— **保真度与探索速度存在真实张力**)。

### D2. PyMTL — [已核实,作者 PDF / DOI]
- Lockhart, Zibrat, Batten. *PyMTL: A Unified Framework for Vertically Integrated Computer Architecture Research.* MICRO 2014, pp. 280–292. DOI 10.1109/MICRO.2014.50。https://www.csl.cornell.edu/~cbatten/pdfs/lockhart-pymtl-micro2014.pdf
- 摘要:"a significant challenge due to the unique languages, design patterns, and tools used in **functional-level (FL), cycle-level (CL), and register-transfer-level (RTL)** modeling … unified design environment for FL, CL, and RTL modeling"。
- 关系:**支持**——"FL / CL / RTL"三层是成熟的方法论分层,本文的"周期精确模型"对应 CL 层;可借用其术语厘清"周期精确模型 vs RTL"。

### D3. Kôika("The Essence of Bluespec")— [已核实,DOI/ACM/作者 PDF]
- Bourgeat, Pit-Claudel, Chlipala, Arvind. PLDI 2020. DOI 10.1145/3385412.3385965。
- 摘要:"gives direct control over the scheduling decisions that determine performance … deterministic operational semantics … Coq definitions … and a verified compiler to circuits";贡献含 "A cycle-accurate operational semantics that does not depend upon any static analysis"。
- 关系:**支持**——"周期级语义可以被形式化并带有可验证的到电路编译",是"受限建模语言提升可实现性"的方向证据。

### D4. Transaction Level Modeling: An Overview — [已核实,ACM/作者 PDF]
- Lukai Cai, Daniel Gajski. CODES+ISSS 2003, pp. 19–24. DOI 10.1145/944645.944651。
- 摘要:"the transaction-level models (TLMs) are not well defined and the usage of TLMs in … modeling, validation, refinement, exploration, and synthesis, is not well coordinated";以 untimed / approximate-timed / cycle-timed 计算与通信构成的系统建模图定义四类 TLM(含 cycle-accurate computation model)。
- 关系:**提供对照/术语来源**——"在哪个抽象层做探索、在哪个抽象层做精化"是 2000 年代 ESL 的老问题;本文的新意需放在"agent 时代成本结构变化"而不是"发明分层"。

### D5. Keutzer et al., Orthogonalization of Concerns — [已核实,DOI/作者主页]
- Keutzer, Malik, Newton, Rabaey, Sangiovanni-Vincentelli. *System-level design: orthogonalization of concerns and platform-based design.* IEEE TCAD 19(12):1523–1543, 2000. DOI 10.1109/43.898830。
- 摘要:"we focus on a method that separates parts of the design process and makes them nearly independent so that complexity could be mastered."
- 关系:**支持(历史先例)**——"把设计过程拆成近乎独立的部分"是系统级设计方法论的经典主张。

---

## E. 周期级模型的可信度 / 对齐 / 协同仿真

### E1. Sources of Error in Full-System Simulation — [已核实,作者 PDF 全文]
- Gutierrez, Pusdesris, Dreslinski, Mudge, Sudanthi, Emmons, Hayenga, Paver. ISPASS 2014. DOI 10.1109/ISPASS.2014.6844457。PDF:http://www.eecs.umich.edu/cse/awards/pdfs/ispass_2014-1.pdf
- 摘要:gem5 对照 ARM Versatile Express TC2(Cortex-A15)硬件;修改后 SPEC CPU2006 平均运行时间误差 5%、MAPE 13%;PARSEC MAPE 16%/17%;多数微架构统计平均误差 20% 以内。
- 正文:沿用 Black & Shen 的误差分类——"1) modeling errors occur when the simulator developer codes the desired functionality erroneously; 2) **specification errors occur when the developer is uninformed or must speculate about the behavior of the functionality being modeled**; 3) abstraction errors occur when the developer abstracts away, or fails to implement, certain details";结论:"**Of the three sources of simulation error described earlier, specification error is the dominant source in this work.**" 以及 "Specification and abstraction errors do not imply inaccurate simulation"。
- 关系:**强支持(关键洞察来源)**——模型追硬件时主导误差是"规约误差"(不知道硬件到底怎么做);若模型就是规约、RTL 追模型,规约误差在定义上不存在。

### E2. Architectural Simulators Considered Harmful — [已核实,作者 PDF / DOI]
- Nowatzki, Menon, Ho, Sankaralingam. IEEE Micro 35(6):4–12, 2015. DOI 10.1109/MM.2015.74。
- 原文定义:"we define simulator as a detailed software modeling tool that provides cycle-level performance, area, power, and/or energy estimates **without relying on a register transfer level (RTL) specification** of the machine it models";三类陷阱:关键现象建模层级错误("simulators act like first-order models, are labeled as detailed simulation, and therefore have the benefit of neither")、当黑盒使用、损害评估标准。
- 关系:**挑战**(周期级模拟器常被高估)且**支持**(问题根源在于模拟器与 RTL 规约脱钩 → 让模型成为 RTL 的规约,即可把模拟器重新绑回 RTL)。

### E3. Calibration of Microprocessor Performance Models — [已核实,IEEE 摘要 + 课程镜像 PDF 片段]
- Bryan Black, John Paul Shen. IEEE Computer 31(5):59–65, 1998. DOI 10.1109/2.675637。
- 摘要:"Experimental results in the article highlight the significant effort required to achieve a truly cycle-accurate performance model and the necessity for systematic validation of performance models";正文:"We propose a systematic method for calibrating a performance model against a register transfer level (RTL) implementation";未校准模型 "can mislead designers into implementing features that do not actually improve performance"。
- 关系:**挑战**(周期精确模型很贵、易误导)+ **提供历史对照**(1998 年已在做"模型对 RTL 校准")。

### E4. XiangShan / MinJie(MICRO 2022)— [已核实,作者 PDF 全文]
- Xu, Yu, Tang et al. *Towards Developing High Performance RISC-V Processors Using Agile Methodology.* MICRO 2022. DOI 10.1109/MICRO56248.2022.00080。PDF:https://talks-pubs.xiangshan.cc/publications/micro2022-xiangshan.pdf
- 关键摘录:"a roughly XIANGSHAN-parameter-aligned GEM5 model achieves ∼7/GHz on SPEC CPU2006 at 2GHz, which is **∼30% less than XIANGSHAN** … For better accuracy, **companies have to maintain independent teams for architectural simulators to align with the micro-architecture of the real RTL model, which requires a huge investment of human resources.** Performance evaluation on RTL models relies on cycle accurate RTL-simulation, with a typical simulation speed of KHz on software and MHz on FPGAs. However, it still takes more than 150 hours for XIANGSHAN to finish SPEC CPU2006 at 50MHz on a single FPGA, resulting in a one week iteration cycle".
- DiffTest / DRAV:RTL 与参考模型(NEMU,类 Spike 的 ISA 解释器)协同仿真;"DRAV relaxes the strict equivalence of testing outcomes to checking specific diff-rules, forming an **N-to-1 correspondence between the designs-under-test (DUTs) and reference model (REF)**. The key insight is that **different outcomes under diverse micro architectures may be legal under the design specifications**."
- 关系:**强支持**(工业级处理器用"可执行参考模型作为 oracle";参数对齐远远不够;对齐需要独立团队)+ **重要修正**(oracle 必须显式建模"合法差异",逐周期严格相等不是正确目标)。

### E5. ISA-Formal(Arm)— [已核实,作者 PDF / 主页]
- Reid, Chen, Deligiannis, Gilday, Hoyes, Keen, Pathirane, Shepherd, Vrabel, Zaidi. *End-to-End Verification of ARM Processors with ISA-Formal.* CAV 2016, LNCS 9780, pp. 42–58。https://alastairreid.github.io/papers/CAV_16/
- 摘要:"A key part of making the approach scale is use of a mechanical translation of ARM's Architecture Reference Manuals to Verilog allowing the use of commercial model-checkers … applied this method to 8 different ARM processors … found bugs that would have been hard for conventional simulation-based verification to find";ASL 规约 "is all executable and has been tested very thoroughly using the same architecture conformance tests"。
- 关系:**强支持**——Arm 把"可执行架构规约"作为 RTL 验证的 oracle;且规约本身先被测试("who guards the guards")。注意:这是 ISA 层,不是周期层。

### E6. FireSim — [已核实,作者 PDF / DOI]
- Karandikar, Mao, Kim, Biancolin, …, Asanović. ISCA 2018, pp. 29–42. DOI 10.1109/ISCA.2018.00014。
- 摘要:"cycle-exact microarchitectural simulation … FPGA-accelerated simulation of silicon-proven RTL designs";1024 节点 3.4MHz(< 1000× 慢于实时)。
- 关系:**提供对照**——"RTL 本身作为评估器"的加速路线;与"桥接模型"并非互斥(CHIA 用 FireSim 做最终 SPEC 验证)。

---

## F. 仅部分核实或未核实(不进入可引用清单)

| 条目 | 状态 | 说明 |
|---|---|---|
| Chipyard (Amid et al., IEEE Micro 2020) | 部分核实 | 仅见于 CHIA 参考文献;未读原文。可作为背景提及,不作论据。 |
| CIRCT / "MLIR as hardware compiler infrastructure"(Eldridge et al., WOSET 2021) | 部分核实 | 同上。 |
| Microbench(Nowatzki 2015) | 部分核实 | CHIA 参考文献,gem5 对齐训练集来源。 |
| Hung & Narasimhan 2004, "Reference model based RTL verification" (HLDVT) | 部分核实 | 仅见于 ChipBench 参考文献。 |
| Calyx(ASPLOS 2021)、Chisel(DAC 2012)、Bluespec 原始文献 | 未核实 | 本次未检索;如需引用须补核。 |
| OpenEvolve / SkyDiscover / AdaEvolve 的具体结论 | 未核实 | 仅在 CHIA 文档/参考文献中出现名称,未读其论文。 |
| gem5 x86 validation(Akram & Sawalha 等) | 未核实 | 未检索到可确认的条目,不引用。 |
| "LLM 自动生成 gem5/周期级模拟器"的专门工作 | 未找到 | arXiv 检索未发现直接对应的论文;CHIA gem5_align 可能是最接近的公开结果。**可作为"空白"写进文章,但措辞需为"笔者检索范围内未见"。** |
| refinement mapping(Abadi & Lamport)等形式化精化理论 | 未核实 | 若在文中提"精化/抽象函数",只作概念用语,不挂引用。 |
| Sail RISC-V 形式化模型、AWS TLA+ 实践 | 未核实 | 本次未检索。 |

---

## G. 调研中形成的、对写作最关键的观察(供 01-plan 使用)

1. **方向性是 gem5 对齐反例的解药**:Gutierrez 2014 指出模型追硬件时规约误差占主导;XiangShan 指出参数对齐差 30%、需独立团队;CHIA gem5_align 也是"模型追 RTL"方向且出现过拟合(kSuppressThreshold)。而 Spike/NEMU/ASL 这类"RTL 追可执行规约"的做法在 ISA 层已是工业标准。→ 本文主张本质上是把 ISA 层已验证有效的方向推广到周期层。
2. **端到端成功案例其实也依赖可执行 oracle**:Design Conductor 1.0 依赖 Spike 逐周期 testbench + 可测量 CPI 要求;2.0 依赖参考 Python 实现做逐周期调试 + 前置 PPA 约束。
3. **ChipBench 是必须正面回应的反证**:NL→参考模型(对一个隐藏 RTL)只有 13–16%。回应要点:这是"复现他人 S_e"的方向,恰恰证明 NL 通道的残余熵大;本文主张中 M 不需要复现隐藏目标,它由探索选择产生、本身即定义。但"LLM 写出可实现、无作弊的周期模型"仍是开放难题(ArchAgent simulator escape)。
4. **保真度—速度张力是真实的**:Assassyn 逐周期对齐的模拟器仅比 Verilator 快 ~2.2×,gem5 长负载快一个数量级;ArchAgent ChampSim 单次 >12h;XiangShan FPGA 跑 SPEC 150h。→ 需要多保真度级联:粗模型广搜,入围者升级为 RTL-ready 周期模型。
5. **物理耦合有两面**:timing_opt 中"保持 IPC 不变"的改动 = 固定 M 只改 R,前 4 轮即平台化;切到 Iron Law 才获得 2.03× 频率(以改变周期行为为代价)。16nm 不泛化说明物理信息强依赖工艺,不宜固化进 M。→ M 应保持工艺无关,但需要来自物理层的"周期内逻辑深度预算"之类的约束反馈。
