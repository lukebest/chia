## 5. 最强的两个反例：gem5 对齐与 Iron Law

CHIA [1] 的五个案例里，有两个正面挑战了本文假设：耗时 10.5 天的 `gem5_align` 与揭示频率—IPC 耦合的时序优化。摊开它们、看清论点收窄到了哪里，比单边辩护更有价值。

### 5.1 gem5 对齐为什么贵——方向反了

第一个质疑：**在 CHIA 的 `gem5_align` 案例里，光让 gem5 对齐现成的 MediumBOOM 就让 agent 跑了 10.5 天，这难道不直接推翻“让架构探索交付周期精确模型”的可行性吗？**

我承认周期级模型与 RTL 的对齐极贵 [20]：香山（XiangShan）团队报告，粗对齐的 gem5 在 SPEC CPU2006 上比实际 RTL 低约 30%，企业不得不设独立团队对齐 RTL [15]；据 CHIA 论文，agent 在 2-wide MediumBOOM 上历经 202 次迭代（约 10.5 天），虽在 36 个 Microbench 训练基准上将平均周期误差压到 $<3\%$，但留出集 Embench IOT 2.0 最佳迭代平均误差仍在 $<7\%$ 且有离群点，还出现针对训练基准调参的阈值 `kSuppressThreshold` 令留出集 Wikisort 恶化的过拟合 [1]。

症结在于：`gem5_align` 之所以这么贵、甚至过拟合，是因为方向反了。**让模型去追 RTL 是逆向工程，误差主要来自“不知道硬件到底怎么做”；让 RTL 去追模型是精化检查——这在 ISA 层早已是工业标准，现在该把它推到周期层了。** `gem5_align` 先有写死的复杂乱序核 RTL，再让模拟器去猜其周期行为；Gutierrez 等人在用 gem5 对照 Cortex-A15 真实硬件的研究中指出，三类误差里**规约误差（specification error，即建模者不了解或必须猜测目标硬件的具体行为）是主导误差源** [14]。而在本文流程里，周期模型 $M$ 先于 RTL 存在且**本身就是规约**：Gutierrez 意义上的规约误差消失了，但转化成了 $M$ 自身是否正确、是否可实现的问题（见第 6 节）；对 RTL 阶段而言，问题变成验证 $R \models M$。回顾验证史，ISA 层早已走通此路（CHIA `riscv_extensions` 用 Spike 做逐指令 lockstep [1]、香山 DiffTest 比对可执行模型 NEMU [15]、Arm 将可执行 ASL 规约机械翻译为 Verilog 落地 8 款处理器的 ISA-Formal [16]）。

据此我把论点收窄为：**本文主张把“RTL 追可执行规约”从 ISA 层推到周期层；至于周期层的“RTL 追模型”是否同样显著比“模型追 RTL”便宜，是我的猜测（目前只有误差源归因与 ISA 层先例作旁证），留待第 8 节 P2 检验。** 同时，若把 `gem5_align` 倒过来跑，它用 O3PipeView 流水线 trace 按 PC 对齐比对 gem5 退休周期与 Verilator 提交周期、配合 BOOM TMA 计数器对照 `stats.txt` 的诊断信号 [1]，恰好就是 RTL 阶段做 $M \leftrightarrow R$ 性能对齐的现成工具链；而且单个 agent 用约 10.5 天、约 \$2,356 API 成本（据 CHIA 论文 [1]）就基本做成了过去“对大团队也是巨大工作量” [1] 的事，说明双向对齐成本都在快速下降。

### 5.2 Iron Law 耦合

第二个反例直击第 1 节的伏笔：**处理器性能受铁律（Iron Law，$\text{Time} = \text{Instructions} \times \text{CPI} \times \text{Cycle Time}$）支配，周期模型 $M$ 不含门级延迟与面积，脱离物理实现探索难道不会失真吗？**

耦合确实存在。据 CHIA 论文，时序优化案例在前 4 次“IPC-neutral（不影响 IPC）”迭代后陷入平台化，团队随即切换到 Iron Law 模式（牺牲少量 IPC 换取关键路径缩短），最佳迭代（第 7 次）频率相对基线约为 $2.03\times$（约 47 MHz $\to$ 95 MHz）[1]；论文没有拆分两阶段各自的贡献，但能确定：**IPC-neutral 走不动之后，继续前进需要改变 $M$**。同样，ArchAgent [2] 承认因缺乏频率与面积数据，确认可实现性高度依赖人工（从微架构模型推断物理存储大小并不平凡）；且据 CHIA 论文，Sky130 上的改进放到商用 16nm 下，仅第一项（发射队列从线性深度改为对数深度，改善关键路径约 $70\%$）泛化，其余均不泛化 [1]。

