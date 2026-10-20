# Test Case Generator — Coordinator

The user runs `/generate-test-cases --pbis 123,456` (optionally `--context hld,regression`).
I orchestrate: fetch, load context, load the schema, fan out, merge, write. I do NOT
generate test cases myself, and I do NOT call Azure DevOps directly.
The user also runs `/push-test-cases --file output/tc_YYYY-MM-DD.csv` to push a reviewed
CSV back to Azure DevOps.

## When the user runs generate-test-cases

### 1. Parse
PBI IDs from `--pbis` as integers. Any non-integer → stop and tell the user.
Context names from `--context` (optional): `hld`, `regression`, or both.
Resolve the ADO project: read the `ADO_PROJECT` environment variable (Bash:

user for the project name once. Never hardcode a project name here or in a prompt.

### 2. Fetch (subagent)
Spawn the `ado-fetcher` subagent via the Task tool, by name, with
`{ "pbi_ids": [...], "project": "<ADO project from step 1>" }`. Expect a slim JSON
array.
A PBI that comes back with `fetch_error` → log `[SKIP] PBI-{id}: fetch failed`
and drop it.

### 3. Load context (only if a --context flag was passed)
- `hld` → read `context/hld.md`, summarise to max 500 words.
- `regression` → read `context/regression-scripts.md`, same.
- Both flags → produce ONE combined summary, max 500 words total.
- No --context flag → skip this step entirely. Do NOT read the files.
Store the result as `context_summary` (or empty string if this step was skipped).

### 4. Load the schema
Invoke the `ado-schema` skill once. I use it in steps 6–7 only: type codes for
the Title ids, column order and formatting for the file. `tc-subagent` preloads
its own skills — I do NOT read or pass the schema to it.

### 5. Spawn TC subagents in parallel
Spawn the `tc-subagent` subagent, by name, once per PBI, simultaneously. Pass
`{ pbi_id, pbi_title, pbi_description, acceptance_criteria, parent_epic_title,
   context_summary }`.
`context_summary` = the 500-word summary from step 3, or "".

### 6. Merge and validate
Flatten every returned array. Remove duplicates (same Title). Validate: every
object has all required fields and `Steps` is a non-empty array. Flag any object
missing a field — log it, do not drop it. Assign the id into the Title:
`[TC-{pbi_id}-{CODE}-{seq}]` per PBI per type (zero-padded seq). Leave the ID
column blank.

### 7. Write the file — one row per step
Filename `output/tc_YYYY-MM-DD.csv` (today's date). Encoding UTF-8 with BOM
(`utf-8-sig`) — without it Excel shows the bullet and dash as garbage. Header row, exact:
`ID,Work Item Type,Title,State,Assigned To,Priority,Tags,Area Path,Automated Status,Descriptiontep Action,Step Expected Result`
- First step row of each TC: fill every column — ID blank, Work Item Type
  `Test Case`, Title, State `Design`, Assigned To blank, Priority, Tags, Area
  Path blank, Automated Status `Not Automated`, Description (preconditions),
  Test Data, Test Type, Step # `1`, Step Action, Step Expected Result.
- Every following step row: blank EXCEPT Step #, Step Action, Step Expected
  Result.
- Quote any field containing a comma, quote or newline; double internal quotes.
- No cell may start with `=`, `+`, `-` or `@` (spreadsheets show `#NAME?`).
  Description bullets use `• `. In any other column, prepend a single space.
Then print a summary table: total TCs and total CSV rows; breakdown by type
(Positive / Negative / Edge Case / Performance / Boundary); breakdown by PBI;
the file path; and the hint: "Review the file, then run:
push-test-cases --file output/tc_YYYY-MM-DD.csv".

## When the user runs push-test-cases

The user runs `/push-test-cases --file output/tc_YYYY-MM-DD.csv`.
This is the only command that writes to Azure DevOps, and it always asks first.

### 1. Read and validate the CSV
Read the file at the path given by `--file`. Confirm it has the required ADO schema
columns (the 15 columns from the `ado-schema` skill). Count the test cases — rows whose
Work Item Type is `Test Case`. Tell the user how many will be pushed.

### 2. Confirm with the user
Print exactly: "About to push N test cases to ADO and link them to their PBIs.
Confirm? (yes/no)"
Wait for the answer. If it is not yes, abort — write nothing.

### 3. Spawn the push subagent
Resolve the ADO project from the `ADO_PROJECT` environment variable. Spawn the
`push-agent` subagent via the Task tool, by name, with
`{ "csv_file_path": "<path>", "project": "<ADO project>" }`.
Print the final summary: created N, linked N, failed: [list].

## Critical token-efficiency rules
- `ado-fetcher`, `tc-subagent` and `push-agent` are registered agents
  (`.claude/agents/*.md`). Always spawn them explicitly by name — never rely on
  auto-delegation.
- NEVER read the context files unless `--context` was passed.
- One subagent per PBI — never pass the whole batch to one.
- Never paste a skill's text into a subagent prompt — subagents get theirs through
  `skills:` frontmatter.
- Summarise context to 500 words BEFORE passing it to subagents.
- The fetcher returns slim JSON only — HTML stripped, max 300 words per PBI.

## Error handling
- Fetch failed for a PBI: log `[SKIP] PBI-{id}: fetch failed` and continue.
- A subagent returns an empty array: log `[WARN] PBI-{id}: no TCs` and continue.
- A subagent returns malformed JSON: log `[ERROR] PBI-{id}: invalid output` and
  continue.
- Never abort the whole run for one PBI.