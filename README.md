# Q-classification

小初高数学题库智能分类管理 **Skill**。面向数学家教与备课场景，自动解析 Word、PDF、图片中的数学题目，
按「教材章节 / 题型 / 知识点考点」三维分类归档，支持增量更新、自定义分类与批量导出，
帮助教师把零散试卷和题目素材变成系统化、标准化、可检索的专属题库。

## 一、项目解决什么问题

家教和一线教师手里通常积累了大量试卷、练习题、真题、模拟题和题目截图，实际使用中存在三个痛点：

1. **素材杂、难录入**：文件格式不一（Word、文本型 PDF、扫描件、手机拍照），常带水印和页眉页脚；
   数学公式、几何图形靠手工录入，费时且容易出错。
2. **题目散、找不到**：题目分散在不同文件里，无法按「教材版本 → 年级册 → 章节 → 题型」快速检索，
   备课时想凑一份同章节、同题型的专项练习，只能逐份翻卷。
3. **题库难维护**：新试卷不断增加，重复题目难以识别；个人惯用的题型分法（如动点问题、结构不良题）
   与教材目录不一致，题库越用越乱。

Q-classification 以 Skill 形式加载到 Agent 中，让 Agent 自动完成「素材识别 → 公式图形标准化 →
三维分类 → 去重归档 → 检索导出」全流程，把零散素材加工为结构化、可增量扩充的专属题库。

## 二、主要功能

- **多格式识别解析**：支持 `.docx` / `.pdf`（文本型与扫描型）/ `.png` / `.jpg`；
  - 文字：完整提取题干、条件、小问与设问，自动剔除水印、页眉页脚等无效文字；
  - 公式：全部公式、符号、算式转为标准 LaTeX，导出 DOCX 时转为 Word 原生 OMML（兼容 MathType）；
  - 图形：几何图形、函数图像、坐标系、统计图、立体模型识别后高清重绘为 SVG，还原比例、角度、坐标与标注。
- **三维智能分类**：
  1. 教材维度：版本、年级、上下册（高中为必修/选择性必修及册号）、章节、小节；
  2. 题型维度：单选/多选、填空、计算、证明、解答、作图、应用、压轴、变式、真题、模拟题等；
  3. 知识点与考点维度：主/次知识点、考查层级（了解/理解/掌握/综合运用）、中考高考属性。
- **自动归档去重**：每题一条 JSON 记录，按「归一化题干 + 公式序列」生成 sha1 指纹自动去重，
  并生成按教材章节、题型、知识点聚合的索引。
- **增量更新与自定义**：随时上传新素材重走处理管线；可在配置文件中新增题型、知识点/考点标签，
  微调章节框架，不改动内置教材体系。
- **检索与批量导出**：按版本、年级、章节、知识点、题型、难度组合筛选，导出 DOCX、PDF、
  Markdown/LaTeX、JSONL、XLSX，支持连续编号、自动目录与学生版（无答案）/教师版（含答案解析）。

适配教材：小学、初中的**人教版、北师大版**；高中的**人教版、湘教版**。

## 三、安装方法

### 环境要求