为此，我将两阶段解耦收窄为三条工程边界：

1. **探索中引入排序正确的 PPA 代理（$\hat f$ 与面积模型）**：在 $M$ 中显式约束“每周期逻辑预算”并配误差有界的 $\hat f$，选出模型的性能损失比即有严格上界（见附引理）。Agentic Architect [3] 结合周期精确模拟与硬件成本模型，在面积/功耗预算下以 TAGE-SC-L $10.58\%$ 的面积达其 $97.62\%$ 的 IPC，正是实例。
2. **桥是双向外环（见图 1）**：综合暴露关键路径无法在固定 $M$ 下收敛时，物理瓶颈经 $C^+$ 回流修改 $M$。
3. **工艺信息不固化进 $M$，而作可替换的 $\hat f$ 与 $C$**：16nm 不泛化说明 $M$ 应保持工艺无关，换工艺只替换 $\hat f$ 与 $C$。

> **附：代理引理（可跳过）**
> 设 $\mathrm{Perf}(M,R)=\mathrm{IPC}(M)\cdot f(R)$，$f^*(M)=\max_{R\in\mathrm{Refine}(M)}f(R)$。若频率代理 $\hat f$ 在候选集上满足 $\ln\hat f(M)=\ln f^*(M)+b+e(M)$（常数偏置 $b$ 任意，$|e(M)|\le\varepsilon$），则按 $\mathrm{IPC}(M)\cdot\hat f(M)$ 选出的 $\hat M$ 相对真实最优 $M^*$ 满足：
> $$\frac{\mathrm{IPC}(M^*)f^*(M^*)}{\mathrm{IPC}(\hat M)f^*(\hat M)}\le e^{2\varepsilon}.$$
> **证明**：由条件有 $f^*(\hat M)\ge\hat f(\hat M)e^{-b-\varepsilon}$、$\hat f(\hat M)\mathrm{IPC}(\hat M)\ge\hat f(M^*)\mathrm{IPC}(M^*)$ 及 $\hat f(M^*)\ge f^*(M^*)e^{b-\varepsilon}$，三步串联消去 $b$ 即得：$\mathrm{IPC}(\hat M)f^*(\hat M)\ge \mathrm{IPC}(\hat M)\hat f(\hat M)e^{-b-\varepsilon}\ge \mathrm{IPC}(M^*)\hat f(M^*)e^{-b-\varepsilon}\ge \mathrm{IPC}(M^*)f^*(M^*)e^{-2\varepsilon}$。这意味着探索无需预测绝对主频，只需一个**在常数倍意义下误差有界**（$\varepsilon$ 小，$b$ 无所谓）的相对代理。

不过，代理引理是条件性的——它说“若有好代理则够用”，并未保证好代理存在；16nm 不泛化说明 $\varepsilon$ 可能很大，若校准 $\hat f$ 必须综合 RTL，综合又回到了内环（见第 8 节下一步中的频率代理校准实验）。反过来，IPC-neutral 模式与 `timing_experiment` 子模块 A/B 综合 [1] 也支持了拆分：固定 $M$、在 $\mathrm{Refine}(M)$ 内做垂直物理优化，本身就是独立子问题（见图 3）。

```text
  最高频率 f*(M)
    ^
    |        . '              . '
    |     . '  [M2: Iron Law 模式]
    |  . '        *  <-- 斜向移动: 改变 M (微降 IPC, 最终达 ~2.03x 基线频率)
    |              \
    |               \  . '
    |                * '  <-- [M1: IPC-neutral 平台期]
    |                ^
    |                |  垂直移动: 固定 M, 在 Refine(M) 内优化 RTL
    |                *  <-- [M1: 初始 RTL]  (虚线: 等性能曲线 IPC·f* = const)
    +---------------------------------------------------------> IPC(M)
```

*图 3：在 $\mathrm{IPC}(M)$–$f^*(M)$ 平面上看时序优化：固定 $M$ 的垂直优化与改变 $M$ 的斜向跃迁。*

---

## 6. 其他几个值得认真对待的质疑

除上述两例，另有七个质疑值得按“承认—收窄—证据”回应。

### 6.1 为什么不直接 NL→RTL？
**承认与收窄**：Design Conductor [4, 5] 展示了端到端潜力，但如第 4 节不等式所示，直接路线仅在教科书级流水线上划算，对新颖微架构必然亏本。**证据**：离开教科书区间后，CVDP [7] 上 SOTA pass@1 $\le 34\%$，ChipBench [6] 上 Claude-4.5-opus 的 Verilog 生成仅 $30.74\%$，错误归因研究 [13] 亦指规约歧义为主因；故 Spec2RTL-Agent [8] 先生成可综合 C++（人工干预减至多 $75\%$），FormalRTL [9] 用软件参考模型作可执行规约。

