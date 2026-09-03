# math-modeling Skill 架构文档

> 基于 `math-modeling-skill-main` 项目全量源码整理

---

## 1. 总体架构

### 1.1 核心设计理念

| 维度 | 设计决策 |
|------|----------|
| **角色分工** | 三阶段流水线：建模手 → 编程手 → 论文手 |
| **交付物边界** | 所有产物仅为**草稿/参考**，不可直接提交参赛 |
| **质量门禁** | 5个强制Subagent质检：M1/P1/P2/W1/W2 |
| **反馈闭环** | 跨阶段双向回退，禁止编造结果 |
| **路径隔离** | `SKILL_ROOT`（只读）↔ `PROJECT_ROOT`（读写） |

### 1.2 目录结构

```
math-modeling-skill-main/
├── SKILL.md                          # 主入口，定义三角色路由
├── 使用指南.md                        # 交付物边界与使用须知
├── CHANGELOG.md
├── README.md
├── references/
│   ├── Subagent调度.md               # 质检门禁与协作规则
│   ├── 算法索引.md                   # 7大类算法文件映射
│   ├── README.md                     # 导航索引
│   └── roles/
│       ├── 建模手/
│       │   ├── SKILL.md              # 角色入口
│       │   └── references/           # 5个参考文档
│       ├── 编程手/
│       │   ├── SKILL.md
│       │   └── references/           # 3个参考文档
│       └── 论文手/
│           ├── SKILL.md
│           └── references/           # 7个参考文档
├── tools/                            # 6大工具链
│   ├── pdf/SKILL.md                  # PDF读写/OCR/表格提取
│   ├── xlsx/SKILL.md                 # Excel读写/公式重算
│   ├── paper_search/SKILL.md         # 双引擎论文搜索
│   ├── latex/SKILL.md                # LaTeX项目构建/编译/校验
│   ├── docx/SKILL.md                 # Word构建/转换/校验
│   └── figure/SKILL.md               # 科研可视化全流程
├── assets/                           # 7类算法说明文档
│   ├── 01-优化算法说明.md
│   ├── 02-预测类算法说明.md
│   ├── 03-评价类算法说明.md
│   ├── 04-图论与网络分析算法说明.md
│   ├── 05-统计分析与数据处理算法说明.md
│   ├── 06-综合类算法说明.md
│   └── 07-机器学习算法说明.md
└── dsh-plugin/                       # DSH框架集成副本
```

---

## 2. 三角色详细设计

### 2.1 建模手

**入口**: `references/roles/建模手/SKILL.md`

| 维度 | 规范 |
|------|------|
| **固定产物** | `题目分析报告.md`、`术语表格.md` |
| **前置合同** | 8项表格：任务定义、数据、结论类型、模型、数学定义、实现、验证、风险 |
| **模型数量限制** | 每子问题最多2个独立模型体系；物理题按模型族计数 |
| **质检门禁** | **M1** - 建模终检（子问题覆盖、公式符号、单位约束、假设依据、可实现性、验证方案、引用追溯） |
| **核心参考** | `工作流程.md`、`前置合同.md`、`建模设计理论.md`、`常见模式.md`、`质检清单.md` |

**执行流程**:
1. 读题+附件 → 建立问题/目标/约束/数据/输出清单
2. 数据驱动检查缺失/异常/量纲/时空范围
3. 先定结论类型，再评估候选模型
4. 搜索文献依据（双引擎：OpenAlex + AnySearch）
5. 写入两个固定产物 → 作者自检 → **派发M1 Subagent**
6. M1 PASS后才能进入编程阶段

---

### 2.2 编程手

**入口**: `references/roles/编程手/SKILL.md`

