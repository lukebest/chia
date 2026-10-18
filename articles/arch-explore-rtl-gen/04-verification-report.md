# 04 · 验证报告：《让 RTL 对模型精确》

> 验证对象：`03-article.md`（修订前已备份比对）。
> 一手来源：`docs/case-studies/*.rst`（5 篇全文）；`docs/信息论.pdf`（`pdftotext -layout` 全文）；CHIA 论文 arXiv 2606.27350 HTML 全文；arXiv 官方 API 元数据与摘要（13 篇 arXiv 文献全部取回）；Design Conductor / Design Conductor 2.0 / ArchAgent / ChipBench 的 arXiv HTML 全文；Assassyn、XiangShan（MICRO'22）、Gutierrez（ISPASS'14）作者 PDF 全文；Crossref API（9 个 DOI）；ISA-Formal 作者主页。
> 记号：✅ 一手来源逐字核实；⚠️ 基本正确但有措辞/范围问题，或只核实到元数据、内容依赖二手笔记；❌ 与一手来源不符（已在正文修复）。

**结论速览**：事实类核实 74 条，其中 ✅ 64 条、⚠️ 8 条、❌ 2 条（时序案例过度归因、DC“能启动 Linux”）。另发现数学/术语类 ❌ 3 处（数据处理不等式前提、$A_M$ 自相矛盾、“泛函层”误译）和 1 处无来源且自相矛盾的数字（表 A 的“MHz 级”）。以上全部已修复，正文共修复 25 处，没有残留的 ❌。最大的事实风险是对时序优化案例的过度归因（已修复）。最大的数学风险是数据处理不等式的前提表述，以及 $A_M = 0$ 与 $A_M^{\text{silent}} > 0$ 的自相矛盾（均已修复）。

---

## 1. 已核实清单

### 1.1 CHIA docs 口径

| # | 条目 | 来源 | 结论 |
|---|---|---|---|
| 1 | `gem5_align`：medium BOOM、10.5 天、平均周期误差 <3% | gem5-alignment.rst “Result” | ✅ |
| 2 | 对齐“即便对大团队也是巨大工作量，对小团队与学术界几乎不可能，通常不保持对齐或从未对齐” | gem5-alignment.rst Overview（“huge lift … nearly impossible … never aligned”） | ✅ 译文忠实 |
| 3 | O3PipeView 按 PC 比对 gem5 retire 与 Verilator commit；BOOM TMA 计数器对照 `stats.txt` | gem5-alignment.rst “The LLM in the loop” | ✅ |
| 4 | `riscv_extensions`：MegaBOOM、Bitmanip/标量密码/Zicond、~5.6% 与 ~3.5%、OpenSSL 最高 10×、无时序回退 | riscv-extensions.rst “Result” | ✅（“几何平均”与 SPEC06 口径来自论文表 2，见 #18） |
| 5 | “No expected output is ever hand-encoded — the golden model *is* the check” | riscv-extensions.rst 原文 | ✅ 逐字一致 |
| 6 | “a mismatch is caught the cycle it happens” / 分歧 trace 交给 debug LLM | riscv-extensions.rst | ✅ |
| 7 | riscv-dv 随机程序 lockstep 数百万条指令 | riscv-extensions.rst（“several million instructions”） | ✅ |
| 8 | directed test 由指令表生成，覆盖率按构造 100% | riscv-extensions.rst（“the table *is* the instruction set”） | ⚠️ 原文是**指令覆盖**，已改为“指令覆盖率” |
| 9 | “the flow verifies a BOOM implementation, it doesn't invent the ISA” | riscv-extensions.rst “Adding an extension” | ✅ 逐字一致 |
| 10 | timing_opt：15 次迭代、频率 >2×、IPC 损失 3.3%、净 1.97×（Sky130） | timing-optimization.rst “Result” | ✅ |
| 11 | 三种 prompt 变体；默认 IPC-neutral；`timing_experiment` 子模块 A/B 综合 | timing-optimization.rst | ✅ |
| 12 | CIRCT：16 个 issue、5 个可复现且方案明确、全部解决、3 个 PR 被上游接受 | circt-issue-solving.rst “Result” | ✅（另 2 个因 “good first issue” 未提交） |
| 13 | assess 枚举实质不同的合理解法，>1 且无依据即 `UNCLEAR`；#8508（“diagnose and refuse” vs “extend to support”） | circt-issue-solving.rst | ✅ |
| 14 | `repro.sh` “exit 0 iff fixed”；verify 无 LLM | circt-issue-solving.rst | ✅ |
| 15 | 架构发现：C++ prefetcher struct、编译、并行 trace、geomean IPC；构建失败得 0.0；三函数接口 | architectural-discovery-basic.rst | ✅ |
| 16 | 简化版：3 条 SPEC CPU 2017 trace、5 个 worker、10 次迭代、1 小时内 | architectural-discovery-basic.rst “Simplified example” | ✅ |

### 1.2 CHIA 论文独有口径（全部正确标注了“据 CHIA 论文”）

