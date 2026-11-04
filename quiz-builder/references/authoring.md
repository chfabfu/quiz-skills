# Question authoring

## Data block shape

Paste comma separated objects at the data marker, one object per question, fields on their own
lines. No outer square brackets, no trailing comma.

```
{
  "q": "菱形判断节点的正确写法是？",
  "opts": ["<code>C{需要审批?}</code>", "<code>C(需要审批?)</code>"],
  "answer": 0,
  "explain": "花括号是菱形，圆括号是圆角起止节点。"
}
```

## Four types, `t` is optional

| Type | Fields | Grading |
| --- | --- | --- |
| `choice` | `opts`, `answer` a number | click to grade immediately |
| `multi` | `opts`, `answer` an array like `[0,2]` | toggle then submit, sets must be equal |
| `blank` | `answers` an array of accepted strings, or one `answer` string | normalised equality, any accepted string wins |
| `essay` | `ref` required, `points` optional array | never auto-graded, shows `ref`, keyword hits, self rate |

Inference when `t` is omitted: `ref` present means essay, else `answers` present means blank,
else `answer` is an array means multi, else choice. Write `t` only when the data would be
inferred wrong.

Blank normalisation on both sides: trim, lower case, collapse runs of spaces, full width
punctuation and letters folded to half width. So a user typing `－.->` or adding a trailing
space still counts correct. Do not rely on it for case-sensitive identifiers, the fold is
deliberate and lenient.

## Escaping, this one bites

The strings are injected as HTML, so **only `<` must be written `&lt;`**. Keep `>` raw.

- Display text may use `-->` after escaping the arrow head, either `--&gt;` or raw `-->` renders
  the same, pick one and stay consistent inside a question.
- A `blank` answer must stay raw: store `-.->` not `-.&gt;`. The stored string is compared against
  what the user types, and an entity never matches typed input.
- Markup is allowed and encouraged for readability: `<code>`, `<b>`, `<i>`.
- Never put a closing script tag literal in the data, it truncates the page.

## Item quality

1. One knowledge point per question. If an option could be "partly right", split it.
2. 3 or 4 options, similar length and grammar. Uneven length leaks the answer.
3. `explain` says why the wrong picks are wrong, never just restates the right one.
4. Rotate the correct index, no more than two in a row on the same letter.
5. Order easy to hard. Put at most one multi and one essay in a 5 question set, they are slow.
6. Never "all of the above", double negatives, or "most correct".
7. For `multi`, an incomplete set is wrong, so say 多选 in the stem to set expectations.
8. For `essay`, `points` should be 3 to 5 words that a correct answer must contain, not the
   whole sentence. It is a hint about coverage, not a score.

## Distractors

Built from mistakes people really make, never from nonsense. Four reliable sources:

- the wrong symbol's real meaning, `C(文本)` is a rounded node, not a random guess
- a form that parses but does something else than intended
- a plausible form that is invalid, `A ->|x|-> B`
- the correct form belonging to a different question, `-.->` against `==>`

Every distractor must be defensible in one sentence, otherwise delete it.

## Converting a foreign question bank

Recognise such data by shape, not by name: `题目 / 选项 / 答案`, or `question / choices / answer`,
where the answer may be one string or an array.

Input:

```
{ "题目": "设计心理学的作者是？", "选项": ["Jodie Moul", "Donald Norman", "Tim Brown"], "答案": ["Donald Norman"] }
```

Output, index found by locating the answer string inside the option array:

```
{
  "q": "《设计心理学》的作者是谁？",
  "opts": ["Jodie Moul", "Donald Norman", "Tim Brown"],
  "answer": 1,
  "explain": "Donald Norman 在本书提出示能与心智模型。"
}
```

If `答案` holds several strings, either set `t` to `multi` with all their indices, or rewrite
into a single answer. Never modify the source file.
