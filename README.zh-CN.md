# quiz-skills

[English README](README.md)

Agent Skills that turn any topic into a **playable quiz**, a **Mermaid diagram**, or an
**HTML flowchart**, delivered inline in the chat. No workspace, no build step, no network required
for the quiz page.

把任意主题变成「聊天里点开就能作答的 HTML 小测」、一张 Mermaid 图，或**一张手机上看舒服的 HTML
流程图**。答题进度自动保存，关掉页面下次接着做。

![预览效果](docs/preview.png)
<!-- 把上面路径换成你的截图，建议就用聊天里点开预览那张 -->

## 包含的五个技能

| 技能 | 干什么 | 形态 |
| --- | --- | --- |
| [`quiz-builder`](quiz-builder) | 出题 → 交互式 HTML 小测 | 附带模板文件，输出**逐字节可复现**，弱模型也稳 |
| [`quiz-spec-builder`](quiz-spec-builder) | 出题 → 交互式 HTML 小测 | **纯规范**，不含模板文件，模型现场写页面，省 token 但每次实现略有差异 |
| [`mermaid-diagram`](mermaid-diagram) | 流程图 / 时序图 / 架构图 / 状态图 / 泳道图 / 甘特 / ER / 类图 | **完整规则书**，八节上限 + 渲染失败排查表 + 交付前自检清单，直接输出 ```mermaid 代码块 |
| [`mermaid`](mermaid) | 同样的图型、同样的通道 | **只有声明**，正文约 50 行。说清通道和渲染场地，只留三条与排版有关的事实，为强模型准备 |
| [`html-flowchart`](html-flowchart) | 流程图用 HTML 画，按手机屏幕排版 | 正文十行左右的规则文件，不带模板、不用 JS，输出一个自包含 `html` 代码块 |

小测支持的题型：**单选**（点选即判分）、**多选**（勾选后提交，漏选算错）、**填空**（规范化比对，
全角字符和多余空格不影响判对）、**问答**（不自动判分，给参考答案 + 关键词命中 + 三档自评）。

已实现的行为：题号导航条（按对错着色，可跳题）、**单题重做**并保留重做次数、结果页逐题回顾与
「只重做错题」、`localStorage` 进度保存（键名含标题与题库指纹，换一套题不会串档）、进度文本
导出/导入（换浏览器时用）、存储被禁用时**显示警告而不是静默丢进度**、题库数据出错时点名第几题
有问题并渲染提示卡而不是整页崩溃。

## 安装

技能就是一个文件夹，`SKILL.md` 里的 `name` 必须与目录名一致。

```bash
git clone https://github.com/chfabfu/quiz-skills.git

# Claude Code，个人级
cp -r quiz-skills/quiz-builder      ~/.claude/skills/
cp -r quiz-skills/quiz-spec-builder ~/.claude/skills/
cp -r quiz-skills/mermaid-diagram   ~/.claude/skills/
cp -r quiz-skills/mermaid           ~/.claude/skills/
cp -r quiz-skills/html-flowchart    ~/.claude/skills/