| # | 条目 | 来源（论文 §） | 结论 |
|---|---|---|---|
| 17 | 202 次迭代 ≈ 10.5 天；2-wide MediumBOOM；36 个 Microbench 训练基准 <3% | §5.1 | ✅ |
| 18 | 留出集 Embench IOT 2.0 最佳迭代平均 <7%，有离群 | §5.1 | ✅ |
| 19 | `kSuppressThreshold` 过拟合，同一迭代令 Wikisort 恶化 | §5.1 | ✅ |
| 20 | 1520 行 diff（不含注释与空行） | §5.1 | ✅ |
| 21 | $2,355.98 总成本（正文写约 $2,356） | §5.1 表 1 | ✅ |
| 22 | Bitmanip/Crypto/Zicond LLM 迭代 2/6/1 次 | §5.2 表 3 | ✅ |
| 23 | cosim 占总机时 50%–66% | §5.2 | ✅ |
| 24 | SPEC06 ref 25+ 万亿条指令，FireSim 中跑通 | §5.2（“all of them booting Linux and running in FireSim”） | ✅ |
| 25 | “the LLM knows nothing about this infrastructure, it only receives the necessary traces” | §5.2 | ✅ |
| 26 | 5.6% / 3.5% 为几何平均（表 2：1.056 / 1.035） | §5.2 表 2 | ✅（正文另有一处写 “5%”，文章统一用 5.6%，合理） |
| 27 | 时序：前 4 次 IPC-neutral 迭代后平台化，切换 Iron Law | §5.3 | ✅ 有 ❌ 引申，见 #30 |
| 28 | 约 47 MHz → 95 MHz、2.03×、最佳为第 7 次迭代；SPEC06 IPC 损失 3.28%（Verilator 3.20%） | §5.3 | ✅ |
| 29 | 16nm 下只有第一项优化（发射队列线性深度改为对数深度，关键路径约 70%）泛化 | §5.3 | ✅ |
| 30 | “最大收益恰恰来自改变 M”“切换后才拿到 2.03× 跃升” | §5.3 | ❌ **过度归因**。2.03× 是第 7 次迭代**相对基线的累计值**，论文没有拆分 IPC-neutral 与 Iron Law 两阶段各贡献多少，而且泛化到 16nm 的那项改进很可能出自 IPC-neutral 阶段。已改为“IPC-neutral 走不动之后，继续前进需要改变 M”（§5.2、表 C、图 3 三处） |
| 31 | ChampSim 无法提供功耗/面积/关键路径反馈 | §5.4.1 | ✅ |
| 32 | 原型中 assessor 与 fixer 对 #8508 的正确解法意见不一，之后才加“枚举所有解法”的规则 | §5.5 | ✅ |
| 33 | 4 个 issue 判为 Fix unclear（#4649 #5626 #7531 #8508） | §5.5 表 5 | ✅ |
| 34 | CHIA 作者 Cui, Hermida-Rivera, Toubes 等 | arXiv API | ✅ |

### 1.3 外部文献

