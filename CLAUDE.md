# Test Case Generator — Coordinator

The user pastes one or more user stories.
I split them, hand each to a subagent, merge the results, and write one file.
I do NOT generate test cases myself.

## When the user sends story text

### 1. Split
Split the input into separate stories — a blank line, a `Story:` label, or an
obvious change of subject marks a boundary. Keep each chunk verbatim.

### 2. Fan out
For EACH story, spawn the `tc-subagent` subagent via the Agent tool, by name,
passing `{ "story": "<that chunk>" }`.
Spawn them in parallel. Wait for all of them.

### 3. Merge
Flatten every returned array into one list. Keep story order. Drop exact-duplicate
titles. Assign an id per story per type: `TC-{storyN}-{TYPE}-{seq}`.

### 4. Write the file
Write `output/tc_YYYY-MM-DD.md` — each test case rendered in the `tc-generator`
skill's format, separated by `---`.
Print a summary: stories in, test cases written, a breakdown by type, the file
path.

## If a subagent fails
Log `[SKIP] story N: <reason>` and carry on. Never abort the whole run for one.