# 项目级则放到仓库内的 .claude/skills/
```

其他支持 Agent Skills 的客户端（如 Rikkahub）把同样的目录放进它自己的技能目录，通常是 `/skills/`。

**⚠️ 装完记得启用。** 新技能默认是关闭状态，需要在客户端的技能面板里手动打开，否则永远不会触发。
这是最常见的"装了没用"原因。

给 claude.ai 用 zip 导入时，**压缩包里第一层必须是技能目录本身**，不是裸的 `SKILL.md`：

```bash
cd quiz-skills && zip -r ../quiz-builder.zip quiz-builder
unzip -l ../quiz-builder.zip   # 应看到 quiz-builder/SKILL.md
```

**别用网页拖拽上传文件夹。** GitHub 的浏览器上传会把嵌套目录平铺，甚至把每个文件夹变成一个同名分支。
用 `git push`，或者 Add file → Create new file 并在文件名里输入带斜杠的完整路径，比如 `quiz-builder/SKILL.md`。

## 怎么用

装好并启用后，直接说人话：

```
画一下订单退款流程图
出 5 道 HTTP 状态码的选择题，要能点选判分
用纯文字版出题，来 5 道 STM32 寄存器
```

出题请求命中哪个看你怎么选；画图请求有三个技能在等，它们是**三选一**而不是接力：

| 你的情况 | 建议 |
| --- | --- |
| 模型一般、要每次输出一致、要能拿同一份文件分享别人 | `quiz-builder` |
| 模型很强、在意 token 成本、单轮用完就丢 | `quiz-spec-builder` |
| 两个出题技能都常开 | 想走纯文字版就点名"用纯文字版出题"，否则默认命中 `quiz-builder` |
| 图主要在手机上看不想放大，且想要自己能控制的产物 | `html-flowchart`，一段 `html` 存成文件用浏览器打开 |
| 要 Mermaid、模型够强、在意成本 | `mermaid`，只有声明，加载约 1.1k token |
| 要 Mermaid 被规则管住，或者图已经裁切了要查原因 | `mermaid-diagram`，上限、排查表和其他图型都在它身上 |
| 几个画图技能都开着，又只说"画个流程图" | 谁的 `description` 先命中算谁。想要确定性就只开一个 |

### 成本实测

一轮出卷的 token 开销（按字符数估算，不含你的对话历史）：

| 路径 | 加载 | 输出 | 合计 |
| --- | --- | --- | --- |
| `quiz-builder` | 1425 + 1254 + 4498 | ≈4750 | **≈11.9k** |
| `quiz-spec-builder` | 2395 | ≈5000-6500 | **≈7.4-8.9k** |
| `mermaid-diagram` | ≈2250 | 视图大小 | **≈2.3k+** |
| `mermaid` | ≈1140 | 视图大小 | **≈1.1k+** |
| `html-flowchart` | 776 | 一段 HTML | **≈0.8k+** |

`quiz-builder` 贵在要把 4.5k 的模板读进来再吐出去；`quiz-spec-builder` 免掉了读取，代价是每次
现场重写引擎，质量取决于模型，且两份卷子行为可能不完全一致。`html-flowchart` 故意不带任何模板，
规则本身就是全部内容，是这个仓库里触发最便宜的一个。

两个 Mermaid 技能按「英文约 4 字符/token、中文约 1.1 token/字」折算，所以 `mermaid` 的实际开销
比它那 3.4KB 看着要多一些，但仍只有 `mermaid-diagram` 的一半左右。它保留了**事实**——Mermaid 只在
空格处断行所以中文标签必须手工 `<br/>`、毁掉手机视图的是宽度而不是节点数、标签里放 `<` `>` `&` `~`
会被吃掉，以及那四个非修复项；砍掉的是**上限、探针、语法速查和排查表**，节奏交给模型自己拿。

常驻上下文的只有 `name` 和 `description`，原来三个技能合计约 640 token，`html-flowchart` 再加约
0.25k，`mermaid` 再加约 0.5k（它的 description 是中文，中文一个字约一个 token），
`mermaid-diagram` 的双语触发词再加约 0.1k。正文只在被触发时加载。

## 题目数据格式

四个字段族，`t` 可省略（按字段自动推断类型）：

| 题型 | 字段 |
| --- | --- |
| 单选 | `opts` + `answer`（从 0 开始的下标） |
| 多选 | `opts` + `answer`（数组，如 `[0,2]`） |
| 填空 | `answers`（可接受答案数组，任一命中即对）或单个 `answer` |
| 问答 | `ref`（参考答案，必填）+ `points`（关键词数组，可选） |

所有题型都可加 `q` 题干、`explain` 解析。字符串按 HTML 注入，**只有 `<` 必须写成 `&lt;`**；
填空答案要保留原文（存 `-.->` 而不是 `-.&gt;`），否则永远比对不上。

## 已知边界

- Mermaid **没有状态、不能判分**，`click` 生成的链接会被 GitHub 等渲染端剥掉。所以"可判分的
  Mermaid 小测"这种需求做不到，技能会改用 HTML 并说明原因。
- 问答题**不自动判分**，只能自评 + 关键词覆盖提示。
- 多选题库（一题多个正确答案的客观题）无法判分，转换外部题库时会提示跳过或改写。
- 预览面板是受限沙箱：`localStorage` 可能被禁用（此时页面会显示警告，用导出/导入文本继续），
  宽度可能小于 640px，`prefers-color-scheme` 可能被忽略。
- `mermaid` 和 `mermaid-diagram` 是**故意重叠**的：一个声明通道，一个强制执行，只开一个。
  模型一般、或者标签正在丢尾要排查时选 `mermaid-diagram`；模型够强、想省一半加载时选 `mermaid`。
  两个都不等于渲染预览，图的版式在你看到之前永远未被验证过。
- `html-flowchart`、`mermaid`、`mermaid-diagram` 都会命中"流程图"。三者是**三选一**，不是接力。
  `html-flowchart` 是竖排的盒子加 `↓`，跨好几行的回退线不画，改用步骤编号标注（"回到 ②"）；
  要真正的连线路由和泳道分隔线，该用图引擎，而不是这个技能。

## 目录结构

```
quiz-skills/
├── quiz-builder/
│   ├── SKILL.md
│   ├── assets/quiz-template.html      小测引擎，含数据标记
│   └── references/authoring.md        出题规范与数据格式
├── quiz-spec-builder/
│   └── SKILL.md                       行为契约 + 踩过的坑清单
├── mermaid-diagram/
│   └── SKILL.md                       画图规则与渲染失败排查
├── mermaid/
│   └── SKILL.md                       只做声明，画图请求默认走 Mermaid
└── html-flowchart/
    └── SKILL.md                       流程图改用 HTML，按手机排版
```

技能目录内不放 README（Agent Skills 规范里那类文件不帮助模型完成任务），文档就写在这里。

## License

MIT — 见 [LICENSE](LICENSE)。Copyright (c) 2026 chfabfu