| # | 条目 | 来源 | 结论 |
|---|---|---|---|
| 35 | DC：12 小时、219 词、1.48 GHz、ASAP7 | DC 摘要 / 表 1 | ✅ |
| 36 | DC：“能够启动 Linux” | DC 全文 | ❌ 正文从未报告启动 Linux，只在标题里称 “Linux-capable”（rv32i-zmmul）。已改为“被作者称为‘Linux-capable’” |
| 37 | DC §5.3：“extremely deliberate, tight, and verifiable/measurable”；没有 CPI 那一行就会出现分支/前递性能显著更差的处理器 | DC §5.3 | ✅ |
| 38 | DC 规约原文 “You need to achieve a CPI <= 1.5”“Aim for a clock rate of 1.6 GHz”“simple 5-stage pipelined design, in-order, single-issue” | DC §3.1（原始 HTML） | ✅（§4 中 $C_0$ 的例子“CPI ≤ 1.5、目标 1.6 GHz”也一并核实） |
| 39 | DC §6.2 “front-load verification work” | DC §6.2 | ✅ |
| 40 | DC 用 Spike 构建 `vercore_tb.v` 集成测试，规约要求 “cycle-by-cycle integration test” | DC §3.1–3.2 | ✅ |
| 41 | DC §5.2：模型把 Verilog 当顺序代码推理，归因于软件语料 | DC §5.2 | ✅（文章 §6.4 原先用 [4] 支撑“DSL 语料少”，属引申，已改为显式旁证表述） |
| 42 | DC 2.0：“80x larger tasks” | DC 2.0 摘要 | ✅ |
| 43 | DC 2.0：“using reference Python implementations to debug functionality and cycle-by-cycle behavior” | DC 2.0 §2.1.3 | ✅（主语是 agent，原文“团队引入”已改） |
| 44 | DC 2.0：人审是主要瓶颈；唯一办法是让人类设计者事先想清楚关键约束（PPA） | DC 2.0 §4.2.4 | ✅ 逐字一致 |
| 45 | ArchAgent：1B 指令单次 ChampSim 模拟可 >12 小时 | ArchAgent §3.5 | ✅ |
| 46 | ArchAgent：simulator escape（被优化掉的 assertion、被旁路的写凭空消失、表观领先 Mockingjay 3%/4%） | ArchAgent §7.3 | ✅ |
| 47 | ArchAgent：确认可实现性“remains a human intensive process”；“inferring physical storage size … is non-trivial” | ArchAgent §7.1 | ✅ |
| 48 | ArchAgent：基于规则的 verifier 统计代码行数 | ArchAgent §7.1 | ✅ |
| 49 | Agentic Architect：周期精确模拟 + 硬件成本模型、面积/功耗预算；分支预测器 97.62% IPC / 10.58% 面积 | arXiv 摘要 | ✅（作者顺序 Kypriotis 第一，文章正确） |
| 50 | ChipBench：Claude-4.5-opus Verilog 30.74%、Python 参考模型 13.33% | ChipBench 摘要 / 表 2 / 表 3 | ✅ |
| 51 | ChipBench：正文 15.93% | ChipBench §1 贡献列表 / 表 3 | ⚠️ **新发现**：表 3 中 Opus 的 pass@1 是 13.33%，15.93% 实际是 **Claude 4.5 Sonnet** 的 pass@1，贡献列表疑似把 Sonnet 的数写到了 Opus 名下。已在正文注明 |
| 52 | ChipBench 任务：prompt.txt + golden.sv，与隐藏 golden RTL 比对 | ChipBench §2 | ✅ |
| 53 | ChipBench：“understand Python syntax, they lack hardware behavior modeling knowledge” | ChipBench §5 | ✅ |
| 54 | 原稿“被测最强模型” | ChipBench | ⚠️ 模型名被模糊了；在 Python 参考模型任务上 Opus 也不是最强（Sonnet 15.93%、GPT-5 pass@5 18.15% 更高）。已恢复为 Claude-4.5-opus（§6.1、§6.2） |
| 55 | CVDP：783 题、13 类、SOTA 代码生成 pass@1 ≤ 34% | arXiv 摘要 | ✅ |
| 56 | Spec2RTL-Agent：先生成可综合 C++ 再走 HLS；人工干预减少至多 75% | arXiv 摘要 | ✅ |
| 57 | FormalRTL：软件参考模型作为形式化可执行规约 + 等价检查 | arXiv 摘要 | ✅ |
| 58 | LLM-FSM：YAML 转成 NL 规约；FSM 复杂度上升时最强 LLM 精度骤降 | arXiv 摘要 | ✅ |
| 59 | HLSPilot：“substantial semantic gap …”；用 C/C++ + HLS | arXiv 摘要 | ✅ |
| 60 | ClarifyGPT：代码一致性检查检测歧义并生成澄清问题 | arXiv 摘要 | ✅ |
| 61 | Zhang 等：失败源含 “ambiguous specifications”；TCAD | arXiv 摘要 / comment（“Accepted by IEEE TCAD”） | ✅ |
| 62 | XiangShan：参数粗对齐的 gem5 比 XiangShan 低约 30%（SPEC06）；“independent teams … huge investment of human resources” | MICRO'22 PDF §II | ✅ |
| 63 | XiangShan：软件 RTL 仿真 KHz 级；单 FPGA 50 MHz 跑完 SPEC06 >150 小时；一周迭代 | MICRO'22 PDF §II | ✅ |
| 64 | DRAV 引语 “different outcomes under diverse micro-architectures may be legal under the design specifications”；N 对 1 | MICRO'22 PDF §I | ✅（原文是 “micro-architectures”，有连字符，可选修正） |
| 65 | DiffTest + NEMU 参考模型 | MICRO'22 PDF 图 4 | ✅ |
| 66 | Gutierrez：规约误差定义；“specification error is the dominant source **in this work**” | ISPASS'14 PDF | ⚠️ 结论仅限其 gem5 对 Cortex-A15 **真实硬件**（非 RTL）的研究；原稿写成了普遍结论，已补上适用范围 |
| 67 | Assassyn：“perfectly align”；比 Verilator 快 2.2×；gem5 在 qsort/rsort 等长负载上快一个数量级 | ISCA'25 PDF | ✅ |
| 68 | ISA-Formal：ARM 规约机械翻译为 Verilog；8 款处理器 | 作者主页摘要 | ✅ |
| 69 | Black & Shen 1998：校准“需要巨大努力”，未校准模型会误导 | Crossref 仅确认元数据；内容依赖 02 笔记 | ⚠️ 未读一手全文 |
| 70 | Nowatzki 2015：模拟器定义为“不依赖 RTL 规约” | Crossref 仅确认元数据；内容依赖 02 笔记 | ⚠️ 未读一手全文 |
| 71 | PyMTL FL/CL/RTL；Kôika 周期级语义 + 可验证编译；Cai–Gajski TLM 分类；Keutzer 正交化 | Crossref 元数据 ✅；内容依赖 02 笔记 | ⚠️ 未读一手全文 |
| 72 | 信息论 PDF：定义 1.1–1.3、定理 2.1、命题 3.1、定理 4.1、推论 4.2、命题 5.1、注记 5.1、策略表 | `docs/信息论.pdf` 全文 | ✅（命题 5.1 的记号问题见 §B） |

### 1.4 参考文献列表（23 条全部抽查）

| # | 条目 | 结论 |
|---|---|---|
| 73 | [1]–[13] 的 arXiv 编号、标题、第一作者、年份 | ✅ 13/13 与 arXiv API 一致，链接可解析 |
| 74 | [14]、[15]、[17]、[18]、[19]、[20]、[21]、[22]、[23] 的 DOI | ✅ 9/9 在 Crossref 注册，标题与刊物一致；[16] 作者主页 HTTP 200。⚠️ [23] Crossref 只列 4 位作者（Keutzer, Newton, Rabaey, Sangiovanni-Vincentelli），文中多写了 Malik，请作者以 IEEE Xplore 为准核对 |

