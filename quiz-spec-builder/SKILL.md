---
name: quiz-spec-builder
description: Implements an interactive HTML quiz from a written behaviour contract, with no template file to load, so the page is written from scratch each time and no engine is ever read into context. Supports single choice, multiple choice, fill in the blank and free text questions, with click grading, per question redo, question strip navigation, a result screen with redo wrong ones only, progress kept in localStorage under a fingerprinted key, export and import of progress, and a visible warning when storage is blocked. Use for the pure text version, the spec version, the token saving quiz builder, 纯文字版出题, 规范版出题, 不读模板出题, or 省 token 出题. Diagram requests belong to the mermaid-diagram skill, and the template backed twin is quiz-builder.
compatibility: Needs no files, tools or workspace. Output quality depends on the model, intended for strong models, output is not byte reproducible between runs.
---

# Quiz Spec Builder

No template file exists here, you write the whole page from this contract. That saves reading a
4.5k token template, and costs it: the engine is regenerated every time, so two quizzes may behave
slightly differently, and a defect report cannot be compared against a known file. Prefer the
sibling skill `quiz-builder` when the output must be reproducible or the model is weak.

If the user wants a diagram rather than questions, this skill does not apply, use mermaid-diagram.

## 1. Deliverable

One self-contained HTML document in a single `html` fenced block, which the user opens from the
chat's preview control. No external scripts, no CDN, no build step, no network. Keep the reply text
to two short lines, how to open it and that progress saves itself. Do not narrate the code and do
not print the document twice.

## 2. Question data

Put the questions in one array near the top, `const Q=[...]`, one object per question, fields on
their own lines so a human can edit them.

| Type | Fields | Grading |
| --- | --- | --- |
| single | `opts`, `answer` index from 0 | grade on click |
| multi | `opts`, `answer` array like `[0,2]` | toggle then submit, sets must be equal, missing one is wrong |
| blank | `answers` array of accepted strings, or one `answer` string | normalised equality, any accepted string wins |
| essay | `ref` required, `points` optional array | never auto graded, show `ref`, keyword coverage, three way self rating |

Infer the type when `t` is absent, `ref` means essay, else `answers` means blank, else `answer` is
an array means multi, else single. Default 5 questions, at most one multi and one essay.

`explain` must say why the wrong picks are wrong. Stems carry the whole task, options stay similar
in length, and the correct index rotates.

## 3. State model, this is the part people get wrong

Keep answers in one object keyed by question index, and render only from it:

```
S[i] = { v: submitted value, ok: true|false, self: 'ok'|'vague'|'no', redo: count }
```

Two separate facts, never conflate them:

- `v !== undefined` means the user **submitted** something
- `self` means the user **finished the self rating**

The reference answer of an essay must be shown from `v !== undefined`. Testing `self` for it is a
real historical bug, right after submitting an essay `self` is still empty, so the answer never
appears and the user reports "无法查看答案". Same for the redo button and the answered counter.

## 4. Required behaviour

- progress bar, answered count, correct count, always visible
- question strip with one button per question, coloured by state, tap to jump anywhere
- per question redo that keeps a redo counter and clears only that question
- result screen, per question your own answer next to the right one, an expandable 答案 control
  showing the correct or reference answer plus `explain`, a redo-wrong-ones-only action, redo-all
- blank normalisation on both sides, trim, lower case, collapse space runs, full width to half
  width, so a trailing space or a full width symbol still counts correct
- progress in `localStorage` under a key built from the title and a hash of the question array, so
  a new question set never reads the old answers
- an export and import panel for the state as text, the only way to carry progress elsewhere
- a run time data check that names the broken question index and renders a notice card for it,
  because there is no build step left to catch bad data
- if storage read or write throws, show a visible warning that progress will not survive and keep
  the page fully working. Never lose progress silently

## 5. Pitfalls that cost real debugging time

1. A closing script tag literal anywhere inside the question data truncates the page.
2. Do not write the marker or placeholder text into comments inside the file, it gets matched a
   second time and injects the data twice.
3. Replacing the data must replace only the data. Swallowing the `const Q=[` line or the closing
   `];` yields a blank page that still passes a bracket balance check.
4. Multi choice selections made before submitting must live in a draft variable, otherwise every
   re-render wipes them and nothing can be selected.
5. Only `<` needs escaping as `&lt;`. A blank answer must be stored raw, `-.->` not `-.&gt;`,
   because the stored string is compared against typed input and an entity never matches.
6. A failed progress import must not turn persistence off as a side effect.
7. A question missing `opts` must render a notice card. Calling `o.opts.map` on it throws and takes
   the whole page down, which defeats the point of validating.
8. The preview pane can be narrower than 640px, and `prefers-color-scheme` may be ignored, so the
   default theme must be readable on its own.
9. Keyboard shortcuts need focus inside an iframe, so on screen buttons must be sufficient alone.

## 6. Budget

Plain CSS in a `style` block, no framework, no icons, no animation beyond a colour change. Keep the
whole document under about 200 lines. Do not add a timer, sound, charts or sharing unless asked,
every unrequested feature is extra output the user pays for.

## 7. Before delivering

- [ ] Every item in section 4 is present, checked against the code you just wrote, not assumed.
- [ ] Essay reference answer is gated on `v !== undefined`, not on `self`.
- [ ] Each `answer` index is inside its own `opts`, multi answers are arrays of valid indices.
- [ ] No raw `<` in strings except intended tags, no closing script tag literal in the data.
- [ ] Blank answers stored raw, comparison normalises both sides.
- [ ] Storage failures degrade to a warning, never to silent loss or a blank page.
- [ ] The document is one fenced block, printed once, with at most two lines of prose.

## 8. If the user reports a defect

Ask which question and what they tapped, then fix the specific gate or state, not the whole page.
Recurring symptom and cause pairs, nothing happens on submit means the state key or the answered
flag, answer never shows means `self` gating, blank always wrong means normalisation or an escaped
entity, progress gone after reload means the key lacks a question fingerprint, multi choices
disappearing means the draft variable is missing.