| 维度 | 规范 |
|------|------|
| **固定产物** | `.py`/`.m`代码、`results/*.csv`/`.xlsx`、≥9张图表、`results/复现清单.json` |
| **图表要求** | 3类×≥3张=≥9张；每子问题在3类中各≥1张；命名`raw_qN_*`/`process_qN_*`/`result_qN_*` |
| **语言选择** | Python/MATLAB/双语言；动态检查依赖（`check_env.py`/`check_matlab_env.m`） |
| **求解稳健性** | 9项检查：数值稳定、无量纲化、矩阵病态、优化器选型、随机性、收敛控制、数据规模、Sanity Check、跨平台复现 |
| **质检门禁** | **P1**最小可运行结果（纵向切片）、**P2**编程终检（独立复现+哈希+图表语义+文件完整性） |
| **核心参考** | `工作流程.md`、`MATLAB规范.md`、`质检清单.md` |

**执行流程**:
1. 读取建模手产物+题目附件
2. 选语言+动态检查依赖
3. 实现核心求解链 → 小实例跑通
4. **派发P1 Subagent** → PASS后才能全量计算+正式出图
4. 可视化全流程（数据剖析→契约→选图→绘制→自检→导出→审计）
5. 生成复现清单 → 作者自检 → **派发P2 Subagent**
6. P2 PASS后才能进入论文阶段

---

### 2.3 论文手

**入口**: `references/roles/论文手/SKILL.md`

| 维度 | 规范 |
|------|------|
| **默认交付** | `完整论文.docx`（Word） |
| **可选交付** | `完整论文-LaTeX/` + `完整论文.pdf`（用户显式要求） |
| **双格式一致性** | 正文/数据/图表/结论必须一致 |
| **官方规则优先级** | 用户提供 > 官网获取 > 内置基线（不替代官方） |
| **图表基线** | 默认≥8幅正式图；每子问题≥1幅；编号/题注/引用齐全 |
| **质检门禁** | **W1**证据大纲（主张-证据映射）、**W2**论文终检（规则/主张-证据/数值/图表/文献/渲染） |
| **完成门禁命令** | Word: `paper_format.py validate` + `office/validate.py` + `equations.py verify-conversion`<br>LaTeX: `latex_paper.py doctor/build/validate` |
| **核心参考** | `工作流程.md`、`章节模板.md`、`论文格式规范.md`、`写作规范.md`、`自审框架.md`、`LaTeX格式规范.md`、`英文化工作流.md` |

**执行流程**:
1. **锁定官方规则**（竞赛/届次/语言/模板/来源）
2. **检查输入**（题目/建模产物/代码/结果/图表/文献）——缺一不可
3. 建立主张-证据映射 → **派发W1 Subagent**
4. W1 PASS后构建论文（先锁定共同正文，再双格式生成）
5. 运行确定性门禁命令（退出码必须为0）
6. **派发W2 Subagent** → PASS后交付

---

## 3. Subagent 质检体系

### 3.1 5大固定门禁

| 门禁 | 阶段 | 派发时机 | 核心验收点 | 通过后解锁 |
|------|------|----------|------------|------------|
| **M1** | 建模 | 两产物自检后 | 子问题覆盖、公式符号、单位约束、假设、模型数量、可实现性、验证、文献 | 进入编程/交付建模 |
| **P1** | 编程 | 核心求解链首次跑通后 | 最小命令执行、退出码、输入追溯、单位/范围/约束、模型合同 | 全量计算/正式出图 |
| **P2** | 编程 | 代码/结果/图表/清单冻结后 | 独立复现、输入哈希、种子/参数/数值/边界/量纲、图表语义、文件完整 | 进入论文/交付编程 |
| **W1** | 论文 | 主张-证据映射完成后 | 每结论有证据路径、摘要数值一致、公式/图表/引用有章节落点 | 开始长篇正文/排版 |
| **W2** | 论文 | 所有格式门禁通过后 | 当届规则、主张-证据、数值单位、图表引用、文献、渲染效果、双格式一致 | 交付论文/宣称完成 |

### 3.2 质检铁律

1. **禁止事后补检** —— 必须在规定节点立即派发
2. **三权分立** —— 作者自检 ≠ 脚本校验 ≠ 独立验收
3. **只读质检** —— Subagent不修改权威产物，只返回带证据问题
4. **实质变化即失效** —— 被审产物变化，原PASS立即失效
5. **环境无Subagent时** —— 只能报告 `BLOCKED` 或受限交付，不得声称完整完成