正文引用编号集合 = 文末列表 = {1…23}，一一对应，没有悬空引用，也没有未被引用的条目。写作简报编号表里的 [24] MAGE、[25] AlphaEvolve 正文没有使用，列表也没有收录，这是一致的。

---

## 2. 已修复清单

| # | 位置 | 原文 | 修复后 | 理由 |
|---|---|---|---|---|
| 1 | §0 第 2 段 | 能够启动 Linux 的 RISC-V 处理器 | 被作者称为“Linux-capable”的 RISC-V 处理器 | DC 正文没有报告启动 Linux（❌ #36） |
| 2 | §0 第 3 段 | 团队干脆直接引入参考 Python 实现来调试模块的功能与逐周期行为 | agent 为各模块编写 testbench 时干脆直接用参考 Python 实现来调试功能与逐周期行为 | 原文主语是 Conductor 2.0（agent） |
| 3 | 表 A “评估器与速度” | MHz 级等效吞吐，单次评估分钟至小时级 | 单次评估分钟至小时级，ChampSim 跑 10 亿条指令可超过 12 小时 [2] | “MHz 级”没有来源，并且与文章自己引用的 ArchAgent 数字矛盾（1B 指令 / 12 小时 ≈ 23 KIPS） |
| 4 | §1 商化公式前 | 令周期模型 $M$ 代表某个行为等价类 $S_e$ | 补“$M$ 取遍 $h$ 的像集，即每个 $M$ 都至少有一份 RTL 实现” | 等式 $\max_R J=\max_M\tilde J$ 只在这个定义域上成立；若 $M$ 可以不可实现，只有 $\le$ |
| 5 | §2 PyMTL 句 | 泛函层（FL） | 功能层（FL） | 术语误译：FL = functional level，不是数学上的“泛函” |
| 6 | §2 末 → §3 | 图注后直接接 `## 3.` | 补空行与 `---` | 与其他节的格式一致，避免部分渲染器把标题并入上一段 |
| 7 | §3.1 第二点 | 上式取紧的前提是 $I(G;R\mid M,C,K)\approx 0$ | 上式**成立**的前提正是马尔可夫性，即 $I(G;R\mid M,C,K)=0$；读 NL 文档时这一项大于零，上界不再成立 | **数学错误**：该项为 0 是不等式**成立**的条件（马尔可夫性本身）。取等号需要的是 $I(G;M,C\mid R,K)=0$ |
| 8 | §3.3 公式引入句 | 把通道 $X$ 的残余歧义进一步拆成可检测与沉默两部分 | 明确拆分对象是“接收方据此形成的解释与 $S_e$ 之间的残余偏差”，拆分标准是“能否被现有 oracle 在激励下暴露” | 原定义有歧义：§3.2 已定义 $A_X$ 是消息层面的条件熵，§3.3 却把它当成验证层面的量来拆 |
| 9 | §3.3 第 2 段 | $A_M^{\text{silent}}$ 只剩“$\mathcal{O}$ 之外的行为”和“未覆盖的输入” | $A_M$ 已为 0，剩下的是误读/误实现 $M$ 的偏差；沉默部分只剩激励未覆盖输入上的偏差；“$\mathcal{O}$ 之外的行为”按定义不在 $S_e$ 内，须经 $C$、$\Delta$ 显式交代 | **自相矛盾**：§3.2 写 $A_M=0$，§3.3 又写 $A_M^{\text{silent}}>0$；而且 $\mathcal{O}$ 之外的行为按 §1 的定义本来就不属于 $S_e$ |
| 10 | §3.4 | 对任意 $\alpha_{NL}>1$，……成本必然高于或等于直接写代码 | 对任意 $\alpha_{NL}>\alpha_M$（原框架默认代码一侧 $\alpha$ 取 1），……必然高于 | §3.2 已把代码改为 $\alpha_M\ge 1$，比较条件应同步改；$\alpha$ 严格大于时也是严格不等 |
| 11 | §3.5 首段 | 当修正收敛时总信息量满足 $\sum_t H(m_t)=H(S\mid K)$；拆为 $m_t^h$ 与 $m_t^o$ | 写明命题 5.1 的 $H(m_t)$ 实为条件互信息 $I(S;m_t\mid K,m_{<t})$（每轮消除的不确定性），按记账口径读作“每轮至少必须传输的信息量”，冗余消息的描述长度只会更大；用链式法则先计 $m_t^o$、再计“看过 $m_t^o$ 之后”的 $m_t^h$，由此定义 $H(m_t^h)$ | 用户指定的严谨性问题。原写法把信息增益当成消息熵，再当成人力成本，三个层次混在一起。新写法下守恒式严格成立，$E_{\text{human}}$ 是人工描述长度的下界，独立公式不变 |
| 12 | §3.5 公式后 | ……只取决于无法由机器代劳的 $m_t^h$。 | 补“$E_{\text{human}}$ 同样是记账量，是人工描述长度的下界” | 防止读者把 $E_{\text{human}}$ 当作测量量 |
| 13 | §4 不等式前 | （无） | 补“这里把写 $M$ 的 $\alpha_M$ 近似取 1；若计入建模样板，左边第一项变为 $(\alpha_{NL}-\alpha_M)H(S_e\mid K)$，结论不变” | 与 §3.2 的 $\alpha_M\ge1$ 保持一致；原式隐含 $\alpha_M=1$，推导本身未改 |
| 14 | §4 末 | `<!-- 疑点: … -->`（含 “01-plan.md” 字样） | 删除；补 `---` 与空行 | 用户要求删除；该注释还泄露了幕后文件名。处理结论见 §3 |
| 15 | §5.1 | Gutierrez……指出三类误差中规约误差是主导误差源 | 补“在他们用 gem5 对照 Cortex-A15 真实硬件的研究中” | 原文是 “dominant source in this work”，原稿泛化过度（⚠️ #66） |
| 16 | §5.2 第 1 段 | agent 在前 4 次……很快陷入平台化，直到切换……才拿到约 2.03× 的频率跃升……——最大收益恰恰来自改变周期行为 $M$ | 前 4 次 IPC-neutral 后平台化，随即切换；最佳迭代（第 7 次）相对基线约 2.03×；论文没有拆分两阶段贡献，但能确定 IPC-neutral 走不动之后，继续前进需要改变 $M$ | ❌ #30 过度归因；“很快”原文也没有 |
| 17 | §5.2 代理引理结语 | 只需一个**排序近似正确**（$\varepsilon$ 小）的相对代理 | 只需一个**在常数倍意义下误差有界**（$\varepsilon$ 小，绝对偏置 $b$ 无所谓）的相对代理 | 引理条件是对数误差一致有界，不是“排序正确”；与 IPC 相乘后，只是排序正确并不能保证结论 |
| 18 | 图 3 | 斜向移动: 改变 M（微降 IPC, 换取 ~2.03x 频率） | （微降 IPC, 最终达 ~2.03x 基线频率） | 同 #16：2.03× 是累计值 |
| 19 | §6.1 | ChipBench [6] 上 Verilog 生成仅 30.74% | 补模型名 Claude-4.5-opus | 按用户要求恢复文献中的真实模型名 |
| 20 | §6.2 | 被测最强模型……13.33%（注：……15.93%），低于 Verilog 生成，因 LLM 缺乏硬件行为建模知识 | Claude-4.5-opus……pass@1 仅 13.33%（摘要口径；贡献列表作 15.93%，而结果表中 15.93% 对应 Claude 4.5 Sonnet），整体低于其 Verilog 生成的 30.74%，作者归因于…… | 恢复模型名；补上新发现的文献内部不一致；把归因标为作者观点 |
| 21 | §6.4 | 且 DSL 训练语料远少于 C++/Python，先验互信息更低 [4] | ……（旁证：Design Conductor 观察到模型会把 Verilog 当顺序代码推理，并归因于软件语料占主导 [4]） | 过度归因：[4] 说的是 Verilog，没有谈 DSL |
| 22 | §6.5 | 死守逐拍严格相等会封死后端物理优化空间 [1] | ……——时序优化案例中 IPC-neutral 模式的平台化就是旁证 [1] | 过度归因：CHIA 没有说过这句话，这是作者的推论 |
| 23 | §6 末 → §7 | 段末直接接 `## 7.` | 补空行与 `---` | 格式 |
| 24 | 表 C 时序行 | IPC-neutral 提示词**证明**……；Iron Law 耦合使**最大收益**来自改变周期行为 $M$ | ……**表明**……；……使 IPC-neutral 平台化之后的进一步收益必须改变周期行为 $M$ | 违反写作约定（“证明”）；同 #16 |
| 25 | 表 C RISC-V 行 | 覆盖率按构造达 100% | 指令覆盖率按构造为 100% | 原文是指令覆盖，不是功能/状态覆盖（⚠️ #8） |

