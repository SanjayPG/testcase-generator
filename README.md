# Test Case Generator

A multi-agent Claude Code project that reads Product Backlog Items (PBIs) from Azure DevOps (ADO),
generates structured test cases for each one, writes them to a CSV for human review, and then pushes the
reviewed CSV back to ADO as Test Case work items linked to their PBIs.

You drive it with two slash commands:

| Command | What it does |
|---|---|
| `/generate-test-cases --pbis 123,456` | Fetch the PBIs, generate test cases, write `output/tc_YYYY-MM-DD.csv`. Writes nothing to ADO. |
| `/push-test-cases --file output/tc_YYYY-MM-DD.csv` | Push the reviewed CSV to ADO. Always asks for confirmation first. |

---

## How it works

```
CLAUDE.md (coordinator)
├── .claude/commands/generate-test-cases.md   ← the /generate-test-cases slash command
├── .claude/commands/push-test-cases.md       ← the /push-test-cases slash command
├── .claude/agents/ado-fetcher.md             ← reads PBIs and parent Epics from ADO (read only)
├── .claude/agents/tc-subagent.md             ← generates test cases, one per PBI, in parallel
├── .claude/agents/push-agent.md              ← creates the Test Cases in ADO and links them to PBIs
├── .claude/skills/ado-schema/                ← CSV columns and naming rules
├── .claude/skills/tc-generator/              ← how to write good test cases
├── contexts/                                 ← optional HLD and regression notes
└── output/                                   ← generated CSV files land here
```

The generate flow: parse the PBI ids, fetch them through `ado-fetcher`, optionally summarise the
context files, spawn one `tc-subagent` per PBI in parallel, then merge, validate and write the CSV.
The coordinator never generates test cases itself and never calls ADO directly.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) (`npm install -g @anthropic-ai/claude-code`)
- Node.js 18+ (the ADO MCP server is started through `npx`)
- An Azure DevOps project with PBIs, and a Personal Access Token (PAT) with these scopes:
  - **Work Items** → Read & write
  - **Test Management** → Read & write

---

## Setup

### 1. Get the three values

None of these go into a file in this repo.

| Variable | What it is | Where to find it |
|---|---|---|
| `ADO_ORG` | Organization name | `https://dev.azure.com/<ADO_ORG>` |
| `ADO_PROJECT` | Project name | `https://dev.azure.com/<ADO_ORG>/<ADO_PROJECT>` |
| `ADO_PAT_B64` | Your PAT, base64-encoded | Created below |

**Create the PAT**

1. Sign in at `https://dev.azure.com/<ADO_ORG>`.
2. Click **User settings** (top right) → **Personal access tokens** → **+ New Token**.
3. Name it (for example `tc-generator`), pick your organization, and set a short expiry.
4. Under **Scopes** choose **Custom defined** and enable **Work Items: Read & write** and
   **Test Management: Read & write**.
5. Click **Create** and copy the token now. ADO never shows it again.

**Encode it as base64**

The MCP server expects the base64 of `<email>:<pat>`, not the bare PAT.

PowerShell:
```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("you@company.com:YOUR_PAT"))
```

bash / Git Bash:
```bash
printf '%s' 'you@company.com:YOUR_PAT' | base64 | tr -d '\n'
```

### 2. Create `.mcp.json`

`.mcp.json` is listed in `.gitignore`, so it is **not** in a fresh clone. Create it in the project root
with exactly this content. Nothing in it is hardcoded: `${ADO_ORG}`, `${ADO_PAT_B64}` and
`${ADO_PROJECT}` are expanded by Claude Code from your environment when it starts.

```json
{
  "mcpServers": {
    "ado": {
      "command": "npx",
      "args": [
        "-y",
        "--package=@azure-devops/mcp",
        "mcp-server-azuredevops",
        "${ADO_ORG}",
        "--authentication",
        "pat"
      ],
      "env": {
        "PERSONAL_ACCESS_TOKEN": "${ADO_PAT_B64}",
        "ado_mcp_project": "${ADO_PROJECT}"
      }
    }
  }
}
```

Never put the real PAT, organization or project in this file.

### 3. Set the environment variables

Set all three **before** you start Claude Code, in the same terminal. The coordinator also reads
`ADO_PROJECT` to tell the subagents which project to use.

PowerShell (this session only):
```powershell
$env:ADO_ORG = "your-org"
$env:ADO_PROJECT = "your-project"
$env:ADO_PAT_B64 = "<base64 from step 1>"
```

