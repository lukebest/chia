# 成文共享要求(所有分段写作者必须遵守)

## 文章定位
- 中文技术文章,作者第一人称"我",发布在公司内部技术交流社区,目的:建立个人技术品牌,体现深度思考与洞察力。读者:资深架构师、RTL 工程师、AI 工程师。
- 主标题:**让 RTL 对模型精确:AI 时代架构探索与 RTL 生成的分界线**
- 副标题:一个信息论记账模型、五个 CHIA 案例,以及几个可证伪的预测
- 核心立场:自动架构探索与 RTL 生成应拆成两个课题,以**周期级可执行模型(cycle-level executable model,简称周期模型 M)**作桥;探索的**交付物**就是 M;RTL 生成以 M 为信息对齐的主要手段(代码的歧义性比自然语言小,更关键是"可执行 ⇒ 沉默歧义变成可检测分歧")。
- 已做的收窄(必须保留,并写成亮点):(1) M 是探索的"交付物"而不是"全程评估器",探索用多保真度级联;(2) oracle 分两层:功能精确 + 性能在规则/容差内一致,合法差异 Δ 显式建模;(3) 桥是双向外环,物理信息经侧信道 C 回流;(4) 周期层"RTL 追模型"比"模型追 RTL"便宜,是假说。
- 风格:反直觉判断开路;克制但敢下判断;主动暴露最弱子论点并展示如何收窄;不写综述,不写 CHIA 产品宣传;每个公式后紧跟一句"这意味着……"。

## 材料来源(不得编造)
- 事实、数字、引语只能来自:`00-context-brief.md`、`01-plan.md`、`02-research-notes.md`(同目录)。需要精确引语时查 02。
- 严禁编造论文/数字/链接/引语。
- CHIA 论文独有数字(plan 中标 [CHIA-paper])须写"据 CHIA 论文"。
- plan 中标 [假说] 的判断,正文写成"我的猜测(目前只有旁证)"之类。
- gem5 对齐"<3%"只能说成训练集结果,同时给出留出集 Embench <7%(据 CHIA 论文)。
- "LLM 自动生成 gem5/周期级模拟器"相关研究:写"我检索范围内未见专门工作"。
- 不要写"CHIA 证明了……";写"CHIA 的 X 案例提供了 Y 方面的证据,但没有检验 Z"。
- gem5 对齐与时序优化两个案例要以"它们挑战了我什么、我因此收窄了什么"的方式出现。

## 格式与禁忌
- Markdown + LaTeX(`$...$`、`$$...$$`)。全文独立公式不超过 7 个(恒等式、评估成本对比、数据处理不等式、A=A^detect+A^silent、付费者拆分、分治不等式、代理引理),与 `01-plan.md` §2 保持一致,不得自行改推导;发现疑似错误请在你那一段末尾用 HTML 注释 `<!-- 疑点: ... -->` 记录。
- 全文只在开头声明一次"这是记账模型,不是测量理论";不得出现"某设计熵为 X 比特"。
- 术语首次出现"中文(English)",之后统一。工具名/案例名保持英文(gem5、Spike、ChampSim、BOOM、`gem5_align`、`riscv_extensions`)。不要解释 IPC、cosim、关键路径等;要解释本文新概念(周期级可执行模型、观测接口 O、精化、沉默歧义、可检测分歧)。
- 禁用空泛词:赋能、颠覆、革命性、全面、助力、打通、闭环生态、降本增效、新范式(除非紧跟具体定义)。
- 不提 AI 协作、"规划文档""成文模型""Gemini""Opus"等幕后信息。
- 引用用方括号编号,**必须使用下表的统一编号**,只引用下表里的条目。

## 统一引用编号表(参考文献最终由整合者统一生成,分段写作者只需在正文里使用编号)