修改后复检：没有未闭合的 `$`；表格列数一致（表 A 3 列、表 B/C 4 列）；Mermaid 未改动，人工检查语法无误（所有标签都已加引号，subgraph 可以作边的端点）；独立公式为主线 7 个加引用块里的引理 1 个；禁用词、AI 协作、规划文档名、模型代号零命中（“证明”只剩引理的“**证明**”一处）。

---

## 3. 原文 HTML 疑点注释的处理结论

1. **疑点 1（$L_X(\delta)$ 的定义口径）**：正文写法“将残余歧义压到 $\delta$ 以下所需的最短消息长度”，与 $L_X(\delta)=\min\{|x|: H(S_e\mid x,K)\le\delta\}$ 一致，**无需修改**。剩下的小瑕疵是：$A_X$ 用的是逐点条件熵（条件于某条具体消息 $x$），$\alpha_X$ 的分母却是平均量 $H(S_e\mid K)$。在“记账模型”的声明之下可以接受，但不建议在文中进一步展开。
2. **疑点 2（伸缩求和的严格含义）**：判断正确。命题 5.1 的 $H(m_t)$ 是条件互信息，消息有冗余时描述长度之和 $\ge H(S\mid K)$。**已按 §2 #11、#12 修复**：不改推导，只改解释口径，并补上 $m_t^h$ 的链式法则定义。

---

## 4. 数学核查（逐项）