PowerShell (saved for your Windows user, open a new terminal afterwards):
```powershell
setx ADO_ORG "your-org"
setx ADO_PROJECT "your-project"
setx ADO_PAT_B64 "<base64 from step 1>"
```

bash / zsh / Git Bash:
```bash
export ADO_ORG="your-org"
export ADO_PROJECT="your-project"
export ADO_PAT_B64="<base64 from step 1>"
```

### 4. (Optional) Add context files

Put these in `contexts/` if you have them. Plain text or markdown only. They are summarised to 500 words
before use, and only read when you pass `--context`.

- `contexts/hld.md`: High Level Design notes
- `contexts/regression.md`: existing regression test names and descriptions

---

## Start it

1. Open a terminal in the project folder, with the three variables set (step 3).
2. Start Claude Code:

   ```bash
   claude
   ```

3. Check the ADO server: run `/mcp` and confirm `ado` shows as **connected**.
   If it times out or fails, the usual causes are a variable that is unset in this terminal, a PAT that
   was encoded without the `email:` prefix, or an expired PAT. Fix it, then use `/mcp` to reconnect.
4. Run a command from the table below.

---

## Generate test cases

```
/generate-test-cases --pbis 123,456
/generate-test-cases --pbis 123,456 --context hld
/generate-test-cases --pbis 123,456 --context hld,regression
```

| Flag | Meaning |
|---|---|
| `--pbis` | Comma-separated PBI ids (integers). Required. |
| `--context` | Optional. `hld`, `regression`, or both. Without it, the context files are not read. |

The result is `output/tc_YYYY-MM-DD.csv`, plus a summary in the terminal: total test cases, CSV rows,
the breakdown by type and by PBI, and the file path. This step does not write to ADO.

Each PBI gets at least 2 Positive, 3 Negative, 2 Edge Case and 1 Boundary test case, plus a Performance
test case when the PBI touches an API or a database.

---

## Push to Azure DevOps

Open the CSV, review and edit it, then:

```
/push-test-cases --file output/tc_YYYY-MM-DD.csv
```

The coordinator validates the CSV, tells you how many test cases it will push, and asks
`Confirm? (yes/no)`. Only on `yes` does `push-agent` create the Test Case work items and link each to its
PBI. The summary reports created, linked and failed. This is the only command that writes to ADO.

Pushing creates real work items and this project has no delete tool, so run it against a project you are
happy to clean up.

---

## CSV format

One row per test step, matching the ADO import format. The test case columns are filled on the first step
row only; later step rows fill only `Step #`, `Step Action` and `Step Expected Result`.

```
ID, Work Item Type, Title, State, Assigned To, Priority, Tags, Area Path, Automated Status,
Description, Test Data, Test Type, Step #, Step Action, Step Expected Result
```

- Titles look like `[TC-123-NEG-01] Login returns error when password is incorrect`.
- Type codes: `POS`, `NEG`, `EDGE`, `PERF`, `BOUND`.
- UTF-8 with BOM so Excel shows the bullets and dashes correctly.
- No cell starts with `=`, `+`, `-` or `@`, to avoid spreadsheet formula errors.

The full rules live in `.claude/skills/ado-schema/SKILL.md`.

---

## Permissions

`.claude/settings.local.json` enables the `ado` MCP server for this project. Claude Code will ask before
each ADO tool call the first time. To avoid the prompts, allow the tools in `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__ado__wit_work_item",
      "mcp__ado__wit_work_item_write",
      "mcp__ado__testplan_test_case_write",
      "mcp__ado__search_workitem",
      "Read",
      "Write",
      "Agent"
    ]
  }
}
```

The two `*_write` tools are only used by the push. Leave them out if you want every push write to
prompt.

---

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `ado` not connected in `/mcp` | A variable is unset in this terminal, a PAT without the `email:` prefix, or an expired PAT. Fix it and reconnect with `/mcp`. |
| `/generate-test-cases` is not found | Claude Code was started outside this folder. Start `claude` in the project root. |
| Every PBI shows `[SKIP] ... fetch failed` | Wrong `ADO_PROJECT`, or the PBI id is in another project. |
| Excel shows `â€¢` | The CSV was re-saved without the BOM. Re-generate it, or open it with UTF-8 encoding. |

---

## Never commit

- `.mcp.json` if you ever put a literal value in it (it is git-ignored for that reason)
- A PAT, in any file

See [`CLAUDE.md`](./CLAUDE.md) for the full coordinator plan.
