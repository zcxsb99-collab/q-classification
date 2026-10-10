---
name: q-classification
description: >-
  小初高数学题库智能分类管理：解析 Word、PDF、图片中的数学题目（纯文本提取、公式转 LaTeX/MathType、
  几何与函数图形识别重绘），按「教材版本-年级-册-章节」「题型」「知识点/考点」三维自动分类、
  去重归档、增量更新、检索与批量导出（DOCX/PDF/Markdown/JSONL/XLSX）。当用户上传数学试卷、
  练习题、真题、模拟题或题目截图（.docx/.pdf/.png/.jpg 等），要求识别题目、整理题库、按教材章节
  或题型归类、标注知识点考点、组卷或导出题库时使用。适配小学、初中的人教版与北师大版，
  高中的人教版与湘教版，服务家教备课、同步训练、分层教学与真题汇编场景。
license: MIT
---

# Q-classification 数学题库智能分类管理

## 概述

Q-classification 面向数学家教与备课场景，把零散的 Word、PDF、图片题目素材加工为结构化、层级化的
专属题库。处理管线为五个阶段：**素材接入 → 智能解析 → 三维分类 → 归档入库 → 检索导出**。
题库可随时增量扩充，分类框架可自定义微调。

## 适用教材范围与编码

| 学段 stage | 版本 publisher（编码） | 年级 grade | 册次标识 |
| --- | --- | --- | --- |
| primary 小学 | pep 人教版、bnu 北师大版 | 1–6 | volume: up 上册 / down 下册 |
| junior 初中 | pep 人教版、bnu 北师大版 | 7–9 | volume: up / down |
| senior 高中 | pep 人教版、hnep 湘教版 | 10–12 | track: compulsory 必修 / elective 选择性必修；book: 1/2/3 |

完整章节、知识点与题型目录见 `references/knowledge-framework.md`，分类时**必须**对照该文件取值，
不得自造章节名称。

## 标准工作流

1. **素材接入**：登记来源文件名、格式、页码/题号、上传时间；支持多文件批量处理。
2. **智能解析**：按「文字 / 公式 / 图形」三条通道解析并质检（见阶段一）。
3. **三维分类**：为每题打教材、题型、知识点标签并给出置信度（见阶段二）。
4. **归档入库**：去重、写记录、更新索引（见阶段三）。
5. **检索导出**：按筛选条件检索并生成导出包（见阶段五）。

每批处理结束后输出处理报告：来源文件、题目总数、成功入库数、重复数、待复核数与分类分布。

## 阶段一：多格式智能解析

### 1. 文字精准提取

- 完整保留题干、已知条件、小问与设问，保留题号层级（一、1、(1)、①）。
- 剔除水印、页眉页脚、边框装饰文字、广告与背景干扰文字；无法确认是否为题目内容的文字不写入题干。
- 素材若含答案或解析，分别写入 `answer`、`analysis` 字段，**不得**混入题干。
- 一道大题含多个小问时，拆分为 `sub_questions`，公共条件保留在题干。

### 2. 公式识别与转换

- 题目中全部数学公式、符号、算式输出为 LaTeX：行内公式 `$...$`，独立公式 `$$...$$`。
- 标准化要求：变量斜体、函数名用正体命令（`\sin`、`\log`、`\lim` 等）、中文不进入公式、
  公式编号保留；分式、根式、指数、矩阵、分段函数、求和求积、上下标不得错乱或丢失。
- MathType 兼容：MathType 支持直接导入/粘贴 LaTeX（「预置 → 工作区参数 → 允许从键盘输入 TeX」，
  或使用「切换 TeX」）；导出 DOCX 时将公式转为 Word 原生 OMML，MathType 可再行转换。
- 推荐工具路径（按需选用，不强制）：
  - **.docx**：优先 `pandoc input.docx -f docx -t markdown`，文档中的 OMML 公式会自动转为 LaTeX；
    或解析 `word/document.xml` 中的 `m:oMath` 节点。
  - **PDF（文本型）**：`pdftotext -layout input.pdf out.txt`；**PDF（扫描型）**：先用
    `pdftoppm -png input.pdf page` 渲染为图片，再逐页视觉识别。
  - **图片**：直接视觉识别，按区域分别读取文字、公式与图形。