| 项 | 结论 |
|---|---|
| 链式法则 $H(R)=H(R,h(R))=H(S_e)+H(R\mid S_e)$ | ✅ 在 $h$ 为确定性函数时严格成立，前提是把 $R$ 视为随机变量（由开头的“记账模型”声明兜底）。 |
| 商化 $\max_R J=\max_M\tilde J$ | ✅ 已补定义域（#4）。 |
| 成本对比 $Nc_M+k(g_R+c_R)$ vs $N(g_R+c_R)$ | ✅ 代数正确；“省去 $(N-k)g_R$”成立。 |
| 数据处理不等式 $I(G;R\mid K)\le I(G;M,C\mid K)$ | ✅ 方向正确，条件于 $K$ 的马尔可夫链写法正确。❌ 前提的措辞写错，已修复（#7）。 |
| $\alpha_X(\delta)$ 定义与“$\alpha_{NL}$ 随精度上升” | ✅ 定义自洽；“随精度上升”已明确标为“我的猜测（目前只有旁证）”。“$\alpha_M$ 大致是常数”可以推出：由于 $A_M=0\le\delta$，$L_M(\delta)\le|M|$ 对所有 $\delta$ 成立，因此有界，不是假说。 |
| $A_X=A^{\text{detect}}+A^{\text{silent}}$ | ⚠️ 原定义有歧义并与 $A_M=0$ 矛盾，已修复（#8、#9）。 |
| §3.4 悖论区间的使用 | ✅ 已与 $\alpha_M$ 同步（#10）。推论 4.2 原式 $I\to0$ 时 $\alpha H(S)>H(S)$ 成立。 |
| 命题 5.1 与 $E_{\text{human}}$ | ⚠️ 已修复（#11、#12）。 |
| 分治不等式 $(\alpha_{NL}-1)H(S_e\mid K)+V_{NL}>H(C^+)+H(\Delta)$ | ✅ 由 $E_{\text{direct}}>E_{\text{split}}^{(B)}$ 直接推出，$S_a$ 与 $C_0$ 的相消假设都写明了。补充三点：① 隐含 $\alpha_M=1$，已注明（#13）；② $H(C^+)=H(C_{\text{split}})-H(C_0)\ge0$ 隐含 $C_0$ 是 $C_{\text{split}}$ 的函数，更严谨的写法是 $H(C_{\text{split}}\mid C_0)$，可以不改；③ 情形 A 的 $c_{\text{ready}}$ 和其他项的量纲不同（工作量 vs 比特），只能当记账量看。**会不会被误当作可测量量？** 全文只在开头声明一次记账口径（遵守简报），但 §4 的“盈亏边界由四项信息量共同决定”和推论 2 的“框架同时解释了正反两种结果”容易给读者“可以算”的印象，见 §5 判断项 J3。 |
| 代理引理与三行证明 | ✅ 三步不等式逐项验证正确，$b$ 正确相消，界 $e^{2\varepsilon}$ 正确。隐含前提：IPC 由 $M$ 精确给出、$M^*$ 是**候选集**上的最优、$\varepsilon$ 一致有界。结语的措辞已修（#17）。 |
| 伪量化 | ✅ 全文没有出现“某设计熵为 X 比特”；图 2 中 “≈H(S_e\|K)” 是记账表达，可以接受。 |

---

## 5. 未解决 / 需要作者判断的问题

| # | 位置 | 问题 | 建议 |
|---|---|---|---|
| J1 | §6.2 | “ChipBench……恰恰体现了自然语言通道残余熵 $A_{NL}$ 极大”**与被引文献自己的归因相反**：ChipBench 把失败归因于“缺乏硬件行为建模知识”（能力问题），不是规约歧义。 | 建议改成“ChipBench 的任务方向是‘从 NL 追隐藏 golden’，低分既可能来自能力不足（作者归因），也可能来自 NL 歧义，二者无法从其数据中分离”。这样更诚实，也不削弱论证。 |
| J2 | §5.1 | “此时规约误差在定义上不存在”说得过满：Gutierrez 意义上的规约误差确实消失了，但会出现新的误差源——$M$ 相对意图 $G$ 是否正确、是否可实现。 | 加半句“……转化为 $M$ 自身是否正确、可实现的问题（见 6.2）”。 |
| J3 | §4 推论 2、§0 | “框架同时解释了正反两种结果”容易被读作“不可证伪”。 | 加一句：“这只是事后一致性，真正的检验见第 8 节 P1–P3”。 |
| J4 | §6.2 | “对 $M$ 的功能幻觉可用 Spike 等 ISA 级 oracle 自检”只适用于执行驱动的核模型；trace 驱动的预取器/替换策略模型没有功能语义（ArchAgent 原话：“no notion of correct computation”）。 | 加一个限定词：“对执行驱动的核级 $M$”。 |
| J5 | 表 A | “单次评估分钟至小时级”“逻辑综合……单次十数分钟至数小时”属于常识性量级，没有文献来源。 | 保留可以，但别和“已核实量级”的说法混在一起；或改成定性描述。 |
| J6 | §5.1 末 | “单个 agent 用约 10.5 天……就做成了过去‘对大团队也是巨大工作量’的事”——结果是训练集 <3%、留出集 <7% 且有离群，说“做成”略显宽松。 | 改为“基本做成”或“做到了训练集 <3%”。 |
| J7 | 代理引理 | 写作约定是“折叠引用块”，当前是普通引用块，不能折叠。 | 若发布平台支持 `<details>` 且其中的 KaTeX 能正常渲染，可以改；否则保持现状。 |
| J8 | §0 | 信息论框架（`docs/信息论.pdf`）在正文以“定理 4.1 / 命题 5.1”被引用，却没有出处，读者无从追溯。 | 加一个内部链接或脚注。 |
| J9 | [23] | 作者是否含 Malik：Crossref 只列 4 位作者。 | 以 IEEE Xplore 为准核对。 |
| J10 | [20] [21] [18] [19] [22] [23] | 只核实到元数据，内容引语依赖 02 笔记（二手）。 | 如需完全稳妥，发布前人工翻一下 Black & Shen 与 Nowatzki 的原文。 |
| J11 | §6.5 DRAV 引语 | 原文是 “micro-architectures”（有连字符），文中写成 “micro architectures”。 | 可选修正。 |
| J12 | 字数 | 本轮修复为了严谨性净增约 340 汉字（12,381 → 12,722，不含参考文献）。 | 见 §7 的删减计划。 |

