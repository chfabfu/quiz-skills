---
name: mermaid-diagram
description: Draws Mermaid diagrams inline in the chat and explains why one failed to render. Covers flowchart, sequence, architecture, state, swimlane, Gantt, ER, class and pie diagrams, plus a minimal probe test for whether the host renders Mermaid at all. Use for 流程图, 时序图, 架构图, 状态图, 泳道图, 甘特图, ER 图, 类图, 结构图, 画个图, 图能不能渲染, or when a Mermaid block shows up as plain text instead of a picture. Not for 出题, 选择题 or quizzes, Mermaid has no state and cannot grade answers, those belong to a quiz skill.
compatibility: No tools or workspace needed, the output is one fenced code block that the chat renders.
---

# Mermaid Diagram

## 1. Output rules

Emit only the fenced `mermaid` block plus at most one caption line. No prose walk through of the
nodes, the picture is the explanation.

Default `flowchart TD`. Use `graph TD` only when the host is a legacy renderer. Keep the first
version under about 12 nodes. Add `classDef`, `style` or colours only when colour was asked for.

## 2. Does it render here

When the user is testing the host, send the minimal probe first, one stadium node, one decision,
two labelled branches, one end node. Only after it renders, send the real diagram. This way a
failure is attributable to syntax rather than to size, and a host with no renderer is identified
in one turn.

If the probe shows as plain text, stop. Say the host has no Mermaid renderer, a bigger or simpler
diagram will not change that, and offer an indented list or a table as the fallback.

## 3. Failure causes, most frequent first

1. `subgraph` with no matching `end`, or a node id literally named `end`.
2. A label holding `{ } [ ] ( ) / #` without quotes. Quote it, `A["审批 (二级)"]`.
3. Three backticks inside a label while the diagram sits in a fenced block, which closes the fence
   early and leaks the rest as prose. Describe such a token in words instead.
4. Invalid edge label `A ->|x|-> B`. Only `-->|x|` and `-- x -->` parse.
5. Escape-heavy shapes `[/x\]` and `[\x/]`, the chat Markdown eats the backslash. Use `[x]` and put
   the shape note in the label text.
6. `<br/>` inside a label needs `htmlLabels`, on by default. `\n` is not a line break.
7. A stray `graph` header mixed into a `flowchart` file, or a `%%` comment on the first line.

## 4. Syntax crib

Shapes `([stadium])` `[(database)]` `[[subroutine]]` `((circle))` `{{hexagon}}` `[/parallelogram/]`
`[rect]` `{decision}`. Edges `-->` `---` `==>` `-.->` `--x` `--o`, labels `-->|text|` or
`-- text -->`. Grouping `subgraph id[Title]` then `end`, with `direction LR` inside if needed.
Styling `classDef n fill:#e9f8ef,stroke:#16a34a` then `class A,B n`.

## 5. Other diagram types

- sequence, `sequenceDiagram` with `A->>B: text`, `alt` `else` `end` blocks, `Note over A,B: text`
- state, `stateDiagram-v2` with `[*] --> Idle`, `Idle: 待处理`, `Idle --> Done: 事件`
- gantt, `gantt` then `dateFormat YYYY-MM-DD`, `section 名称`, `任务名 :id, start, duration`
- ER, `erDiagram` with `CUSTOMER ||--o{ ORDER : places` and attribute blocks
- class, `classDiagram` with `Animal <|-- Dog` and `+method()` visibility marks
- swimlane is a flowchart whose lanes are `subgraph` blocks, one `direction` per lane

For these the same traps apply, colon text must not contain a raw `end`, and every block opened
needs closing.

## 6. Hard limits, say them rather than fake around them

- Mermaid has no state, no variables and no scoring. A "quiz in Mermaid" is a decision tree the
  human walks alone, it cannot grade. Route that request to a quiz skill and say why in one line.
- `click ID "https://..." "tooltip"` produces a link and needs `securityLevel: loose`, the callback
  form needs a JS host, and GitHub plus most chat renderers strip both. Never promise a clickable
  diagram without checking the host.

## 7. Before delivering

- [ ] Every branch reaches a terminal node, no orphan nodes, no unreachable subgraph.
- [ ] Node ids unique, none named `end`, labels with special characters quoted.
- [ ] Nothing inside the block can truncate the fence, no triple backticks.
- [ ] The right channel, a diagram asked means a diagram given, not an HTML page.
- [ ] Claims match what was seen, if it was never rendered say it is unverified.
