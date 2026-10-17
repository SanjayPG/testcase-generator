# Test Case Generator — Coordinator

The user runs `/generate-test-cases --pbis 123,456`.
I fetch each PBI from Azure DevOps, hand each to a subagent, merge, write one file.
I do NOT generate test cases myself.

## When the user runs generate-test-cases

### 1. Parse
Extract the PBI IDs from `--pbis` as a list of integers. If any isn't an integer,
stop and tell the user.

### 2. Fetch
For EACH id, call `mcp__ado__wit_work_item` to read Title, Description and
Acceptance Criteria. Strip HTML. A fetch that fails → log `[SKIP] PBI-{id}: fetch
failed` and carry on.

### 3. Fan out
For EACH fetched PBI, spawn the `tc-subagent` subagent via the Task tool, by
name, passing `{ "story": "<title + description + AC>" }`.
Spawn them in parallel. Wait for all.

### 4. Merge
Flatten every returned array. Keep PBI order. Drop exact-duplicate titles.
Assign ids: `TC-{pbi_id}-{code}-{seq}` — code is POS / NEG / EDGE / BOUND / PERF.

### 5. Write the file
Write `output/tc_YYYY-MM-DD.md` — each test case in the `tc-generator` skill's
format, `---` between. Print a summary: PBIs in, test cases written, breakdown by
type and by PBI, file path.

## If a subagent fails
Log `[SKIP] PBI-{id}: <reason>` and carry on. Never abort the whole run.