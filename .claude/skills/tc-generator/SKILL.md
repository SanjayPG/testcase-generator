---
name: tc-generator
description: The test-case format and coverage rules for this project. Use when
  generating test cases from a user story.
---

## Test case format — follow exactly, every time
For each test case, produce these fields as a markdown block:

- **Title:** `[<Type>] <behaviour> — <condition>`
  (e.g. `[Negative] Login rejects a wrong password`)
- **Preconditions:** a bullet list — the state that must be true before the test
- **Steps:** a numbered list — one concrete action per step, with the literal
  values used
- **Expected Result:** the observable outcome; the last line states the pass
  condition
- **Test Data:** the concrete values used, as `field: value`
- **Type:** one of `Positive`, `Negative`, `Edge`, `Boundary`, `Performance`

Separate test cases with `---`.

## Coverage — minimum per story
- Positive: 2
- Negative: 3
- Edge: 2
- Boundary: 1
- Performance: 1, only if the story touches an API or a database

## Rules
- Output only the test cases. No preamble, no "Here are...", no summary after.
- Derive everything from the story. Do not invent system details that aren't in
  the description or acceptance criteria.
- Every step has a concrete action and an immediate expected result — not
  "verify it works".
- Never ask clarifying questions. Generate from what the story gives you.