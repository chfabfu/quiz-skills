---
name: mermaid-diagram
description: Default channel for any diagram request. Answers with one fenced mermaid code block in chat, for requests written in Chinese or English, without the user having to name Mermaid. Covers flowchart, sequence, architecture, state, swimlane, Gantt, ER, class and pie. Render target is a phone chat client about 375px wide, where the host CSS, the theme and the renderer version are unknown and the result is not visible while generating, so the drawing must stay narrow and CJK labels must be broken by hand. Trigger on 流程图, 时序图, 架构图, 状态图, 泳道图, 甘特图, ER 图, 类图, 结构图, 画个图, 图能不能渲染, 显示不全, 被裁切, 丢字, 偏左, and on flowchart, sequence diagram, state diagram, class diagram, Gantt chart, ER diagram, swimlane, architecture diagram, draw me a diagram, diagram not rendering, labels clipped or cut off, does not fit on my phone. Not for 出题, 选择题, quiz, multiple choice, practice test, since Mermaid has no state and cannot grade answers.
compatibility: No tools or workspace needed, the output is one fenced code block that the chat renders.
---

# Mermaid Diagram

## 0. Standing declaration

- The user does not name the tool. A request to draw a diagram, phrased in Chinese or in English,
  is answered here in the chat with one fenced `mermaid` block.
- Render target, a phone chat client about 375px wide. The host CSS, the light or dark theme and
  the Mermaid version are not yours to choose and are not known, and the result cannot be seen
  before it is sent.
- Everything below serves one goal, text complete and legible at that width, which in practice
  means keeping the drawing narrow.

## 1. Output rules

Emit only the fenced `mermaid` block plus at most one caption line. The picture is the
explanation, do not walk through the nodes in prose.

Default `flowchart TD`, use `LR` only for 4 nodes or fewer because a left to right drawing is wide
by construction on a portrait screen, and `graph TD` only for a legacy renderer. Add `classDef`,
`style` or colour only when colour was asked for.

- **Keep the drawing narrow, at most 2 columns side by side at any depth.** Width, not node count,
  is what ruins a phone. The host scales the whole SVG down to fit and the text shrinks with it,
  so a 20 node single chain stays readable while 5 parallel branches at 6 nodes do not. When the
  flow fans out wider than that, or runs longer than a screen, split into a main flow and sub
  flows as separate fenced blocks, joined by stub nodes such as `(["转 LED 流程"])`.
- **Never draw an edge between two different subgraphs.** A cross subgraph edge makes the layout
  reserve a long return channel down one side, the drawing gets wider, and every label scales down
  with it. Group only where nothing crosses, otherwise split the diagram.

### CJK labels must be broken by hand

Mermaid wraps text only at spaces. A Chinese or Japanese sentence contains none, so it becomes one
unbreakable token, the node stays a single long line, and everything past the box edge is clipped.
Quoting the label does not help, changing the shape does not help, only `<br/>` helps. `\n` does
nothing at all.

- Break long labels with `<br/>`, one clause per line.
- Line budget measured in display width, not in characters, **8 CJK glyphs or 16 half width
  characters per line**. In mixed text count each CJK glyph as two half width ones.
- Never put `<` `>` `&` or `~` inside a label. `htmlLabels` parses them as markup, so a bit
  expression such as `~(1<<led_index)` silently swallows the rest of the line. Write it in words,
  对应位输出低.

Counter example, it loses its tail on a 375px viewport, do not emit this:

```
A["ODR 将 led_index 对应位输出低，点亮该 LED"]
```

Correct, two short lines, fully visible on the same viewport:

```
A["ODR 对应位<br/>输出低，点亮 LED"]
```

## 2. Does it render here

When the user is testing the host, send the minimal probe first, one stadium node, one decision,
two labelled branches, one end node. Only after it renders, send the real diagram. A failure then
points at syntax rather than at size, and a host with no renderer is identified in one turn.

If the probe shows as plain text, stop. Say the host has no Mermaid renderer, a bigger or simpler
diagram will not change that, and offer an indented list or a table as the fallback.

## 3. Failure causes, most frequent first

1. **A long CJK label with no spaces.** The symptom is decisive, the tail of the sentence is
   missing while every box, diamond and branch label still draws correctly. Fix by inserting
   `<br/>` so each line is within the budget, not by quoting and not by resizing.
