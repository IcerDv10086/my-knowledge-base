# IC验证AI Agent资料综述与学习计划

> 整理时间：2026年5月
> 目标读者：IC验证工程师，想将AI Agent应用到实际工作中

---

## 一、背景与行业痛点

IC验证消耗项目60-70%的开发时间，是芯片设计中最耗人力的环节。传统验证面临三大挑战：

1. **数据稀缺**：公开的UVM代码和验证案例极少，绝大多数真实验证环境是公司私有资产
2. **长上下文依赖**：生产级UVM涉及数十到上百个文件，宏定义、继承关系、工厂注册机制复杂
3. **跨阶段协作**：从规格→验证计划→testbench→覆盖率→debug，需要在多种工具和文档间切换

AI Agent技术的出现，为解决这些问题提供了新的可能。

---

## 二、商业工具现状

### 2.1 西门子 Questa One Agentic Toolkit

**产品定位**：企业级AI Agent工具包，深度集成到Questa验证平台

**核心Agent能力**：

| Agent | 功能 | 实际价值 |
|-------|------|----------|
| **RTL Code Agent** | 自然语言描述生成RTL代码 | 生成可综合的高质量SystemVerilog |
| **Lint Agent** | 自动检测代码违规并修复 | 自动化代码风格检查 |
| **CDC Agent** | 时钟域交叉分析 | 早期发现跨时钟域问题 |
| **Verification Planning Agent** | 验证计划创建与管理 | 组织任务、覆盖率目标 |
| **Debug Agent** | 仿真失败分析 | 自动定位根因并给出修复建议 |

**官网**：<https://www.siemens.com/en-us/products/ic/questa-one/agentic-toolkit/>

**推荐理由**：与现有Questa工作流深度集成，适合已使用西门子工具链的团队

---

### 2.2 Moores Lab AI - VerifAgent™

**产品定位**：AI驱动的端到端IP验证平台

**核心能力**：

- 自动生成测试计划、UVM testbench、测试用例
- 功能覆盖率模型和断言生成
- 据称可将验证速度提升**7倍**，成本降低**86%**

**官网**：<https://www.mooreslab.ai/>

**特色**：与Breker合作，提供SoC级验证方案

---

### 2.3 ChipStack UVMAgent

**产品定位**：增强现有UVM环境，而非替代

**核心工作流**：

1. **Test Plan创建**：分析设计规格，生成UVM测试计划
2. **Functional Coverage生成**：自动创建covergroup模型
3. **Testbench增强**：自然语言描述，Agent自动实现testbench更新
4. **覆盖率缺口分析**：跑仿真后自动分析覆盖率报告
5. **Debug指导**：失败时给出修复建议

**官网**：<https://www.chipstack.ai/product-pages/uvmaiagent>

**推荐理由**："增强而非替代"的定位更务实，适合渐进式引入AI

---

### 2.4 Cadence AI方案

**产品组合**：

- **ChipStack AI Super Agent**：覆盖design、verification、testbench coding、test-plan creation、debug
- **JasperGold AI**：formal验证辅助

**特色**：强调错误分析能力，能追踪信号到信号，给出类似"这个信号走到了X，应该走到Y"的诊断

---

### 2.5 新思科技 Synopsys - AgentEngineer™

**产品定位**：行业首个L4级编排的多智能体AI芯片设计和验证工作流

**核心架构**：

基于Synopsys行业领先的Agentic AI框架，构建可推理、规划、学习和执行工程任务的多智能体系统

**核心Agent能力**：

| 能力 | 功能描述 |
|------|----------|
| **RTL生成** | 自然语言描述或形式化规格 → 可综合RTL代码 |
| **Lint检查** | 自动检查代码规范，识别违规模式 |
| **Testbench生成** | 自动生成单元级验证testbench |
| **形式验证** | 形式验证flow，提升签核深度和效率 |
| **智能编排** | 跨多个Synopsys EDA工具的智能编排与自适应学习 |

**关键成果**：