### 3.3 固定回执格式

```markdown
范围: 本次检查的门禁与未覆盖内容
输入快照: 路径、版本或 SHA-256
状态: PASS | FAIL | BLOCKED
证据: 来源URL、命令、退出码、文件位置、页码、关键数值
发现: P0(正确性/硬约束) / P1(复现/一致性/关键质量) / P2(非阻塞改进)
返工: 回到角色、必须修正项、复验入口
```

### 3.4 可选协作（默认关闭）

| 任务 | 触发条件 | 输出 | 边界 |
|------|----------|------|------|
| 官方规则核验 | 竞赛/届次/模板未核验 | 官方URL、硬约束、模板路径哈希 | 可与附件盘点并行；论文构建须等待 |
| 数据附件盘点 | 多附件/复杂PDF/字段多 | 文件哈希、工作表、字段、缺失异常 | 只读；模型选择等关键数据事实 |
| 文献调研 | 主张/关键词/模型族明确 | 题名/作者/DOI/理论/适用条件/风险 | 按主张簇拆分；模型取舍仍由主Agent |
| 隔离算法原型 | 验证可实现性/接口/复杂度 | 最小代码、命令、输出、限制 | 不修改权威代码；主Agent决定整合 |
| 独立实验批次 | 模型/指标/参数冻结、互不依赖 | 参数/种子/命令/结果/耗时/失败 | 并行敏感性/消融/边界实验 |
| Python/MATLAB对照 | 要求双语言/跨语言一致性 | 隔离实现、命令、环境、差异 | 同一模型合同；主Agent统一权威结果 |
| 术语英文核验 | 术语多/生成英文论文 | 术语对照、冲突、建议位置 | 只建议，不写论文 |

---

## 4. 工具链体系

### 4.1 工具加载策略（渐进式）

| 任务 | 读取入口 |
|------|----------|
| 选模型/查算法 | `references/算法索引.md` → 匹配 `assets/*.md` |
| 搜索论文 | `tools/paper_search/SKILL.md` |
| 读取题目PDF | `tools/pdf/SKILL.md` |
| 处理Excel | `tools/xlsx/SKILL.md` |
| 画图 | `tools/figure/SKILL.md` |
| 生成Word | `tools/docx/SKILL.md` |
| 生成LaTeX | `tools/latex/SKILL.md` |
| LaTeX转Word | `tools/docx/SKILL.md` (convert-latex) |
| 派发Subagent | `references/Subagent调度.md` |

---

### 4.2 6大工具详细能力

#### PDF工具 (`tools/pdf/SKILL.md`)
- **库**: pypdf(合并/拆分/旋转/元数据)、pdfplumber(文本/表格提取)、reportlab(创建PDF)、pytesseract+pdf2image(OCR)
- **CLI**: pdftotext、qpdf、pdftk
- **核心**: 表格提取→DataFrame→Excel、水印、图片提取、密码保护

#### Excel工具 (`tools/xlsx/SKILL.md`)
- **原则**: 输入只读、优先CSV、仅题目要求XLSX/保留公式/多Sheet/模板时用XLSX
- **读取**: 禁止隐式`header=0`；首行是数据必须用`header=None`或`read_excel_rows(header=False)`
- **公式重算**: `scripts/recalc.py` 调用LibreOffice隔离重算，原子替换，超时保留原文件
- **校验**: 公式引用/范围/类型/单位/表头声明/首末行/预期行数/错误值(`#VALUE!`等)

#### 双引擎论文搜索 (`tools/paper_search/SKILL.md`)
- **数据源**: OpenAlex(结构化元数据) + AnySearch Academic(垂直搜索)
- **融合**: DOI交叉验证；标题高度相似+年份相容时合并；预印本折叠保留更完整记录
- **排序**: 查询词覆盖率优先于引用量；多术语时要求≥2个有效查询词命中
- **核验**: 引用前必须打开DOI/出版页面核对元数据