- 一个支持 **Skill 机制**的 Agent 运行环境（Agent 通过读取 `SKILL.md` 的 frontmatter 自动发现技能）。
- Skill 本身是纯 Markdown / YAML 文件夹，**无需编译、无需安装 Python 包**。
- 解析与导出 DOCX/PDF 时，Agent 会按需调用以下可选外部工具：
  - [pandoc](https://pandoc.org/)：DOCX 解析与 DOCX 导出（公式自动转 OMML）；
  - [Poppler](https://poppler.freedesktop.org/)：`pdftotext`（文本型 PDF）、`pdftoppm`（扫描件渲染）。

### 安装步骤

1. 获取本项目（当前阶段为本地开发，直接复制文件夹即可；尚未初始化 Git）。
2. 将 `skills/Q-classification` **整个目录**复制到 Agent 约定的技能目录，例如环境中的
   `workspace/.user_skills/` 或其他已配置的 skill 根目录。
3. 重启或刷新 Agent 会话，Agent 会根据 `SKILL.md` 中的 `name`、`description` 自动识别该技能。
4. （可选）安装 pandoc、Poppler，以获得完整的 DOCX/PDF 解析与 DOCX 导出能力。

### 验证安装

在 Agent 中直接询问「q-classification 能做什么？」，若能正确描述题目识别、三维分类与导出能力，
即表示安装成功；也可使用 skill-creator-for-work 提供的 `quick_validate.py` 校验目录结构。

## 四、使用方法

1. 用自然语言描述需求并上传素材，Agent 会在匹配任务中自动加载本技能，例如：
   - “把这份试卷里的题目全部识别出来，按人教版九年级上册章节分类建库。”
   - “这 20 张题目截图整理成题库，标注题型和知识点，不确定的列给我确认。”
   - “导出二次函数章节所有解答题，教师版，Word 格式。”
2. 首次处理时告知默认教材版本与年级；未指定时 Agent 不臆测版本，会在报告中列出待确认项。
3. Agent 逐题解析、分类并质检，输出处理报告（来源、入库数、重复数、待复核数与分类分布）；
   置信度不足的题目进入待复核清单，确认后再归档。
4. 题库默认生成在工作目录的 `library/` 下；自定义题型与框架微调写入
   `library/taxonomy.override.yaml`；后续上传的素材增量追加到同一题库。

## 五、输入输出示例

### 示例 1：整卷识别与分类（本项目真实运行）

> **示例数据声明**：`library/` 中收录的深圳二调试题仅用于功能演示与教学用途，试题版权归原命题方所有；完整声明见文末「示例数据声明」。

**输入**：`深圳二调(1).pdf` —— 2026 年深圳市高三年级第二次调研考试数学，4 页纯扫描件
（无文字层，含页眉页脚水印）。指令：“使用 q-classification 把这份题目识别分类。”

**输出**：19 题全部解析入库（单选 8、多选 3、填空 3、解答 5，满分 150），水印已剔除。
其中第 1 题在 `library/questions.jsonl` 中的真实记录为：

```json
{
  "id": "q-20261010-001",
  "content_hash": "sha1:f17ad48ba31d076b029e2a6b6a88e82bf53c09ec",
  "source": {
    "file_name": "深圳二调(1).pdf",
    "format": "pdf",
    "locator": "第1页 第1题",
    "added_at": "2026-10-10T18:00:00+08:00"
  },
  "stem": {
    "text": "已知 z=\\dfrac{2}{1-i}，则 |z|=",
    "latex": "已知 $z=\\dfrac{2}{1-i}$，则 $|z|=$",
    "formulas": ["z=\\dfrac{2}{1-i}", "|z|"],
    "figures": [],
    "options": {
      "A": "$\\sqrt2$",
      "B": "$\\sqrt3$",
      "C": "$2$",
      "D": "$\\sqrt5$"
    }
  },
  "sub_questions": [],
  "answer": null,
  "analysis": null,
  "classification": {
    "textbook": {
      "stage": "senior",
      "publisher": "pep",
      "grade": 12,
      "track": "compulsory",
      "book": 2,
      "chapter": "第七章 复数",
      "section": "7.2 复数的四则运算",
      "confidence": 0.98
    },
    "question_types": [
      {"name": "选择题", "confidence": 0.99},
      {"name": "单选题", "confidence": 0.99},
      {"name": "基础题", "confidence": 0.95},
      {"name": "高考模拟题", "confidence": 0.9}
    ],
    "difficulty": 1,
    "knowledge_points": [
      {"name": "复数代数形式的乘除运算", "level": "掌握", "primary": true},
      {"name": "复数的模", "level": "理解"}
    ],
    "exam_points": ["复数代数形式的乘除运算", "复数的模"],
    "cross_chapters": []
  },
  "custom_tags": [],
  "status": "verified",
  "duplicate_of": null
}
```

处理报告（真实结果）：

| 来源 | 题目数 | 入库 | 重复 | 待复核 |
| --- | --- | --- | --- | --- |
| 深圳二调(1).pdf | 19 | 19 | 0 | 0 |

难度分布为 1 档 1 题、2 档 4 题、3 档 6 题、4 档 6 题、5 档 2 题；19 题覆盖 14 个教材章节，
聚合结果写入 `library/index.json`。第 18 题的圆锥配图已重绘为
`library/figures/q-20261010-018/fig-1.svg`。

### 示例 2：自定义题型（本项目真实运行）

第 17 题第 (2) 问要求从三个条件中选两个证明第三个，属于「结构不良题」，内置受控词表中没有该题型，
Agent 自动在 `library/taxonomy.override.yaml` 中登记：

```yaml
question_types:
  add:
    - name: 结构不良题
      parent: 解答题
      source: custom
      note: 题目给出若干条件，要求选取部分条件并完成证明/求解
```

### 示例 3：筛选导出

**输入**：“导出高中人教 A 版选择性必修第二册第五章一元函数的导数及其应用里的所有解答题，
学生版，Word。”

**输出**：`library/exports/` 下生成对应 DOCX，题目连续编号、公式为 Word 原生公式、图形嵌入、
不含答案；教师版另含答案与解析。DOCX 由 Markdown 经 pandoc 转换，例如：

```bash
pandoc selected.md -o 导数专题-学生版.docx --mathml
```

## 目录结构

```text
First_Skill/
├── README.md
├── LICENSE                                  # MIT
├── skills/
│   └── Q-classification/
│       ├── SKILL.md                         # Skill 指令：完整工作流与分类规范
│       ├── LICENSE                          # MIT
│       ├── agents/
│       │   └── openai.yaml                  # OpenAI Assistant / Agents SDK 部署描述
│       └── references/
│           └── knowledge-framework.md       # 六套教材章节知识框架与分类目录
└── library/                                 # 运行后生成的题库（示例 1 的真实产物）
    ├── questions.jsonl                      # 全部题目记录，每行一题，增量追加
    ├── index.json                           # 教材/题型/知识点/难度聚合索引
    ├── taxonomy.override.yaml               # 自定义题型与框架微调
    ├── figures/
    │   └── q-20261010-018/
    │       └── fig-1.svg                    # 第 18 题圆锥配图重绘
    └── exports/                             # 筛选导出产物
```

## 示例数据声明

本项目 `library/` 目录中的示例题目（2026 年深圳市高三年级第二次调研考试数学试题）**仅用于功能演示与教学用途**，不附带任何商业使用授权；试题及相关内容的版权归原命题方（原考试组织与命题单位）所有。如需公开传播、出版或商用这些题目，请联系原命题方获取授权，或将示例数据替换为自有题目。

本项目的**代码部分**（Skill 指令、知识框架、配置与文档）仍按 MIT License 开源，MIT 许可不覆盖上述示例试题。

## 开源许可

本项目代码基于 [MIT License](LICENSE) 开源（Skill 目录内另附一份相同的 [LICENSE](skills/Q-classification/LICENSE)）。