### 3. 图形识别与重绘

- 识别几何图形、函数图像、坐标系、统计图、立体几何模型等全部配图。
- 重绘为 **SVG（首选，可编辑、无损）**，需要位图时同时导出 PNG；还原原图尺寸、比例、角度、
  坐标、刻度、点线字母标注，保留辅助线、直角标记与「已知/求证」标记。
- 函数图像须标注坐标轴、原点、刻度与关键点（交点、顶点、极值点等）。
- 图形统一存放为 `library/figures/<question_id>/fig-<n>.svg`，题目记录中引用相对路径。

### 4. 解析质检清单

逐题核对：文字无残缺且无水印杂字；公式可正常渲染并与原图一致；图形比例、标注齐全；
小问数量与原图一致。任一项不通过，标记 `status: needs_review` 并注明具体问题。

## 阶段二：三维智能分类

每题输出三个维度的标签，每个标签附带 `confidence`（0–1）。

### 维度一：教材维度

字段：`stage / publisher / grade / volume（或 track + book）/ chapter / section`，
取值严格对照 `references/knowledge-framework.md`。题目通常不自带版本信息：优先使用用户指定的
默认教材版本；无法判定时对应字段置空并在报告中列出，**禁止臆造版本与章节**。

### 维度二：题型维度

受控词表（可通过自定义配置扩展）：

- 作答形式：选择题（单选/多选）、填空题、判断题、计算题、证明题、解答题、作图题、应用题、
  阅读理解题、综合题、变式题
- 定位标签：基础题、中档题、压轴题、拔高题、易错题、真题（中考/高考/会考）、模拟题
- 难度 `difficulty`：1（基础）–5（压轴）

### 维度三：知识点与考点

- `knowledge_points`：命中的知识点，可多个，**主知识点排第一**。
- `exam_points`：考点标签；标注考查要求层级（了解 / 理解 / 掌握 / 综合运用）。
- `exam_scope`：同步、中考、高考、竞赛等适用范围。

### 置信度与歧义处理

- 置信度 ≥ 0.85：自动通过；0.60–0.85：给出候选标签并标记 `needs_review`；
  < 0.60：不自动归档，进入待复核清单。
- 跨章节综合题：`chapter` 取主载体章节，其余章节写入 `cross_chapters`。

## 阶段三：归档与题库结构

### 题目记录格式

每题为一个 JSON 对象，入库时追加到 `library/questions.jsonl`（每行一题）：

```json
{
  "id": "q-20261010-001",
  "content_hash": "sha1:....",
  "source": {
    "file_name": "2025成都中考数学.pdf",
    "format": "pdf",
    "locator": "第3页 第18题",
    "added_at": "2026-10-10T18:00:00+08:00"
  },
  "stem": {
    "text": "已知二次函数图象经过点 A(1,0)、B(0,-3)，且对称轴为直线 x=2。",
    "latex": "已知二次函数图象经过点 $A(1,0)$、$B(0,-3)$，且对称轴为直线 $x=2$。",
    "formulas": ["A(1,0)", "B(0,-3)", "x=2"],
    "figures": ["figures/q-20261010-001/fig-1.svg"]
  },
  "sub_questions": [
    {"index": "(1)", "text": "求该二次函数的表达式；"},
    {"index": "(2)", "text": "求图象与 x 轴的另一个交点坐标。"}
  ],
  "answer": null,
  "analysis": null,
  "classification": {
    "textbook": {
      "stage": "junior",
      "publisher": "pep",
      "grade": 9,
      "volume": "up",
      "chapter": "第二十二章 二次函数",
      "section": "22.1 二次函数的图象和性质",
      "confidence": 0.9
    },
    "question_types": [
      {"name": "解答题", "confidence": 0.95},
      {"name": "中考真题", "confidence": 0.88}
    ],
    "difficulty": 3,
    "knowledge_points": [
      {"name": "待定系数法求二次函数解析式", "level": "掌握", "primary": true},
      {"name": "二次函数图象与坐标轴交点", "level": "理解"}
    ],
    "exam_points": ["二次函数解析式", "抛物线与 x 轴交点"],
    "cross_chapters": []
  },
  "custom_tags": [],
  "status": "verified",
  "duplicate_of": null
}
```