#### LaTeX工具 (`tools/latex/SKILL.md`)
- **模板优先级**: 官方模板 > 官网模板 > 内置基线(`assets/templates/cumcm|mcm-icm/`)
- **初始化**: `latex_paper.py init` 复制模板目录，生成`latex-project.json`(哈希/版本/资源绑定)
- **环境诊断**: `doctor` 检查latexmk/引擎/BibTeX-Biber/pypdf/pdfimages/pdftoppm/Pandoc
- **资源绑定**: `bind` 建立代码/图表哈希绑定，防止漂移
- **编译**: `build` 优先latexmk，临时目录编译，哈希比对，`--allow-warning`需精确正则+理由
- **校验**: `validate` 检查摘要/公式/图表/页数/label重复/引用/哈希绑定/空白页/字体嵌入/DPI
- **转Word**: 调用`docx/equations.py convert-latex`

#### DOCX工具 (`tools/docx/SKILL.md`)
- **推荐流程**: 官方模板 + python-docx + OMML公式 + OOXML校验 + 渲染抽检
- **公式**: `equations.py` LaTeX子集→OMML；复杂公式用Pandoc转换
- **LaTeX转Word**: `convert-latex` 需Pandoc，citeproc处理BibTeX，生成`.conversion.json`
- **OOXML底层**: `scripts/office/` 解包/校验/重打包（共用XLSX）
- **修订/批注**: 隔离LibreOffice配置接受修订、添加批注
- **必做验证**: `check_env.py` + `self_check.py` + `office/validate.py` + `paper_format.py validate` + `equations.py verify-conversion`

#### 科研可视化工具 (`tools/figure/SKILL.md`)
- **核心原则**: 先思考再画 / 先理解数据再选图 / 先想清核心结论 / 主动拦截经典错误 / 维度过多建议拆图
- **9步工作流**: 数据剖析 → 图表契约(核心结论/证据链/版型/后端/期刊规范) → 选图决策 → 期刊规范 → 配环境 → 绘制 → 三层自检(语义/形式/视觉) → 导出(SVG+300DPI PNG) → 文件审计
- **数学建模专用**: 三类图体系(raw/process/result)，每类≥3张、合计≥9张，每子问题三类各≥1张，图型≥3种
- **主动拦截**: n<10画均值柱→箱线+stripplot；双Y轴无关变量→拆上下子图；饼图→横向柱状；3D柱/饼→2D/热力图；Y轴非零起→从0起或log；分类用折线→散点/柱状；rainbow/jet→viridis/magma/RdBu_r
- **双审计**: `check_figure.py`(单图格式) + `figure_audit.py`(三类数量+子问题覆盖，P2时由Subagent执行)
- **参考文档体系**: 选图决策/配方模板/自检合规/设计布局/数据工具 分类按需加载

---

## 5. 算法知识库

### 5.1 7大类算法映射 (`references/算法索引.md`)

| 问题特征 | 参考文件 |
|----------|----------|
| 约束优化、调度、路径、资源分配 | `assets/01-优化算法说明.md` |
| 时间序列、回归预测、灰色预测 | `assets/02-预测类算法说明.md` |
| 多指标评价、排序、效率分析 | `assets/03-评价类算法说明.md` |
| 最短路、网络流、复杂网络 | `assets/04-图论与网络分析算法说明.md` |
| 检验、降维、聚类前处理、统计推断 | `assets/05-统计分析与数据处理算法说明.md` |
| 模拟、系统动力学、元胞自动机 | `assets/06-综合类算法说明.md` |
| 分类、聚类、集成学习、神经网络 | `assets/07-机器学习算法说明.md` |

### 5.2 建模常见模式 (`references/roles/建模手/references/常见模式.md`)

