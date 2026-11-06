# quiz-skills

[中文文档](README.zh-CN.md)

Agent Skills that turn any topic into a **playable quiz**, a **Mermaid diagram**, or an
**HTML flowchart**, delivered inline in the chat. The quiz is a single self-contained HTML file:
no workspace, no build step, no network, and your answers survive closing the page.

![Preview](docs/preview.png)
<!-- replace this path with your own screenshot -->

## What's in here

| Skill | Does | Form |
| --- | --- | --- |
| [`quiz-builder`](quiz-builder) | Questions → interactive HTML quiz | Ships the real template, output is **byte reproducible** and safe on weaker models |
| [`quiz-spec-builder`](quiz-spec-builder) | Questions → interactive HTML quiz | **Spec only**, no template file, the model writes the page from a behaviour contract. Cheaper on tokens, less reproducible |
| [`mermaid-diagram`](mermaid-diagram) | Flowchart, sequence, architecture, state, swimlane, Gantt, ER, class | Single-file rules, emits a `mermaid` block inline |
| [`html-flowchart`](html-flowchart) | Flowcharts as HTML, laid out for a phone screen | ~10 line rule file, no template, no JS. Emits one self-contained `html` block |

### Question types

**Single choice** graded on click · **multiple choice** with a submit step, a missing pick is wrong ·
**fill in the blank** graded after normalisation (trim, case fold, space collapse, full width to
half width) · **free text** never auto graded, but shown against a reference answer with keyword
coverage and a three way self rating.

### What the quiz page already does

- question strip coloured by state, tap any number to jump
- **per question redo**, with a redo counter, and a "redo wrong ones only" action
- result screen listing your own answer next to the correct one, each expandable to show the
  answer and its explanation
- progress saved to `localStorage` under a key fingerprinted by title and question set, so a new
  quiz never reads the old answers
- export and import of progress as text, the only way to carry it to another browser
- a visible warning when storage is blocked, instead of silently losing progress
- a run time data check that names the broken question index and renders a notice card, so bad
  data degrades instead of blanking the page

## Install

A skill is just a folder, and the `name` field in `SKILL.md` must equal the folder name.

```bash
git clone https://github.com/chfabfu/quiz-skills.git

# Claude Code, personal
cp -r quiz-skills/quiz-builder      ~/.claude/skills/
cp -r quiz-skills/quiz-spec-builder ~/.claude/skills/
cp -r quiz-skills/mermaid-diagram   ~/.claude/skills/
cp -r quiz-skills/html-flowchart    ~/.claude/skills/

# or project level: put them in .claude/skills/ inside your repo
```

Other clients that read Agent Skills (for example Rikkahub) take the same folders in their own
skills directory, typically `/skills/`.

**⚠️ Enable it after installing.** New skills ship disabled. Open the skills panel in your client
and switch it on, or nothing will ever trigger. This is the single most common reason a skill
"doesn't work".

For claude.ai import, the zip must contain **the skill folder as its first level**, not a bare
`SKILL.md`:

```bash
cd quiz-skills && zip -r ../quiz-builder.zip quiz-builder
unzip -l ../quiz-builder.zip   # expect quiz-builder/SKILL.md
```

**Do not drag folders into the web uploader.** GitHub's browser upload flattens nested folders and
can even turn each folder into its own branch. Use `git push`, or Add file → Create new file and
type the full path with slashes, for example `quiz-builder/SKILL.md`.

## Usage

Just ask, once the skill is enabled:

```
draw the refund flow as a flowchart
make 5 multiple choice questions about HTTP status codes, clickable and auto graded
use the spec version, 5 questions on STM32 registers
```

Quiz requests match whichever quiz skill you picked. Flowcharts are now claimed by
`html-flowchart`, every other diagram type stays with `mermaid-diagram`:

| Your situation | Use |
| --- | --- |
| Ordinary model, output must be consistent, you want to hand the same file to someone | `quiz-builder` |
| Strong model, you care about token cost, one shot per conversation | `quiz-spec-builder` |
| Both quiz skills enabled | say "spec version" explicitly to hit `quiz-spec-builder`, otherwise `quiz-builder` wins |
| You read diagrams on a phone | `html-flowchart`, one `html` block you can open in the browser |
| Sequence / architecture / state / swimlane / Gantt / ER / class | `mermaid-diagram`, it is the only one with those rules |
| Both diagram skills enabled and you just say "flowchart" | whichever `description` matches first wins. Enable one of the two if you want it deterministic |

### Measured cost

Tokens per quiz turn, excluding your conversation history, estimated from character counts:

| Path | Loaded | Output | Total |
| --- | --- | --- | --- |
| `quiz-builder` | 1425 + 1254 + 4498 | ≈4750 | **≈11.9k** |
| `quiz-spec-builder` | 2395 | ≈5000-6500 | **≈7.4-8.9k** |
| `mermaid-diagram` | 1407 | diagram size | **≈1.4k+** |
| `html-flowchart` | 776 | one HTML block | **≈0.8k+** |

`quiz-builder` pays to read a 4.5k template and echo it back. `quiz-spec-builder` skips the read and
rewrites the engine every time, which is cheaper but model dependent, and two quizzes may
differ slightly in behaviour. `html-flowchart` deliberately carries no template at all: the rule is
the whole skill, so it is the cheapest thing in this repo to trigger.

Only `name` and `description` stay resident in context, about 640 tokens for the three original
skills plus ~0.25k for `html-flowchart`. The body loads only when a request matches.

## Question data format

The `t` field is optional, the type is inferred from which fields are present.

| Type | Fields |
| --- | --- |
| single | `opts` + `answer`, a 0 based index |
| multi | `opts` + `answer`, an array such as `[0,2]` |
| blank | `answers` an array of accepted strings, or one `answer` string |
| essay | `ref`, required, plus optional `points` keyword array |

Every type also takes `q` for the stem and `explain` for the reasoning. Strings are injected as
HTML, so **only `<` needs escaping** as `&lt;`. Keep blank answers raw, store `-.->` not `-.&gt;`,
because the stored value is compared against what the user types and an entity never matches.

## Known limits

- Mermaid has no state and cannot grade answers. Its `click` links are stripped by GitHub and most
  chat renderers. A "clickable auto graded Mermaid quiz" is not possible, the skills route that
  request to HTML and say why in one line.
- Free text questions are never auto graded, only self rated with keyword coverage.
- Multi answer objective questions cannot be scored by this page; converting an external bank
  flags them to rewrite or skip.
- The preview panel is a sandbox. `localStorage` may be blocked (the page warns and you fall back
  to export/import), it can be narrower than 640px, and `prefers-color-scheme` may be ignored.
- `html-flowchart` and `mermaid-diagram` both match "flowchart / 流程图". They are alternatives, not
  a pipeline. `html-flowchart` draws stacked boxes with `↓` between them, so a loop back across
  several rows is labelled with step numbers ("回到 ②") instead of routed as a real connector line.
  If you want routed arrows and swimlane dividers, you want a diagram engine, not this skill.

## Layout

```
quiz-skills/
├── quiz-builder/
│   ├── SKILL.md
│   ├── assets/quiz-template.html      the engine, with data markers
│   └── references/authoring.md        question authoring rules
├── quiz-spec-builder/
│   └── SKILL.md                       behaviour contract + pitfalls list
├── mermaid-diagram/
│   └── SKILL.md                       diagram rules and render failures
└── html-flowchart/
    └── SKILL.md                       use HTML for flowcharts, phone first
```

No README inside the skill folders, that is against the Agent Skills guidance, documentation
lives here.

## License

MIT — see [LICENSE](LICENSE). Copyright (c) 2026 chfabfu