### 存储约定

```text
library/
  questions.jsonl          # 全部题目记录，增量追加
  index.json               # 教材/题型/知识点聚合索引，入库后更新
  taxonomy.override.yaml   # 用户自定义分类与框架微调
  figures/<question_id>/   # 重绘的 SVG/PNG 图形
  exports/                 # 导出产物
```

- 「教材版本 → 年级册 → 章节 → 题型板块」的层级视图由 `index.json` 生成；每题物理上只存一份。
- **去重**：以「归一化题干文本 + 公式序列」生成 `content_hash`；命中已存在题目时标记
  `duplicate_of` 并保留新来源，不重复入库。

## 阶段四：增量更新与自定义编辑

1. **动态增量更新**：随时上传新试卷、练习题、真题、模拟题、专项习题，重走解析→分类→归档管线，
   报告新增、重复、待复核数量。
2. **自定义题型与标签**：在 `library/taxonomy.override.yaml` 中新增题型、题型子类、知识点标签、
   考点标签，自定义项标记 `source: custom`，不修改内置框架。
3. **知识框架微调**：通过 `chapter_alias`、`move_knowledge_point`、`split_chapter` 等规则微调
   章节结构与知识点划分，记录调整原因，适配同步教学、专项培优与冲刺刷题。

`taxonomy.override.yaml` 示例：

```yaml
question_types:
  add:
    - name: 动点问题
      parent: 综合题
      source: custom
    - name: 新定义问题
      parent: 阅读理解题
knowledge_tags:
  add:
    - "隐圆模型"
    - "二次函数区间最值"
framework:
  chapter_alias:
    - { publisher: bnu, chapter: "第三章 位置与坐标", alias: "平面直角坐标系" }
  move_knowledge_point:
    - { point: "反比例函数综合", from_chapter: "第六章 反比例函数",
        to_chapter: "第二章 二次函数", reason: "家教冲刺阶段合并复习" }
```

## 阶段五：检索与导出

### 检索调用

按任意字段组合过滤：学段、版本、年级、册次、章节、知识点、题型、难度、标签；返回题目卡片列表
与各分组计数，可直接查看、调用任意章节或题型的全部题目。

### 批量导出

- 按筛选结果一键导出同一板块、同一类型的汇总题目，支持：
  - **DOCX**：可编辑，公式为 Word 原生 OMML（MathType 可转换），图形嵌入；
  - **PDF**：排版定稿；
  - **Markdown / LaTeX**：便于二次编辑与讲义制作；
  - **JSONL**：结构化数据，供程序处理；
  - **XLSX**：题目清单与双向细目表。
- 导出选项：连续编号、自动目录、学生版（无答案）/ 教师版（含答案解析）、按题型分节。
- DOCX 导出示例：`pandoc selected.md -o 二次函数专题.docx --mathml`
  （LaTeX 公式自动转为 Word 原生公式）。
- 导出前校验：题目数与筛选结果一致、图形链接有效、公式可渲染；校验通过再交付。

## 沟通与质量规范

- 每批处理后以表格给出处理报告，待复核项说明原因与候选标签，不静默丢弃任何题目。
- 全程使用中文；文件与字段命名稳定；不修改、不覆盖用户原始素材。
- 教材换版（如 2024 修订版逐年启用）时，先确认用户使用的教材年份版本；框架差异通过
  `taxonomy.override.yaml` 修正，并在报告中注明。

## 参考资源

- `references/knowledge-framework.md`：六套教材（小学人教/北师大、初中人教/北师大、
  高中人教/湘教版）的章节知识框架、知识点与题型目录，并标注教材版本年份。分类时按
  「学段 → 版本 → 册 → 章」检索读取。