---

## 6. 额外的强反驳（建议在第 6 节择 2 条加入）

1. **“方向翻转”可能只是把逆向工程挪到了 $c_{\text{ready}}$ 这一步。**（**强烈建议加入**）在多保真度级联里，探索产出的是 ChampSim 式的 trace 驱动代码，没有时序语义，也没有功能语义。把它“升级为 RTL-ready 周期模型”时，必须**补写**原来不存在的周期行为，这一步本身就是在猜“这个机制在真实流水线里会怎么表现”，正是 Gutierrez 所说的规约误差。因此第 5.1 节的方向论证只在情形 B（直接写 $M$）下干净；情形 A 的规约误差并没有消失，只是转移了。建议回应：承认这一点，并把 P3 明确设计成能检验 $c_{\text{ready}}$ 步骤的误差。
2. **周期层的“第一处分歧”远不如 ISA 层那样有信息量。**（**强烈建议加入**）ISA lockstep 有效，是因为提交时的架构状态有规范的对齐点。到了周期层，一旦允许 $\Delta$（合法时序差异），两条执行轨迹在第一次合法偏移之后就会失去逐拍对齐，“第一处分歧”可能是合法差异的连锁反应，而不是 bug。也就是说，$m_t^o$ 的信息量和 $H(\Delta)$ 是此消彼长的，而且新颖微架构往往恰好引入新的合法不确定性。这直接削弱第 3.5 节的“付费者转移”在性能层的强度。建议回应：承认性能层 oracle 只能做统计/计数器级的定位（`gem5_align` 的 %diff + TMA 正是这种形态），把 3.5 节的强结论限定在功能层。
3. **λ 下降是双刃剑。**（建议加入，一句话即可）文章预测“λ 下降 ⇒ 分界线向分治移动”，但 λ 下降同样会压低 $g_R$（agent 写 RTL 更便宜）和 $c_R$（FireSim 等 FPGA 加速），使第 1 节的“在 RTL 上探索”路线变得可行。CHIA 论文 §5.4.1 自己就提到可以把设计放到 FireSim 上获得更高保真度的反馈。建议回应：分治的长期优势应主要建立在信息论论证（第 3 节）上，而不是成本论证（第 1 节）上。
4. **代理引理是条件性的：它证明了“若有好代理则够用”，没有证明好代理存在。**（可选）16nm 不泛化恰恰说明 $\varepsilon$ 可能很大。读者会问：$\hat f$ 从哪来？如果必须综合 RTL 才能校准 $\hat f$，那综合又回到了内环。建议在 5.2 节加一句承认，并指向“下一步 4”。

---

## 7. 逐节删减建议与目标字数

总目标约 **8,600 汉字**（不含参考文献、公式与表格内的英文）。我认为不必压到 7,500：第 3 节和第 5 节承载核心论证，压得过狠会把“收窄”的推理链切断。删减原则：同一事实或金句在全文出现 3 次以上的，只保留 1 次正文加 1 次结尾。