### 6.2 LLM 写周期模型本身也会幻觉
**承认**：ChipBench [6] 显示 Claude-4.5-opus 生成 Python 参考模型 pass@1 仅 $13.33\%$（摘要口径；贡献列表作 $15.93\%$，结果表中 $15.93\%$ 对应 Claude 4.5 Sonnet），低于其 Verilog 的 $30.74\%$；且我检索范围内未见专门做 LLM 自动生成 gem5/周期级模拟器的工作。$M$ 亦有功能与“不可实现/作弊”幻觉，如 ArchAgent [2] 的 simulator escape（被优化消除的 assertion 令旁路写消失，虚增 $3\%\text{–}4\%$ 性能）。**收窄**：ChipBench 的任务方向是“从自然语言追隐藏 golden”，低分既可能来自能力不足（作者自己的归因：缺乏硬件行为建模知识），也可能来自自然语言歧义，二者无法从其数据中分离；本文中 $M$ 由探索选出、本身即定义。**证据**：对执行驱动的核级 $M$，功能幻觉可用 Spike 等 ISA 级 oracle 自检（trace 驱动的预取器/替换策略模型没有功能语义，ArchAgent 原话“no notion of correct computation” [2]）；不可实现与作弊靠受限建模语言/接口（Kôika [19]、Assassyn [17]）配规则化 verifier（如代码行数上限 [2]），并以 **RTL 阶段作最终可实现性检查**（$\mathrm{Refine}(M)$ 为空则回流）。

### 6.3 这不就是业界现状吗？
**承认**：正交化关注点 [23]、TLM [22]、PyMTL 的 FL/CL/RTL [18] 与独立性能模型团队 [15] 早有先例。**收窄与证据**：本文新在两点——**角色与方向**：现状中模拟器按 Nowatzki 等定义是“不依赖 RTL 规约”的估算器 [21]，常滞后失准；本文中 $M$ 是前置契约，机械检查 $R \models M$。**成本结构**：agent 压低了精化与对齐成本，使 ISA 层做法得以推向周期层。

### 6.4 单源方案已让模拟器与 RTL 天然对齐，桥是否多余？
**承认**：Assassyn [17] 用统一抽象同时生成对齐的周期精确模拟器与 RTL，相当于 $c_{\text{ready}} \to 0$。**收窄与证据**：这反印证**桥就在周期级**。但探索受限于硬件 DSL：其模拟器速度贴近 Verilator，gem5 在长负载上再快一个数量级 [17]；且 DSL 语料远少于 C++/Python（旁证：Design Conductor 见模型把 Verilog 当顺序代码推理，归因于软件语料占主导 [4]）。故**拆的是“探索”与“精化”两个问题，未必是两份制品**。

### 6.5 逐周期严格一致会把 RTL 绑死，合法差异怎么办？
**承认**：死守逐拍严格相等会封死物理优化空间——IPC-neutral 模式的平台化即旁证 [1]。**收窄与证据**：香山 DRAV 指出“在设计规约下，不同微架构产生不同结果可能是合法的（different outcomes under diverse micro-architectures may be legal under the design specifications）”，故用差异规则建立 DUT 与 REF 的 $N$ 对 $1$ 对应 [15]。同理，本文 oracle 分为**功能层精确 + 性能层在规则/容差内一致**，显式建模 $\Delta$，并将 $H(\Delta)$ 与 $H(C^+)$ 计入第 4 节成本项。

### 6.6 方向翻转是否只是把逆向工程挪到了 $c_{\text{ready}}$ 这一步？
**承认**：情形 A 中探索产出 ChampSim 式 trace 驱动代码 [1, 2]，没有时序与功能语义；升级为 RTL-ready 周期模型必须**补写**原本不存在的周期行为，这本身是在猜“这个机制在真实流水线里会怎么表现”，规约误差 [14] 只是被转移而非消失。**收窄与证据**：第 5.1 节的方向论证只在情形 B（直接写 $M$）下干净；对情形 A，我把第 8 节 P3 设计成能检验 $c_{\text{ready}}$ 步骤的误差。

### 6.7 周期层的第一处分歧是否远不如 ISA 层有信息量？
**承认**：ISA lockstep 有效是因为提交时的架构状态有规范对齐点；周期层一旦允许 $\Delta$，两条轨迹在第一次合法偏移后就失去逐拍对齐，“第一处分歧”可能只是合法差异的连锁反应，不是 bug，$m_t^o$ 的信息量与 $H(\Delta)$ 此消彼长。**收窄与证据**：第 3.5 节的“付费者转移”强结论限定在功能层；性能层 oracle 只能做统计/计数器级定位（`gem5_align` 的周期误差百分比 + TMA [1] 正是这种形态）。

---
