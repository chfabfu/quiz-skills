---
name: quiz-builder
description: Builds an interactive HTML quiz from the template shipped inside this skill, so the output is byte reproducible and safe on weaker models. Supports single choice, multiple choice, fill in the blank and free text questions, with click grading, per question redo, question strip navigation, a result screen with redo wrong ones only, progress kept in localStorage under a fingerprinted key, export and import of progress, and a visible warning when storage is blocked. Use for 出题, 选择题, 填空题, 问答题, 测验, 自测, 随堂测, 刷题, 练习卷, quiz, multiple choice, practice test, exam questions, or clickable and auto-graded questions. Diagram requests such as 流程图 or 时序图 belong to the mermaid-diagram skill, not to this one.
compatibility: Reading this skill needs no workspace. The delivered page needs no network, no build step and no tools, it opens in any browser.
---

# Quiz Builder

The page comes from a shipped template. Read it, replace three places, output it. Never
re-implement the engine, that is what the sibling skill `quiz-spec-builder` is for when the user
wants the no-template version.

If the user wants a diagram rather than questions, this skill does not apply, use mermaid-diagram.

## 1. Workflow

1. Author the questions. Read [authoring rules](references/authoring.md) once per conversation,
   not once per quiz.
2. Reuse before reading. If the template or a reference file is already in this conversation, do
   not read it again, edit the copy already in context.
3. Read [the template](assets/quiz-template.html) once per conversation. Replace exactly three
   places, the two `/*QUIZ_TITLE*/` markers, and the single `/*QUIZ_DATA*/` line that sits between
   `const Q=[` and `];`, which becomes the comma separated question objects. Keep the `const Q=[`
   and `];` lines. Keep everything else byte for byte.
4. Output the whole document in one `html` fenced block, then at most two short lines, how to open
   it and that progress saves itself. Do not narrate the code and do not print it twice.

## 2. What the template already does

Never reimplement these and never list them back at the user:

- single choice graded on click, multi with a submit step, blank graded after normalisation, essay
  never auto graded but shown against `ref` with keyword coverage and a three way self rating
- question strip coloured by state, jump anywhere, per question redo keeping a redo counter,
  result screen with your own answer next to the right one, an expandable 答案 per question,
  redo wrong ones only, redo all
- progress in `localStorage` under a key fingerprinted by title and question hash, plus an export
  and import panel, plus a visible warning when storage is blocked instead of silent loss
- a run time data check that names the broken question index and renders a notice card rather than
  crashing, since there is no build step left to catch bad data

Two states the template deliberately keeps apart, and the reason it matters: `v !== undefined`
means the user submitted, `self` means they finished rating themselves. The essay reference answer
is gated on `v`. Gating it on `self` was a real bug here and produced "无法查看答案".

## 3. Rebuild cheaply

Only the data block changes. The full document still has to be re-emitted for the preview to work,
so keep questions terse and the default count at 5. If the user is happy editing a file themselves,
offer to hand over just the question objects plus one line saying where to paste them.

## 4. Before delivering

- [ ] The request was for questions, not a diagram.
- [ ] Three markers replaced and nothing else changed, `const Q=[` and `];` both still present.
- [ ] Each `answer` index is inside its own `opts`, multi answers are arrays of valid indices.
- [ ] Each `explain` says why the wrong picks are wrong.
- [ ] Only `<` escaped as `&lt;`, blank answers stored raw, no closing script tag literal in data.
- [ ] One fenced block, printed once, at most two lines of prose, nothing claimed about rendering.

## 5. Offer, do not add

Per question timer, random sampling from a larger bank, print or PDF sheet, a Mermaid figure per
question through the official CDN, and a two pane fix the broken diagram mode. Each is a small
patch to the template, built on request only.