2. Too wide for the phone, from parallel branches, an `LR` layout, or a cross subgraph edge, which
   shrinks the whole drawing and amplifies cause 1.
3. `subgraph` with no matching `end`, or a node id literally named `end`.
4. A label holding `{ } [ ] ( ) / #` without quotes. Quote it, `A["审批 (二级)"]`.
5. Three backticks inside a label while the diagram sits in a fenced block, which closes the fence
   early and leaks the rest as prose. Describe such a token in words instead.
6. Invalid edge label `A ->|x|-> B`. Only `-->|x|` and `-- x -->` parse.
7. A label holding `<` `>` `&` or `~`, which htmlLabels consumes as markup.
8. Escape heavy shapes `[/x\]` and `[\x/]`, the chat Markdown eats the backslash. Use `[x]`.
9. `\n` used as a line break, it is not one, `<br/>` is, and it needs `htmlLabels`, on by default.
10. A stray `graph` header mixed into a `flowchart` file, or a `%%` comment on the first line.

## 4. Syntax crib

Shapes `([stadium])` `[(database)]` `[[subroutine]]` `((circle))` `{{hexagon}}`
`[/parallelogram/]` `[rect]` `{decision}`. Edges `-->` `---` `==>` `-.->` `--x` `--o`, labels
`-->|text|` or `-- text -->`. Grouping `subgraph id[Title]` then `end`, with `direction LR` inside
if needed, and no edge leaving it. Styling `classDef n fill:#e9f8ef,stroke:#16a34a` then
`class A,B n`.

## 5. Other diagram types

- sequence, `sequenceDiagram` with `A->>B: text`, `alt` `else` `end`, `Note over A,B: text`
- state, `stateDiagram-v2` with `[*] --> Idle`, `Idle: 待处理`, `Idle --> Done: 事件`
- gantt, `gantt` then `dateFormat YYYY-MM-DD`, `section 名称`, `任务名 :id, start, duration`
- ER, `erDiagram` with `CUSTOMER ||--o{ ORDER : places` plus attribute blocks
- class, `classDiagram` with `Animal <|-- Dog` and `+method()` visibility marks
- swimlane is a flowchart whose lanes are `subgraph` blocks, one `direction` each, still with no
  crossing edges

The same traps apply everywhere, message text after a colon must not contain a bare `end`, and
every opened block needs closing.

## 6. Hard limits, say them rather than fake around them

- Mermaid has no state, no variables and no scoring. A quiz in Mermaid is a decision tree the
  human walks alone, it cannot grade. Route that request to a quiz skill and say why in one line.
- `click ID "https://..." "tooltip"` produces a link and needs `securityLevel: loose`, the callback
  form needs a JS host, and GitHub plus most chat renderers strip both. Never promise a clickable
  diagram without checking the host.

## 7. Non fixes, do not offer these

- `%%{init: {"flowchart": {"wrappingWidth": 150}}}%%` does force wrapping, but on some hosts the
  leading `%%` directive is swallowed as a plain comment and the whole diagram then fails to
  render. Last resort on a host you confirmed accepts it, never the default answer.
- `padding` only adds whitespace around the drawing. It does not change the SVG size, so it fixes
  neither clipped text nor a picture stuck to the left edge.
- `defaultRenderer: elk` balances and centres better, but if the host has not loaded the ELK
  renderer the diagram comes out completely blank. Never enable it by default.
- A picture that sits flush left or is not centred is usually the host missing
  `svg { margin-inline: auto }`. State that plainly and stop, it cannot be fixed from inside the
  diagram code, and re issuing the diagram to chase it is wasted work.

## 8. Before delivering

- [ ] Each node's longest label line is within 8 CJK or 16 half width characters, insert `<br/>`
      wherever it is not.
- [ ] At most 2 columns side by side, no edge crossing subgraphs, no label containing `<` `>` `&`
      or `~`.
- [ ] Every branch reaches a terminal node, no orphan, no unreachable subgraph, ids unique and none
      named `end`.
- [ ] Nothing inside the block can truncate the fence, no triple backticks.
- [ ] Acceptance is text completeness at a phone width of about 375px, no lost tail. If you have
      not seen it rendered, say the layout is unverified instead of claiming it fits.
- [ ] The right channel, a diagram asked means a diagram given, not an HTML page.