| 节 | 现字数 | 目标 | 必须保留（核心） | 具体删减 |
|---|---:|---:|---|---|
| TL;DR | ~405 | 300 | 5 条要点的骨架 | 第 3、4 条各压成一句；删去第 4 条括号里的“模型追 RTL / RTL 追模型”展开（第 5 节会讲） |
| §0 开篇 | ~910 | 650 | 反直觉判断；DC 的 §5.3/§6.2 钩子；主论点粗体段；记账声明 | DC 段删“到了 DC 2.0……”之后的半句复述；路线图段（“接下来，第 1 节……”）压成一句；记号段删去对 $I(\cdot;\cdot)$ 的解释 |
| §1 拆分 | ~1,500 | 1,000 | 表 A；恒等式及其“这意味着”；商化；成本对比；“本质/偶然是相对的”伏笔 | 量级例证段删 Assassyn（§2 和 §6.4 还会出现）；“一旦承认这个拆分……”过渡句删除；恒等式后的解读段删去“比如用独热码……”的三个例子，只留一个 |
| §2 三重身份 | ~1,320 | 900 | $M$ 的定义与 $R\models M$；“精确是关系不是属性”；表 B；多保真度收窄；图 1 | 收窄段删去对 Assassyn 速度的复述，以及 ArchAgent 可实现性那句（§5.2、§6.2 重复）；PyMTL/TLM 两句合并为一句；删“这里最容易引发的第一个质疑是……”的设问，直接给定义 |
| §3 信息论 | ~1,810 | 1,400 | 3.1 的 DPI 与“单一事实来源”规则；3.2 的 $\alpha$ 定义与假说；3.3 全部（全文最大的洞察）；3.4 的金句；3.5 的付费者拆分与 riscv 原话 | 3.1“第一”段删“原本端到端不可分解的……”；3.3 的 CIRCT/ClarifyGPT 段压成一句（§7 表 C 和“第二条线索”重复讲了两遍）；3.4 删“简言之……”一段（与上方金句重复）；图 2 保留 |
| §4 分治账 | ~650 | 600 | 不等式与三条推论；情形 A 对 C2 的回扣；λ 假说 | 四个符号定义改成一行内的括注；推论 2 可以和 §6.1 合并（二者重复），以 §4 为主 |
| §5 两个反例 | ~1,730 | 1,300 | 5.1“方向反了”金句与 Gutierrez；收窄为假说 P2；“倒过来跑 `gem5_align`”；5.2 三条工程边界、代理引理、图 3 | 5.1 承认段把 Black & Shen、XiangShan、CHIA docs、论文细节四重证据压成两重（保留 XiangShan 30% 与论文的 <3%/<7%/过拟合）；“回顾处理器验证史”段与金句重复，压成一句；5.2 删去与 §7 表 C 重复的 docs 数字 |
| §6 其他质疑 | ~1,060 | 800 | 6.2（ChipBench，最直接的反证）；6.3（资深读者的第一反应）；6.5 的 DRAV 引语 | 6.1 压成两句，结论指回 §4；6.4 删去重复的 2.2× 数字；6.5 删末句“与 CIRCT 的 UNCLEAR 门遵循同一原则”（第三次出现）；若采纳 §6 的反驳 1、2，各加约 120 字 |
| §7 CHIA 案例 | ~1,450 | 800 | 表 C（但要瘦身）；“it doesn't invent the ISA”这条线索；“我不会把五个案例写成都支持我”的态度 | 表 C“关键事实”列每格只留 1–2 个数字（目前和 §5 大量重复）；第二条线索（CIRCT）压成一句；第三条线索与 §5 开头重复，压成一句 |
| §8 预测与下一步 | ~1,395 | 850 | P1–P3（可观测量与否证条件）；下一步 4 条；收尾段 | P1–P3 每条压到 2–3 句；P4 降级为一句话的趋势判断，或删除；“三句话带走”与正文三个金句一字不差，建议保留结尾这组，把正文 §3.4、§3.5、§5.1 的引用块降级为普通粗体句（或反过来） |
| **合计** | **~12,720** | **~8,600** | | 若采纳 §6 反驳 1、2，约 8,850，仍在 9,000 以内 |

---

## 8. 整体评估

**能否作为内部技术品牌文章立住？能。** 前提是做完上面的删减，并处理 J1–J4。文章有鲜明而可被反驳的立场（“让 RTL 对模型精确”）。它主动暴露最弱的子论点并收窄，正反证据都摆了出来，最后给出写清可观测量与否证条件的预测，这些都是资深读者会认可的写法。事实层面，修复后所有数字、引语都能回溯到一手来源，CHIA 论文独有数字全部有“据 CHIA 论文”标注，假说也一律标成“我的猜测”。

**最大的 3 个风险：**

1. **核心新意建立在一个尚无直接证据的假说上。** “周期层 RTL 追模型比模型追 RTL 便宜”目前只有 ISA 层先例和误差源归因作旁证；CHIA 的五个案例没有一个检验了周期层的完整桥。文章已经诚实标注，但“方向反了”这类小标题的语气比证据强。再加上 §6 反驳 1（$c_{\text{ready}}$ 里规约误差会复现）和反驳 2（周期层分歧信息量低），资深读者很可能在评论区精确地指出这两点。建议主动写进第 6 节。
2. **信息论框架可能被读成“装饰性数学”。** 7 个独立公式里的量都不可测，开头只声明了一次记账口径；推论“框架同时解释了正反两种结果”还会招致“不可证伪”的批评。本轮修掉的两处数学瑕疵（数据处理不等式的前提、$A_M$ 的矛盾）恰恰是懂信息论的读者最先会挑的。修复后推导已经站得住，但建议按 J3 把“事后一致性”和“可证伪检验（P1–P3）”的分工写明。
3. **篇幅与重复，以及对预印本数字的依赖。** 12,700 字里同一组事实（gem5 10.5 天 / <3%、2.03×、UNCLEAR 门、“golden model is the check”）各出现 3–4 次，内部读者很可能读不完。另外，大部分外部证据来自 2026 年未经同行评审的 arXiv 预印本，而且至少一篇（ChipBench）存在内部数字不一致。一旦被读者抓到误引，对个人技术品牌的伤害远大于删掉一段的损失。本轮已逐条核对，但如果在发布前再改数字，建议对照本报告第 1 节复核。