| 类型 | 模式 | 核心组合 |
|------|------|----------|
| 优化 | 模式1 | 线性/整数规划 → 灵敏度分析 |
| 优化 | 模式2 | 启发式(GA/SA/PSO) → 对比验证 |
| 优化 | 模式3 | 多目标(NSGA-II) → Pareto分析+TOPSIS |
| 预测 | 模式4 | 单一预测(GM/回归/指数平滑) → 误差检验 |
| 预测 | 模式5 | 双模型对比/集成(ARIMA/LSTM/XGBoost) |
| 预测 | 模式6 | 串联(聚类→回归、STL→ARIMA) |
| 评价 | 模式7 | 赋权(AHP/熵权) + 综合评价(TOPSIS/灰色关联/模糊) |
| 评价 | 模式8 | DEA效率评价(CCR/BCC) + 投影+规模收益 |
| 综合 | 模式9 | 优化 + 模拟(蒙特卡洛场景→优化→统计风险指标) |
| 综合 | 模式10 | 评价 + 优化(先评后优) |
| 综合 | 模式11 | 预测 + 决策(先预测再优化) |

**决策树**: 按核心类型(优化/预测/评价/组合) → 细分条件(随机性/规模/数据量/精度要求) → 选模式

---

## 6. 竞赛适配规范

### 6.1 CUMCM (全国大学生数学建模竞赛)

| 项目 | 规范 |
|------|------|
| **题型** | A(连续/机理)、B(离散/优化)、C(数据/评价) |
| **时间** | 72小时 |
| **摘要** | 五段式(背景→问题→方法→结果→创新)，600-900字 |
| **假设** | 3-7条（关键≤3，可放宽≤7），分级标注 |
| **符号表** | 必须有单位列 |
| **模型数量** | 每子问题最多2个独立模型体系 |
| **页数** | 2026官方：正文≤30页（无最低页数/字数要求） |
| **质量目标** | 工具默认~15000字词单位、~20页（可被官方规则/用户覆盖） |
| **图表** | 默认≥8幅正式图 |

### 6.2 MCM/ICM (美赛/数模竞赛)

| 项目 | 规范 |
|------|------|
| **语言** | 强制英文 |
| **时间** | 96小时 |
| **页数** | 25页硬上限（附录代码不计页数但正文须引用） |
| **摘要** | 1页Summary（权重极高） |
| **ICM特有** | F题必写Letter（2页给决策者的信） |
| **评分维度** | Communication权重高（排版/图表/叙事清晰度） |
| **数据来源** | 常需引用真实数据集(UN/World Bank等) |
| **英文化工作流** | 三阶段：理解(中文→论证结构) → 写作(论证结构→英文草稿) → 校准(论证/语言/动词三轮) |

---

## 7. 关键文件路径速查

### 7.1 角色入口
- 建模手: `references/roles/建模手/SKILL.md`
- 编程手: `references/roles/编程手/SKILL.md`
- 论文手: `references/roles/论文手/SKILL.md`

### 7.2 核心调度与索引
- Subagent调度: `references/Subagent调度.md`
- 算法索引: `references/算法索引.md`

### 7.3 工具入口
- PDF: `tools/pdf/SKILL.md`
- Excel: `tools/xlsx/SKILL.md`
- 论文搜索: `tools/paper_search/SKILL.md`
- LaTeX: `tools/latex/SKILL.md`
- DOCX: `tools/docx/SKILL.md`
- 可视化: `tools/figure/SKILL.md`

### 7.4 执行脚本路径模板
```
<SKILL_ROOT>/tools/figure/scripts/profile_data.py
<SKILL_ROOT>/tools/figure/scripts/export_figure.py
<SKILL_ROOT>/tools/figure/scripts/check_figure.py
<SKILL_ROOT>/references/roles/编程手/scripts/figure_audit.py
<SKILL_ROOT>/tools/docx/scripts/paper_format.py
<SKILL_ROOT>/tools/docx/scripts/equations.py
<SKILL_ROOT>/tools/docx/scripts/office/validate.py
<SKILL_ROOT>/tools/latex/scripts/latex_paper.py
<SKILL_ROOT>/tools/xlsx/scripts/recalc.py
<SKILL_ROOT>/references/roles/编程手/scripts/repro_manifest.py
<SKILL_ROOT>/references/roles/编程手/scripts/check_env.py
<SKILL_ROOT>/references/roles/编程手/scripts/apply_publication_style.m
```

