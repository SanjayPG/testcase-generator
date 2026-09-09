# Test Case Generator

A Claude Code project that turns a user story into a structured set of test cases.

## What it does

Paste a user story — a title, a description, and acceptance criteria — and Claude
generates test cases for it in a fixed markdown format. Output is test cases only:
no preamble, no summary.

## Usage

1. Open this project in Claude Code (the `CLAUDE.md` in this repo configures the
   generation rules).
2. Paste your user story, including:
   - Title
   - Description
   - Acceptance criteria
3. Claude returns a set of test cases, one markdown block per case, separated by `---`.

### Example prompt

```
Title: Login with email and password

Description: As a registered user, I want to log in with my email and
password so that I can access my account.

Acceptance Criteria:
- Given a registered email and correct password, the user is logged in
  and redirected to the dashboard.
- Given a wrong password, the user sees an error and is not logged in.
- Given an unregistered email, the user sees an error and is not logged in.
```

See `login-with-email-and-password.testcases.md` for a full example of the
generated output.

## Test case format

Each test case includes:

- **Title:** `[<Type>] <behaviour> — <condition>`
- **Preconditions:** state required before the test
- **Steps:** numbered, concrete actions with literal values
- **Expected Result:** observable outcome, ending with the pass condition
- **Test Data:** concrete values used, as `field: value`
- **Type:** `Positive`, `Negative`, `Edge`, `Boundary`, or `Performance`

## Coverage rules

Per story, the generator produces at minimum:

| Type        | Minimum |
|-------------|---------|
| Positive    | 2       |
| Negative    | 3       |
| Edge        | 2       |
| Boundary    | 1       |
| Performance | 1 (only if the story touches an API or a database) |

Everything is derived from the story as given — no invented system details, and
no clarifying questions asked back.

See [`CLAUDE.md`](./CLAUDE.md) for the full generation rules.
