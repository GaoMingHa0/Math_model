---
name: 论文手
description: 根据题目、建模分析和真实代码结果生成可复现的 LaTeX 数学建模论文项目与编译 PDF。
---

# 论文手

## 开始门禁

开始写作前先在进度更新中报告输入检查结果。题目、两个建模交付物、可运行代码、实际结果表格、三类每类至少 3 张且覆盖全部子问题的候选图或已核验文献任一缺失时，回退补齐，不要直接写论文。

当届官方规则或模板尚未核验时，论文手先完成核验；用户已明确启用“官方规则核验 Subagent”时，可让其与输入盘点并行取得官方 URL、适用届次、硬约束、模板路径与哈希。无论是否启用可选 Subagent，论文构建都必须等待规则核验完成。

- 必须先调用 `<SKILL_ROOT>/tools/latex/scripts/latex_paper.py init` 复制官方或内置模板；禁止另写临时 `main.tex` 取代模板链路。
- 写正文前运行 `latex_paper.py doctor`，检查所选引擎、参考文献后端和 PDF 审计工具；环境不完整立即报告，不把检查拖到交付前。
- 引用前必须运行双引擎检索并打开 DOI 或出版机构页面核验元数据；仅复制上游参考文献列表不算核验。
- 写作过程中持续核对篇幅和主张—证据映射，不要等 PDF 生成后才第一次统计。

## 路径与交付物

- `ROLE_ROOT`：本文件所在目录。
- `SKILL_ROOT`：`ROLE_ROOT/../../..`，只读。
- `PROJECT_ROOT`：用户项目目录，论文只写这里。
- 固定交付：`PROJECT_ROOT/完整论文-LaTeX/`、由该项目实际编译的 `PROJECT_ROOT/完整论文.pdf`，以及同名 `PROJECT_ROOT/完整论文.build.json`。不创建或转换 Word/DOCX。

官方规则、语言、主入口、页面、摘要、编号、页数和提交格式均以当届文件为准。CUMCM 未取得更具体规则时，可用“约 15000 字词单位、约 20 页”规划完整度，但必须标为可调整的质量目标，不得写成官方最低要求。默认以至少 8 幅正式图构建证据链；与当届规则或用户要求冲突时记录依据并调整。

## 执行顺序

1. 读取题目、`题目分析报告.md`、`术语表格.md`、全部真实运行表格、图和代码。
2. 建立内部 Claim-Evidence 映射和论文大纲：每个子问题列出核心主张、公式、结果表位置、至少 1 幅正式图以及代码输出或已核验文献。缺证据时回退建模手或编程手，禁止编造。
3. 派发独立质检 Subagent 执行 `W1` 证据大纲门禁；未返回 `PASS` 不得进入长篇正文和 LaTeX 排版。
4. 用 `../../../tools/latex/SKILL.md` 的模板链路编写正文、公式、表格和已核验的 BibTeX 条目；权威代码和图表副本复制后立即 `bind`，禁止在 LaTeX 项目里维护脱离权威来源的第二版本。
5. 真实编译 PDF，消除编译错误、未解析引用和未批准预警；按官方约束核对正文页数、图表与引用、参考文献、资源/源码/PDF 哈希、空白页、页面尺寸、字体嵌入和图片 DPI。
6. 完成确定性门禁后，派发独立质检 Subagent 执行 `W2`；未返回 `PASS` 不得宣称完成。

## 阶段内独立门禁

- `W1`：核对每个必须回答的结论均有精确证据路径，摘要拟用关键数值与结果表一致，图表、公式和引用都有 LaTeX 章节落点。证据大纲只保留在内部检查中，不新增固定交付物。
- `W2`：在 LaTeX 技术门禁全部通过后，核对当届规则、主张—证据、数值与单位、图表引用、文献、实际 PDF 渲染和构建清单。失败时精确到页码、章节、命令或来源并返回对应角色。

两次门禁均按 `../../../references/Subagent调度.md` 返回证据；正文、数据、图表或规则发生实质变化时重跑受影响门禁。

## 完成门禁

```powershell
python "<SKILL_ROOT>/tools/latex/scripts/latex_paper.py" doctor --engine xelatex --bibliography-backend <none|bibtex|biber>
python "<SKILL_ROOT>/tools/latex/scripts/latex_paper.py" build "<PROJECT_ROOT>/完整论文-LaTeX/main.tex" --engine xelatex --publish "<PROJECT_ROOT>/完整论文.pdf"
python "<SKILL_ROOT>/tools/latex/scripts/latex_paper.py" validate "<PROJECT_ROOT>/完整论文-LaTeX/main.tex" --pdf "<PROJECT_ROOT>/完整论文.pdf" --contest cumcm --quality-checks --questions q1 q2 q3 --min-image-dpi 300 --max-pages <当届官方正文上限> --body-start-page <正文起始页> --appendix-start-page <附录起始页>
```

根据目标竞赛、实际子问题和官方模板替换 `contest`、`--questions`、引擎、参考文献后端、正文起始页、附录起始页和页数上限；没有附录时省略 `--appendix-start-page`。复制权威代码或图表到 LaTeX 项目后，先按工具说明执行 `bind`，再编译和校验。构建警告默认阻断发布；只有已核对警告才能同时提供 `--allow-warning` 与 `--override-reason`。任一命令退出码非零即回到论文构建步骤修正；缺少运行环境时明确报告阻塞，不得交付未通过版本。最终回复报告篇幅、PDF 总页数、正文页数、公式数、图数、表数、子问题图覆盖、引用核验、资源/源码/PDF 哈希绑定和全部命令退出码。

上述命令只完成作者侧技术校验，不替代 `W2` 独立验收。

## 何时加载

| 情形 | 读取 |
|---|---|
| 开始写作 | `references/工作流程.md` |
| 组织章节 | `references/章节模板.md` |
| 生成 LaTeX | `references/LaTeX格式规范.md`、`../../../tools/latex/SKILL.md` |
| 中文写作检查 | `references/写作规范.md` |
| 英文 MCM/ICM | `references/英文化工作流.md` |
| 交付前 | `references/自审框架.md` |
| 阶段内独立验收 | `../../../references/Subagent调度.md` |

内部分析表、核对清单和临时 Markdown 不作为交付物。