---

## 8. 交付物清单总览

### 8.1 按阶段

| 阶段 | 产物 | 位置 | 说明 |
|------|------|------|------|
| **建模手** | `题目分析报告.md` | `PROJECT_ROOT/` | 问题理解、模型选择、分析 |
| | `术语表格.md` | `PROJECT_ROOT/` | 术语/符号/单位统一 |
| **编程手** | `问题N_求解.py/.m` | `PROJECT_ROOT/` | 模型求解实现 |
| | `results/问题N_结果.csv/.xlsx` | `PROJECT_ROOT/results/` | 求解结果表格 |
| | `figures/raw_qN_*` | `PROJECT_ROOT/figures/` | 原始数据图(每子问题≥1) |
| | `figures/process_qN_*` | `PROJECT_ROOT/figures/` | 过程图(每子问题≥1) |
| | `figures/result_qN_*` | `PROJECT_ROOT/figures/` | 结果图(每子问题≥1) |
| | `results/复现清单.json` | `PROJECT_ROOT/results/` | 种子/参数/命令/依赖版本 |
| **论文手** | `完整论文.docx` | `PROJECT_ROOT/` | Word论文草稿 |
| | `完整论文-LaTeX/` | `PROJECT_ROOT/` | LaTeX源码项目(可选) |
| | `完整论文.pdf` | `PROJECT_ROOT/` | LaTeX编译PDF(可选) |
| | `.conversion.json` | `PROJECT_ROOT/` | LaTeX→Word转换哈希绑定 |
| | `.build.json` | `PROJECT_ROOT/完整论文-LaTeX/` | LaTeX编译哈希绑定 |

### 8.2 文件命名约定
- 图表: `raw_q{N}_{描述}.svg/png`、`process_q{N}_{描述}.svg/png`、`result_q{N}_{描述}.svg/png`
- 代码: `问题{N}_求解.py` 或 `问题{N}_求解.m`
- 结果: `问题{N}_结果.csv` 或题目指定的 `.xlsx`

---

## 9. 完成判定标准

仅当**全部**满足时才能宣称完整完成：

- [ ] 强制执行协议全部满足，提供可复核门禁结果
- [ ] 所有涉及的独立Subagent门禁均为 `PASS`，且通过后产物未发生未经复验的实质变化
- [ ] 缺少Subagent能力时只能报告 `BLOCKED` 或受限交付
- [ ] 所有计算结论来自实际运行结果
- [ ] 公式、表格、图表与代码结果一致
- [ ] 引用可由OpenAlex、AnySearch或原始出版页面追溯
- [ ] 论文按目标竞赛当届官方规则配置，篇幅目标已确认
- [ ] 公式/非空图表数量、全部子问题图覆盖已检查
- [ ] 图表编号与正文引用连续
- [ ] 参考文献与正文引用双向对应
- [ ] Word分支：通过原生OMML、DOCX结构、转换警告、渲染页数检查
- [ ] LaTeX分支：通过环境诊断、真实编译、日志、哈希绑定、正文/附录页数、空白页、页面尺寸、字体嵌入、图片DPI检查
- [ ] 所有产物位于 `PROJECT_ROOT`，`SKILL_ROOT` 未被改写

---

## 10. 使用入口与路由

| 用户意图 | 加载入口 | 是否要求前一阶段完成 |
|----------|----------|---------------------|
| 完整建模、完成整题 | 建模手 → 编程手 → 论文手 | 按顺序执行 |
| 只做题目分析、选模型 | 建模手 | 否 |
| 只写代码、跑结果、出图 | 编程手 | 需题目和可执行模型说明；缺失先补齐 |
| 只写或修改论文 | 论文手 | 需题目、模型、真实结果、图表；缺失回退对应阶段 |

**单阶段任务不强制执行完整流程**。

---

*文档生成基于 math-modeling-skill-main 完整源码分析*