- **生产力提升10-20倍**（官方宣称）
- 客户实际反馈：设计验证效率提升**2-5倍**
- NVIDIA正在试点用于AI驱动的形式验证
- 2026年3月发布L4级orchestrated multi-agent workflow

**技术合作**：

- 与NVIDIA NeMo Agent Toolkit集成，使用Nemotron开源模型
- 推动自主设计flow的下一代半导体开发

**L1-L5能力分级**：

| 等级 | 描述 |
|------|------|
| L1 | 基础辅助自动化 |
| L2-L3 | 协作式半自主Agent |
| L4 | 编排型多Agent协同（当前） |
| L5 | 高度自主、自我导向Agent |

**官网**：<https://www.synopsys.com/ai/agentic-ai.html>

**推荐理由**：覆盖从规格到RTL的完整工作流，与Synopsys主流EDA工具链无缝集成，适合已使用Synopsys工具链的团队

---

## 三、开源框架与研究

### 3.1 UVM² - 东南大学等

**论文**：[From Concept to Practice: an Automated LLM-aided UVM Machine for RTL Verification](https://arxiv.org/html/2504.19959v2)

**核心思路**：

- 用LLM生成UVM testbench
- 用覆盖率反馈迭代优化
- benchmark：RTL设计最长1.6K行代码

**关键指标**：

- 测试平台搭建时间降低**38.82倍**
- 代码覆盖率87.44%，功能覆盖率89.58%
- 比同期方案分别高出20.96%和23.51%

**架构**：

```
规格 → 测试计划 → Testbench生成 → 仿真分析 → 测试用例补充 → 覆盖率闭环
```

---

### 3.2 Pro-V - NVIDIA + UCSD

**开源地址**：<https://github.com/stable-lab/Pro-V>

**核心创新**：

- **多Agent协同**：多个Agent分工生成testbench
- **Best-of-N迭代采样**：生成多个候选，选择最优
- **LLM-as-Judge验证框架**：判断失败是RTL问题还是testbench问题

**指标**：

- 验证准确率87.17%（golden RTL）
- RTL mutants上76.28%

---

### 3.3 RTL-CLAW - 同济大学 + 港中文

**开源地址**：<https://github.com/TONGJI-EDA-LAB/RTL-CLAW>

**定位**：AI-Agent驱动的IC设计全流程框架，基于OpenClaw

**当前能力范围**：

- RTL分析、分区、优化
- 验证与testbench生成
- 逻辑综合

**架构**：三层设计（交互层→Agent核心层→工具数据流层）

---

### 3.4 Verilog-TestBench-ACP-Agent

**开源地址**：<https://github.com/masterthefly/Verilog-TestBench-ACP-Agent>

**技术栈**：Gemini 3 + ACP协议 + Icarus Verilog

**工作流**：

```
读Verilog → 生成testbench → 运行iverilog仿真 → 分析波形 → 迭代修复
```

**意义**：展示了用AI Agent做硬件验证的完整闭环

---

## 四、国内方案

### 4.1 合见工软 - UDA 2.0

**公司背景**：上海合见工业软件集团，2020年成立，国内唯一可完整覆盖数字芯片验证全流程的国产EDA公司

**产品定位**：国内首款自主研发Agentic AI EDA平台

**核心能力**：

- 基于DeepSeek等先进大模型
- 内嵌多个专业EDA引擎（逻辑综合、性能评估、功能分析、验证）
- 自动调用底层引擎，完成RTL代码生成→代码检查→验证调试→持续迭代优化的完整闭环
- 支持基于自然语言的指令或Spec，自动生成高质量Verilog RTL代码
- 支持语法和设计规范检查及自动纠错

**关键指标**：模块级设计验证效率提升**3-5倍**

**官网**：<https://www.univista-isg.com/>

---

### 4.2 伴芯科技 (IC Bench) - DVcrew

**公司背景**：2020年成立，已获红杉中国、联想创投等融资，与全球前20大无晶圆厂设计公司合作

**产品定位**："Design Verification Team in a Box"

**核心能力**：

- 自动分析设计和测试
- 自动生成设计假设和检测器
- 端到端自动化验证流程
- 显著加快验证周期
- 增强全面设计空间覆盖的信心
- 支持本地化部署，保障数据安全

**客户案例**：

- 车规芯片设计引领企业：在评估期间发现设计中难以预见的错误
- AIoT智能终端SoC企业：快速处理代码库，提供高质量验证结果
- 高性能射频与混合信号芯片设计企业：复杂状态机和射频接口时序验证

**官网**：<https://www.icbench.com/>

---

### 4.3 华大九天 - PyAether

**公司背景**：国内最大EDA厂商，产品覆盖模拟芯片、平板显示、射频等

**产品定位**：Empyrean PyAether全定制设计平台生态系统

**核心能力**：

- 基于Python统一架构，实现高度自动化与智能化
- 提供超过**1.2万个Python API接口**
- 支持构建电路与版图自动化任务
- 快速进行各类数据处理与分析
- 自带AetherWings定制化工具包
- 开放部分源码和演示Demo示例
- 支持三方工具集成机制

**定位**：适合模拟/射频芯片设计验证，提供Python API扩展能力

**官网**：<https://www.empyrean.com/>

---

### 4.4 芯华章 - 智能SVA生成

**公司背景**：提出"EDA 2.0"理念，推动从AI-Driven向Agentic AI演进

**核心突破**：

- 智能SVA生成工具：与中兴微电子联合研发
- 完成语法、功能、质量三层自动化校验
- 将形式验证从资深专家专属能力变为通用开发标配

**关键成果**：

- 某头部互联网客户AI推理芯片项目：将原本**3个月的验证周期压缩至3周**
- 2025年硬件仿真器发货量突破百台
- 在智算芯片领域完成超40台硬件仿真器超大规模级联部署

**官网**：<https://www.epwaf.com/>（芯华章）

---

### 4.5 广立微 - SemiClaw

**产品定位**：企业级数字员工平台

**核心思路**：

- 不是问答工具，而是能理解任务、自主执行、主动汇报的AI数字员工
- 自然语言下达指令，自动完成数据调取→分析建模→图表生成→报告输出的完整链路

**验证案例**：

- 5个Lot、25片Wafer、40000颗Die的CP数据及WAT/MET/WIP/FDC数据
- 输入："做良率分析并分析背后的原因"
- 系统自主规划七层递进分析路径，几分钟输出完整报告
- 发现整体良率85.17%，LOT1003仅79.08%，Edge区域良率81.88%等关键问题

**官网**：<https://www.scienwonder.com/>（广立微）

---

### 4.6 芯和半导体 - XAI多智能体平台

**产品定位**：多智能体平台XAI

**核心能力**：

- 四大智能体嵌入全流程：建模智能体、仿真智能体、交互智能体、数算智能体
- 覆盖建模→设计→仿真→优化完整闭环

---

### 4.7 中科麒芯 - IC Agent Hub

**背景**：团队来自中科南京智能技术研究院，半导体行业20多年经验

**产品**：IC Agent Hub平台，预装38个IC Skills

#### 4.7.1 功能验证相关Skills

| Skill | 功能描述 |
|-------|----------|
| Spec-Driven Testbench Generator | 从规格生成Testbench |
| UVM Environment Generator | 生成完整UVM验证环境 |
| UVM Testcase & Sequence Generator | 生成测试类和虚拟序列 |
| SVA Assertion Generator | 生成SystemVerilog断言 |
| Compile & Simulation Failure Triager | 诊断编译/仿真失败原因 |
| Waveform Debug Advisor | 波形调试指导 |
| Regression & Coverage Analysis | 回归结果分析、覆盖率收敛 |
| 形式验证与断言 | 规划高价值断言、生成SVA |

#### 4.7.2 其他Skills（按设计阶段）

**系统规格与架构**：

- Specification Fact Extractor：提取接口、寄存器、时钟复位信息
- Microarchitecture Analyzer：分析数据通路、控制逻辑
- Spec Analyzer & Testplan Generator：生成验证测试计划

**RTL设计与集成**：

- RTL Code Generator：生成可综合SystemVerilog
- RTL Architecture Planner：模块划分与实现优先级
- Top-Level Integration & Bring-Up Planner：顶层集成、filelist组织

**工具链自动化**：

- RTL-Design-Flow Orchestrator：全流程编排
- Simulation Runner & Auto-Fixer：EDA调用、日志解析、自动修复

**综合与物理**：

- SDC生成、逻辑综合、QoR分析
- Floorplan、PnR、GDS导出
- 等价性检查、DRC/LVS

#### 4.7.3 安全验证机制

每个Skill上线前必须通过5步验证：

1. **Format Check**：SKILL.md字段完整性
2. **Security Scan**：静态代码分析、高危检测
3. **Dependency Check**：声明依赖vs实际依赖
4. **Runtime Detection**：实际运行检测行为
5. **Assessment Report**：风险分级+人工审核

每个Skill详情页显示：Quality Stars评分、风险等级、安全扫描结果（文件访问/命令执行/网络行为/数据泄露）

**相关文章**：

- [把AI工具装进芯片设计流程，终于有人做了个IC Agent Hub](https://c.m.163.com/news/a/KR4CC5KD0552IBJ1.html)
- [从Spec到GDS，一颗MCU的AI Agent之旅](http://m.163.com/news/article/KSQ246QC0556C9YG.html)

---

## 五、关键洞察：AI在验证中的擅长与不擅长

### 5.1 AI适合的场景 ✅

| 场景 | 原因 |
|------|------|
| 生成testbench骨架和样板代码 | UVM组件结构相对模板化 |
| 编写覆盖率模型covergroup | 语法固定，模式清晰 |
| 分析regression失败日志，归类错误 | 文本处理是LLM强项 |
| 生成测试计划文档初稿 | 自然语言生成 |
| Lint/代码风格检查 | 规则明确，执行确定 |
| 错误信息解释 | 有足够上下文可推断 |
| 生成SVA断言 | 协议规则可文本描述 |

### 5.2 AI困难的场景 ❌

| 场景 | 原因 |
|------|------|
| 保证覆盖率真正收敛 | 需要设计意图理解，当前模型缺乏 |
| 发现corner-case bug | 公开数据少，模型训练不足 |
| 深度debug定位根因 | 跨模块、跨时间窗口的推理 |
| 理解复杂设计架构 | 上下文窗口有限，长依赖处理差 |
| 处理敏感私有设计数据 | 数据安全和隐私顾虑 |

### 5.3 验证工程师的正确姿势

**核心认知**：AI Agent是**工具**，不是**替代者**

```
验证工程师的价值 = 领域专业知识 × AI放大器

AI负责：重复性劳动、文档生成、初步分析
人负责：设计意图判断、覆盖率策略、复杂debug、最终签核
```

---

## 六、分阶段务实学习计划

> 基于"不能全职学，但要用到工作"的约束制定

### 阶段1：先用起来（目标：2-3周）

**原则**：不改变现有工作流，用AI解决具体小问题

| 优先级 | 具体任务 | 预期收益 |
|--------|----------|----------|
| P0 | 用AI辅助生成验证计划文档初稿 | 节省文档编写时间 |
| P0 | 用AI分析regression失败日志，归类错误模式 | 减少重复性debug时间 |
| P1 | 用AI生成常用UVM组件模板（agent、sequence、monitor） | 减少样板代码编写 |
| P1 | 用AI辅助解释仿真错误信息 | 加速问题定位 |

**每周投入**：约5小时（工作日每天30分钟+周末1小时）

**推荐尝试顺序**：

1. 先用ChatGPT/Claude体验"AI生成测试计划文字"（无需任何安装）
2. 用西门子/Cadence工具链中的AI功能（如果有license）
3. 体验ChipStack或Moores Lab的demo

---

### 阶段2：嵌入工作流（目标：4-6周）

**原则**：让AI参与具体验证任务，形成人机协作流程

| 优先级 | 具体任务 | 需要的准备 |
|--------|----------|------------|
| P0 | AI辅助覆盖率分析：让AI读coverage报告，给出covergroup建议 | 积累阶段1经验 |
| P0 | 用AI生成testbench骨架，人工补充细节 | 了解当前AI生成质量边界 |
| P1 | AI辅助debug：让AI分析log，给出可能根因 | 建立对AI能力的信任 |
| P2 | 尝试用自然语言生成简单SVA断言 | 学习断言语法 |

**每周投入**：约7小时

---

### 阶段3：构建个人知识资产（持续迭代）

**目标**：将个人经验封装成可复用工具

| 方向 | 具体行动 |
|------|----------|
| Skill积累 | 把常用的验证模式、UVM模板、checklist整理成Skill |
| 流程自动化 | 串联多个任务成自动化pipeline（如spec→testplan→testbench骨架） |
| 团队共享 | 与团队共享有效Skill，建立团队知识库 |
| 反馈积累 | 记录AI哪些做得好、哪些不行，形成团队认知 |

**最终目标**：形成适合自己团队的"验证AI工作台"

---

### 每日节奏建议

```
工作日：
├─ 30分钟：在实际任务中用AI（不追求完美，能用就行）
├─ 20分钟：记录遇到的问题和AI表现
└─ 10分钟：简单整理

周末（可选）：
└─ 1小时：系统学习一个进阶主题或尝试新工具
```

---

## 七、立即可用的资源

### 7.1 商业工具试用

| 工具 | 获取方式 | 适合场景 |
|------|----------|----------|
| 新思科技 AgentEngineer™ | 联系Synopsys销售 | 已使用Synopsys工具链的团队，L4多Agent工作流 |
| 西门子 Questa One Agentic Toolkit | 联系西门子销售/技术支持 | 企业用户，有Questa license |
| ChipStack UVMAgent | <https://www.chipstack.ai> | 先看demo了解能力边界 |
| Moores Lab VerifAgent | <https://www.mooreslab.ai> | 评估端到端验证效果 |

### 7.2 国内厂商方案

| 厂商 | 产品 | 获取方式 | 适合场景 |
|------|------|----------|----------|
| 合见工软 | UDA 2.0 | <https://www.univista-isg.com/> | 模块级设计验证，RTL生成→验证闭环 |
| 伴芯科技 | IC Bench | <https://www.icbench.com/> | 端到端自动化验证流程，支持私有部署 |
| 芯华章 | 智能SVA生成 | 联系芯华章销售 | 形式验证辅助，压缩验证周期 |
| 中科麒芯 | IC Agent Hub | <https://c.m.163.com/news/a/KR4CC5KD0552IBJ1.html> | 38个预装Skills，覆盖验证全流程 |

### 7.3 开源框架研究

| 项目 | 地址 | 学习价值 |
|------|------|----------|
| UVM²论文 | arXiv:2504.19959v2 | 理解AI+UVM的技术原理 |
| RTL-CLAW | github.com/TONGJI-EDA-LAB/RTL-CLAW | 学习多Agent协同架构 |
| Pro-V | github.com/stable-lab/Pro-V | 了解LLM-as-Judge验证思路 |

### 7.4 学习资料

**文章**：

- [opendv.net - AI如何让硬件验证更容易](https://opendv.net/llm-in-verification/)：入门级介绍
- [SemiEngineering - Toward Agentic Verification](https://semiengineering.com/toward-agentic-verification/)：行业趋势分析
- [麒芯说AI - 验证的入口正在从代码前移到规格](https://c.m.163.com/news/a/KU0M71K60556C9YG.html)：spec-driven verification解读

**视频**：

- 西门子 Questa One Agentic Toolkit 介绍（西门子官网）

---

## 八、总结

1. **AI在验证领域的落地正在加速**：2025-2026年，西门子、Cadence、Moores Lab、ChipAgents等相继推出商业工具，开源框架也在快速发展

2. **验证工程师的正确定位**：成为"AI验证工程师的 supervisor"，而不是被替代。AI负责重复性工作，人负责人工判断

3. **务实的切入策略**：先用起来，再嵌入工作流，最后构建个人/团队知识资产。不要一开始就追求大而全的AI方案

4. **关键成功因素**：持续积累对AI能力边界的认知，形成人机协作的默契