| 编号 | 条目 | URL |
|---|---|---|
| [1] | CHIA: An open-source framework for principled, agentic AI-driven HW/SW co-design research (Cui, Hermida-Rivera, Toubes et al., 2026) | https://arxiv.org/abs/2606.27350 |
| [2] | ArchAgent: Agentic AI-driven Computer Architecture Discovery (Gupta et al., 2026) | https://arxiv.org/abs/2602.22425 |
| [3] | Agentic Architect (Kypriotis, Blasberg, Moiras, Skarlatos, 2026) | https://arxiv.org/abs/2604.25083 |
| [4] | Design Conductor (Verkor Team, 2026) | https://arxiv.org/abs/2603.08716 |
| [5] | Design Conductor 2.0 (Verkor Team, 2026) | https://arxiv.org/abs/2605.05170 |
| [6] | ChipBench (Yu et al., 2026) | https://arxiv.org/abs/2601.21448 |
| [7] | CVDP: Comprehensive Verilog Design Problems (Pinckney et al., 2025) | https://arxiv.org/abs/2506.14074 |
| [8] | Spec2RTL-Agent (Yu et al., 2025) | https://arxiv.org/abs/2506.13905 |
| [9] | FormalRTL (Li et al., 2026) | https://arxiv.org/abs/2603.08738 |
| [10] | LLM-FSM (Wu et al., 2026) | https://arxiv.org/abs/2602.07032 |
| [11] | HLSPilot (Xiong, Liu, Li, Li, 2024) | https://arxiv.org/abs/2408.06810 |
| [12] | ClarifyGPT (Mu et al., 2023) | https://arxiv.org/abs/2310.10996 |
| [13] | Understanding and Mitigating Errors of LLM-Generated RTL Code (Zhang et al., 2025, TCAD) | https://arxiv.org/abs/2508.05266 |
| [14] | Sources of Error in Full-System Simulation (Gutierrez et al., 2014, ISPASS) | https://doi.org/10.1109/ISPASS.2014.6844457 |
| [15] | Towards Developing High Performance RISC-V Processors Using Agile Methodology (Xu et al., 2022, MICRO) | https://doi.org/10.1109/MICRO56248.2022.00080 |
| [16] | End-to-End Verification of ARM Processors with ISA-Formal (Reid et al., 2016, CAV) | https://alastairreid.github.io/papers/CAV_16/ |
| [17] | Assassyn (Weng et al., 2025, ISCA) | https://doi.org/10.1145/3695053.3731004 |
| [18] | PyMTL (Lockhart, Zibrat, Batten, 2014, MICRO) | https://doi.org/10.1109/MICRO.2014.50 |
| [19] | The Essence of Bluespec / Kôika (Bourgeat et al., 2020, PLDI) | https://doi.org/10.1145/3385412.3385965 |
| [20] | Calibration of Microprocessor Performance Models (Black, Shen, 1998) | https://doi.org/10.1109/2.675637 |
| [21] | Architectural Simulators Considered Harmful (Nowatzki et al., 2015) | https://doi.org/10.1109/MM.2015.74 |
| [22] | Transaction Level Modeling: An Overview (Cai, Gajski, 2003) | https://doi.org/10.1145/944645.944651 |
| [23] | System-level design: orthogonalization of concerns (Keutzer et al., 2000) | https://doi.org/10.1109/43.898830 |
| [24] | MAGE (Zhao et al., 2024) | https://arxiv.org/abs/2412.07822 |
| [25] | AlphaEvolve (Novikov et al., 2025) | https://arxiv.org/abs/2506.13131 |

## 分段与文件
- 03a.md:YAML 头 + TL;DR(≤5 条)+ 第 0 节 + 第 1 节 + 第 2 节
- 03b.md:第 3 节 + 第 4 节
- 03c.md:第 5 节 + 第 6 节
- 03d.md:第 7 节 + 第 8 节(预测 + 下一步 + 结尾句)
- 节的定义与字数见 `01-plan.md` §6.2;图表见 §6.3;金句见 §6.4;风格见 §7。
- 每段结尾不要写"未完待续"之类;整合者会拼接。小标题层级:`#` 仅用于主标题(03a);各节用 `##`,小节 `###`。
- 如果你写的段落中用了图表(Mermaid / 表 A / 表 B / 表 C / 图 2 / 图 3),放在最合适的位置:图 1 + 表 A + 表 B 在 03a;图 2 在 03b;图 3 在 03c;表 C 在 03d(§7)。
