# Contributing

This document explains what this project is and the rules for contributing to it. Everyone who works in this repository follows it, whether a person or an AI coding agent. GitHub shows a link to this file to anyone who opens a pull request or creates an issue.[1]

**Contents**

1. [Claude Code, and where to ask about it](#1-claude-code-and-where-to-ask-about-it)
2. [What this project is for](#2-what-this-project-is-for)
3. [How the documents fit together](#3-how-the-documents-fit-together)
4. [Contribution workflow](#4-contribution-workflow)
5. [Coding standards](#5-coding-standards)
6. [Failure rules](#6-failure-rules)
7. [Logging rules](#7-logging-rules)
8. [Repository guards and CI](#8-repository-guards-and-ci)
9. [Contributing with an AI coding agent](#9-contributing-with-an-ai-coding-agent)

## 1. Claude Code, and where to ask about it

Claude Code is Anthropic's agentic coding tool. It runs in a terminal, a desktop app, the web and IDE extensions. It reads and edits files, runs commands, and is extended through hooks, MCP servers, skills and instruction files.[2]

This project is a custom build on top of Claude Code: a memory store and guard hooks, packaged as a Claude Code plugin. It does not change Claude Code and does not replace Anthropic's documentation. For anything about Claude Code itself, use the sources below. They are all published by Anthropic.

| Question | Where to look |
|---|---|
| What Claude Code is, and how to install it | [Overview](https://code.claude.com/docs/en/overview), [Quickstart](https://code.claude.com/docs/en/quickstart), [Setup](https://code.claude.com/docs/en/setup) |
| How it works inside | [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works) |
| Common problems | [Troubleshooting](https://code.claude.com/docs/en/troubleshooting), [Error reference](https://code.claude.com/docs/en/errors) |
| Settings and permissions | [Settings](https://code.claude.com/docs/en/settings), [All settings](https://code.claude.com/docs/en/settings-reference), [Permissions](https://code.claude.com/docs/en/permissions), [Permission modes](https://code.claude.com/docs/en/permission-modes) |
| Hooks | [Hooks reference](https://code.claude.com/docs/en/hooks), [Hooks guide](https://code.claude.com/docs/en/hooks-guide) |
| MCP servers | [MCP](https://code.claude.com/docs/en/mcp) |
| Skills and plugins | [Skills](https://code.claude.com/docs/en/skills), [Plugins overview](https://code.claude.com/docs/en/plugins/overview), [Plugins reference](https://code.claude.com/docs/en/plugins-reference), [How plugins load](https://code.claude.com/docs/en/plugins/loading) |
| Instruction files the agent loads (`CLAUDE.md`, `AGENTS.md`) and auto memory | [Memory](https://code.claude.com/docs/en/memory) |
| Where every file lives | [The .claude directory](https://code.claude.com/docs/en/claude-directory) |
| Command-line flags and scripted runs | [CLI reference](https://code.claude.com/docs/en/cli-reference), [Headless](https://code.claude.com/docs/en/headless) |
| Release notes | [Changelog](https://code.claude.com/docs/en/changelog) |
| Source code, releases, and bugs in Claude Code itself | [anthropics/claude-code on GitHub](https://github.com/anthropics/claude-code) |
| The documentation, for an AI agent | [Documentation map](https://code.claude.com/docs/en/claude_code_docs_map.md) |

Three rules follow from this:

- **A question about stock Claude Code behaviour goes to Anthropic's documentation**, not to an issue here.
- **A bug in Claude Code itself is reported upstream**, in Anthropic's repository. Link it from an issue here if it blocks work.
- **Do not rely on third-party sites about Claude Code.** Treat any other write-up as unverified until Anthropic's documentation confirms it.

## 2. What this project is for

The goal is an agent whose answers can be relied on without re-checking them. It is pursued through four design goals; the limits of each are stated in ARCHITECTURE.md.

1. **Remember.** What was said, decided or learned is stored outside the model and searched on every message, so the agent answers from the record and not from what happens to be in its context.
2. **Verify before stating.** A factual statement is backed by something the agent looked up in that turn: a document, a file, a command result, or a stored fact with its date. When it cannot verify something, it says so.
3. **Be measurably right.** Accuracy is enforced by code and measured by an eval set. It is not requested in a prompt and assumed.
4. **Fail loudly and say how.** Every point of failure has a name. A failure, an empty result and a fallback can always be told apart, and none of them passes silently.

| Property | What delivers it | Where it is described |
|---|---|---|
| Remember | A memory store, searched on every prompt | ARCHITECTURE.md sections 2–4 |
| Verify before stating | Guards that block an action, or send the model back, until the required lookup has run | ARCHITECTURE.md section 5 |
| Be measurably right | Tests, the retrieval eval set, and the coding standards | ARCHITECTURE.md section 3.8; sections 4–5 here |
| Fail loudly | Failure codes, the `events` table and the daily failure report | ARCHITECTURE.md section 6; sections 6–7 here |

The same standard applies to contributions. A change that makes the agent sound more certain without making it more verifiable is rejected.

## 3. How the documents fit together

| Document | What it holds | Read it |
|---|---|---|
| `CONTRIBUTING.md` (this file) | The rules and the workflow | First, in full |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | How the build works: memory, retrieval, the store, guards, failure handling, logging | The sections for the part you are changing |
| [`OPERATIONS.md`](OPERATIONS.md) | Where the build runs, machine setup, scheduled jobs | Before changing anything about deployment or jobs |

Conventions used in all three:

- **Placeholders.** Values in angle brackets are filled in by the reader. Deployment values, such as `<store>` or `<judge-model>`, are set outside the repository and listed in OPERATIONS.md section 1. Workflow placeholders, such as `<branch>` or `<n>`, are defined where they are used. Real hostnames, account names, connection strings and credentials never appear in the repository.
- **Claude Code facts are versioned.** A statement about how Claude Code behaves names the Claude Code version it was checked against. Re-check it before relying on it with a newer version.

## 4. Contribution workflow

This repository uses GitHub, squash-merge as the only merge method, and `main` as the default branch. Maintainers merge.

Placeholders used below: `<owner>/<repo>`, `<branch>`, `<n>` (an issue or pull request number), `<type>` (a commit type such as `feat` or `fix`).

### 4.1 Rules

- **W1. Issue first.** Every pull request answers one issue. If none exists, create it before writing code: what was observed or asked for, and what done looks like. The issue is the whole scope. No change is too small for one. The one exception is a pull request Dependabot opens to move a pinned action (section 8.2).
- **W2. One branch per issue.** Name it `<type>/<short-description>`, cut it fresh from an up-to-date `main`, never reuse it for a second issue, and delete it once its pull request merges. `main` is the only long-lived branch.
- **W3. Test first, where a test can express it.** Write the test, watch it fail for the right reason, then write the minimum that passes. Write an error handler only when a test names the failure it handles. When a test cannot express the change, say so and report what was measured instead.
- **W4. One concern per commit**, with a conventional prefix: `feat:`, `fix:`, `refactor:`, `test:`, `chore:`, `docs:`, `ci:`.[4] A commit that needs "and" in its subject is two commits.
- **W5. Small pull requests.** One purpose each, with the tests for the change in the same pull request.
- **W6. Review before opening.** Review the branch's whole diff against `main`. Fix every must-fix finding on the branch and review again, until a round finds nothing or the remaining findings are recorded as open. Record in the pull request who reviewed, how many rounds, and the verdict.
- **W7. The pull request says what was actually done.** A conventional-commit title, `Closes #<n>`, the commits by concern, a Review section, a Validation section, and a "Found, not fixed" list. A pull request Dependabot opens keeps its own description (section 8.2).
- **W8. Maintainers merge, by squash, and only on green.** The maintainer reads `gh pr checks <n>` and merges only when every check passed. Contributors never merge a pull request, never enable auto-merge, and never push to `main`. Where branch protection is unavailable (section 8.1), a red check does not block a merge by itself, so this reading is the gate.
- **W9. Never rewrite or delete without a maintainer's yes.** That covers a hard reset, a force-push, and deleting a branch, local or remote.
- **W10. Maintainers change repository, account and database settings.** Propose the change with what it does, the risk and the rollback. Read the setting back after it is made.
- **W11. Verify before claiming.** Every statement about remote state (merged, CI green, a setting saved) comes from a fresh read, not from memory or from someone else's report.
- **W12. Scope is the issue.** Write no code for an edge case or a "what if" the issue does not name. Anything else you notice goes under "Found, not fixed", and each item ends with the issue that tracks it. Every item gets an issue. An item that needs no action is still filed, then closed as not planned.
- **W13. Never commit secrets**, in code, tests or commit messages.
- **W14. Leave the repository clean.** End every task with no uncommitted edits, no stashes and no untracked files.

### 4.2 The flow

```mermaid
flowchart TD
  subgraph P["Prepare"]
    A["Issue: what was observed, what done looks like"] --> B["Fresh branch from an updated main"]
    B --> C["Failing test, then the change, one concern per commit"]
    C --> D["Tests and lint pass locally"]
    D --> E["Review the whole diff, fix must-fix findings"]
  end
  subgraph Q["Pull request"]
    E --> F["Push and open the pull request"]
    F --> G{"CI checks"}
    G -- fail --> H["Read the log, fix the cause on the branch"]
    H --> G
    G -- pass --> I{"A maintainer reviews"}
    I -- comment or close --> J["Read every comment, rework from the failing test"]
    J --> C
  end
  subgraph R["After the merge"]
    I -- squash-merge on green --> K["Fetch, then update main"]
    K --> L["Check the branch is fully in main"]
    L --> M["Delete the branch"]
    M --> N["End-of-task check: only main is left"]
  end
```

### 4.3 Creating an issue

```bash
gh issue create --title "<what is wrong or wanted>" --body-file <file>
```

Issue body:

```markdown
**Observed or asked:** <the log line, the failing command, or the request>

**Done looks like:** <what can be checked when it is finished>

**On failure:** <crashes loudly, the default | keeps running, because <reason>; the failure is recorded as <failure code>>

**For a check or a guard:** <fails open | fails closed>, because <why>
```

Keep the issue number. The pull request needs it for `Closes #<n>`.

### 4.4 Branch and commits

Contributors with write access branch in this repository. Everyone else forks it and branches in the fork. In a fork, add this repository as a remote named `upstream` (`git remote add upstream https://github.com/<owner>/<repo>.git`) and read `upstream` wherever the commands in this section and in 4.6–4.7 say `origin` for `main`; push branches to `origin`, the fork.

```bash
git fetch --prune
git switch main
git merge --ff-only origin/main
git switch -c <type>/<short-description>
```

- Always fetch before the fast-forward. Without it git reports "Already up to date" and `main` stays behind.
- Stage by path. Never `git add -A` or `git add .`.
- The `.gitignore` is an allow-list (section 8), so a new file in a path it does not list is invisible to `git status`. `git add <path>` refuses such a path ("ignored by one of your .gitignore files"), and `git add -A` skips it silently. When a path is refused, add a line for it to `.gitignore` in the same pull request; never use `git add -f`.
- Before pushing, run `git status --short --ignored`. Every `!!` entry must be a cache or build directory, such as `.pytest_cache/` or `.ruff_cache/`. A file or directory of yours listed there is hidden by the allow-list and is not in your commits.
- Switch on the repository's pre-commit hook once in each clone: `git config core.hooksPath .githooks`. It runs gitleaks over what is staged and refuses a commit that carries a secret (section 8); without gitleaks on `PATH` it refuses every commit. Install gitleaks at the version the CI workflow pins, from its GitHub release, and check the archive against the release's checksums file. Never skip the hook with `--no-verify`.
- When the tests come first, commit them on their own with "(red)" in the subject, then commit the change that makes them pass. The squash-merge keeps the failing commit off `main`.
- Run the full test suite and the linter before each commit that changes behaviour.

### 4.5 Opening the pull request

Write the body to a file outside the repository and pass it with `--body-file`. Drop a line of the template that does not apply, and say why.

```bash
git push -u origin <branch>
gh pr create --base main --head <branch> --title "<type>: <what changed>" --body-file <file>
```

```markdown
Closes #<n>

<Two or three sentences: what changed and why.>

## Commits
- `<hash>` <subject>. <What it touches.>

## Review
<Who or what reviewed, rounds, verdict. Findings fixed on the branch. Findings left open.>
- Error handling: <n> handlers added, each naming <types> and emitting <failure codes>.
- Failures and empty results: <how a caller tells them apart, or "no fallback added">.

## Validation
- Red first: `<test command>`: <n> failed, <n> passed, and why they failed.
- Green: `<test command>`: <n> passed. `<lint command>`: clean.
- Rendered or run: <what, where, by whom>.
- Not verified: <what was not checked>.

## Found, not fixed
- <Thing noticed outside the issue's scope.> #<n>
- <A thing that needs no action.> #<n> (closed, not planned)
```

Closing keywords such as `Closes` only work when the pull request targets the default branch.[5]

Before opening the pull request, file an issue for every found-not-fixed item, or add a comment to the issue it belongs with. When nothing was found, write "Nothing found".

```markdown
**Found in:** PR #<n>, under "Found, not fixed" (<date>).

**What:** <what was seen, and whether it was run or only traced>

**Why this issue exists:** <why it is worth recording>

**Severity:** <high | medium | low | none>

**Impact:** <who or what it affects>

**Does it need addressing?** <yes | later | a decision from <name> | no>
```

### 4.6 Reading the result

```bash
gh pr checks <n>
gh run view <run-id> --job <job-id> --log        # when a check fails: read the log, find the cause
gh pr view <n> --json state,mergedAt,mergeable,mergeStateStatus
```

Read all three places a reviewer can leave feedback:

```bash
gh api repos/<owner>/<repo>/issues/<n>/comments     # the conversation
gh api repos/<owner>/<repo>/pulls/<n>/reviews       # review verdicts
gh api repos/<owner>/<repo>/pulls/<n>/comments      # comments on lines
```

- **A pull request closed without merging, with a comment, is a rejection.** Rework it for what the comment asks, on a fresh branch from the current `main`, under the same issue with its scope updated.
- **When more is asked while the pull request is open, and it belongs to the same issue:** update the issue first, then add commits to the same branch, test first as before. Undo a pushed change with `git revert`, never with a reset or a force-push. Rewrite the pull request body after each push, and read CI on the new head commit.
- **A request that is a different concern** is a new issue and a new branch.
- **When the branch is out of date with `main`:** update it with a merge commit (`git fetch`, `git merge origin/main`, `git push`). Never rebase a pushed branch.
- **When a check fails:** fix the cause, not the check. If a scanner flags a line that is safe, trace why it is safe, mark that one line with the scanner's own marker and the reason, and say so in the pull request. This never applies to the lint rules in C2 (`BLE001`, `S110`, `E722`).
- **When `--log` returns nothing:** the job never started. Run `gh run view <run-id>` and read the annotations. If the cause is the environment and not the code, report it to a maintainer. Do not re-run the job or edit the workflow to get around it.

### 4.7 After the merge

1. Confirm the merge with `gh pr view <n> --json state,mergedAt,mergeCommit`. Then `git fetch --prune`, `git switch main`, `git merge --ff-only origin/main`.
2. Check the branch is fully in `main`. Print the file list first, `git diff --name-only origin/main...<branch>`, and make sure it is not empty. Then
   `git diff origin/main <branch> -- $(git diff --name-only origin/main...<branch>)` must print nothing.
   If it prints anything, stop and tell a maintainer.
3. Delete the branch, local and remote, with a maintainer's yes (W9). Give the tip hash so it can be restored. GitHub deletes the remote branch itself when automatic deletion is on (section 8.1).
4. Confirm CI passed on the merge commit: `gh run list --workflow CI --commit <merge-sha> --json status,conclusion`, where `<merge-sha>` is the full `mergeCommit` hash from step 1.
   - It must print a run whose `conclusion` is `success`.
   - An empty list (`[]`) is not a pass. The run has not started yet, or the hash is shortened or wrong: the command needs the full hash and exits 0 either way. Run it again, and tell a maintainer if it stays empty.
   - Do not read the newest run on `main` instead. Other workflows, such as GitHub's own Dependency Graph, run on the same commit and can be listed first.
   - If CI failed in the code, tell a maintainer at once and propose a revert pull request.
5. End-of-task check: `git status -sb`, `git branch -vv`, `git stash list`. Only `main` should be left. Then `git status --short --ignored`: every `!!` entry must be a cache or build directory (4.4). Anything else there was never committed; tell a maintainer.

### 4.8 What a squash-merge changes

- **A merged branch looks unmerged.** `git branch --merged main` does not list it. Use the files-identical check in 4.7.
- **A plain `git branch -d` may succeed or refuse**, depending on whether the remote branch still exists.[6] Do not use it to probe.
- **Never keep working on a squash-merged branch.**[7]
- **Keep pull request titles in the same style as commit subjects.** The squash commit on `main` takes its name from one or the other.[8]
- **Undo with a revert pull request**, never by rewriting `main`.[9]
- **If a force-push is ever approved**, use `--force-with-lease=<branch>:<expected-sha>`. The bare form is unsafe when anything fetches in the background, because the lease then checks against the freshly fetched tip.[10]

## 5. Coding standards

### 5.1 Rules

- **C1. Catch the error you expect, by name.** Before writing a handler, look up what the call raises, in the library's documentation or its source, and name that type. The `try` covers only the lines that can raise it. Imports, formatting and plain data handling stay outside.
- **C2. No catch-everything blocks.** Never write `except Exception`, `except BaseException` or a bare `except:`. Catch by name, or let it crash. For cleanup, use `try`/`finally`. Never silence the lint for this rule, by `noqa` comment, by configuration, or by excluding files; CI fails on each (section 8.2). The only exceptions are the two shared wrappers in the failure module (ARCHITECTURE.md sections 2 and 6.1): the boundary wrapper that every entry point Claude Code calls goes through (each hook script and each MCP tool handler), and the job wrapper that every scheduled job and the startup check go through. Each catches everything, records the failure with its traceback, and then blocks, returns a failure note or exits non-zero, as its entry point requires. They log with `logger.exception`, which ruff accepts without a `noqa`.[11] No other code catches everything. The boundary wrapper matters more here than in most projects: Claude Code carries on with the action when a hook crashes, times out or exits with any code other than 0 or 2,[3] so a guard that simply crashed would let its call through. The wrapper turns a crash in an enforcing guard into a block.
- **C3. A failure never looks like an empty result.** Do not return `[]`, `None`, `0` or "skipped" for a failure when the same value also means "nothing found". Raise, or return a failure the caller has to check. A missing setting is a startup error, never a silent default.
- **C4. Let it crash.** Code fails loudly by default. Add handling that keeps code running only when the issue asks for it and gives the reason, and record the failure even then (section 6). "Must not crash" never means "must not show the failure".
- **C5. No bare "never raises", "fail open" or "best-effort".** In an issue, a brief, a docstring or a comment, follow each of those phrases with which failures it covers, the failure code that records them, and how the caller tells a failure from an empty result. A spec that says only "must never raise" invites a silent catch-everything handler.
- **C6. Prove every guard and every check.** It counts as working only after it has been seen to block its own failing case. A check must fail when its tool is missing or broken. A guard's enforcement step blocks: it denies the call, blocks the prompt or blocks the stop. It never only warns. Reminders and `log` mode come before enforcement, not instead of it.
- **C7. Data this repository did not create is not ours to change.** In a database that holds tables, functions or data that this repository's migrations did not create, create new objects under new names, and never alter, rename or drop the existing ones. Objects this repository created change only through new forward migrations. Do not connect with a credential that can delete until a maintainer confirms an export exists.
- **C8. Retrieval changes come with numbers.** A change to chunking, embedding, fusion, reranking, recall delivery or any default in ARCHITECTURE.md section 3.6 reports the eval-set results before and after (ARCHITECTURE.md section 3.8).
- **C9. Code taken from elsewhere meets these rules before it merges.** Code copied or ported from another project, the earlier Claude Code pack, or a generated draft has every handler narrowed to a named error with a failure code, or removed, in the same pull request.
- **C10. A migration works with the build installed before it.** A migration is applied to the store separately from the update of the installed build, and before it (OPERATIONS.md section 7); reinstalling an older build does not undo it. So a migration adds only what an older build can ignore (a table, a nullable column, a new function, a grant), and changes or takes away nothing an installed build uses: no required column, no tighter constraint or policy, no replaced function under the same name. Changing or removing something an installed build uses, such as a rename, a new type or a drop, takes three pull requests: add the new shape, move the code to it, then remove the old shape once no install runs a build that uses it. A pull request with a migration says why the build on `main` before it still works; one that removes something also says that no install runs a build that uses it. This rule covers the files in `migrations/`. Changing the embedding model, which OPERATIONS.md section 3 also calls a migration, is not such a file and cannot meet this rule: every vector is re-embedded, and an install still on the old embedding model refuses to run recall until it is updated, so the re-embedding and the model change are done together.

### 5.2 Why error handling gets its own rules

A handler is a mute rule on an alert. Muting one alert you expect is fine, as long as it is still counted. Muting every alert from a server also mutes the outage.

Code that catches everything turns a real bug into a normal-looking result: an empty list, a "skipped" count, a default value. The program finishes, the tests pass, and the failure is found days later, if at all. For an agent whose purpose is to be trusted, a hidden failure is the worst kind. A crash is visible and gets fixed.

This is a known weakness of AI-written code in particular. In one large study of AI-authored commits, broad exception handling was the most common code smell found.[12] These rules exist so that it is never written here in the first place.

### 5.3 Examples

**Wrong:**

```python
def search(query):
    try:
        from store import client
        rows = client.search(query)
        return [format_row(r) for r in rows]
    except Exception:
        return []        # an outage, a typo and "no matches" all look the same
```

**Right: narrow and loud.**

```python
from store import client, StoreTimeout
from failures import RecallFailed, record_failure

def search(query):
    try:
        rows = client.search(query)                  # the only line that can raise it
    except StoreTimeout as e:
        record_failure("recall.timeout", reason=str(e))   # logged and written to events (section 6)
        return RecallFailed("recall.timeout")             # the caller must handle it
    return [format_row(r) for r in rows]             # [] here really means "nothing found"
```

| What happened | Result |
|---|---|
| Nothing matched | `[]` |
| The store timed out (expected) | `RecallFailed("recall.timeout")`, logged, recorded, and shown in the turn |
| A bug (unexpected) | A crash, recorded as `unhandled`, with its traceback in the log |

**Also right:**

- Cleanup with `try`/`finally`, which needs no `except` at all.
- A handler for one named error that is part of normal operation, such as `FileNotFoundError` when deleting a file that may already be gone, provided the outcome is still recorded where a caller or the daily report can see it.

### 5.4 What the lint catches, and what review catches

The lint fails a pull request on:

- `except Exception` or `except BaseException` (`BLE001`).[11]
- `try`-`except`-`pass` on a catch-everything block (`S110`). By default it flags only `Exception`, `BaseException` and a bare `except:`.[13][14]
- A bare `except:` (`E722`).
- Any suppression of `BLE001`, `S110` or `E722`: a `noqa` comment, a file-level `ruff: noqa` or `flake8: noqa`, ruff configuration that turns a rule off, globally or per file, or that excludes files or lives outside the root `pyproject.toml` (separate CI steps, section 8.2).

The reviewer checks what the lint cannot:

- A catch-everything block that re-raises, or that logs with the traceback. The lint does not flag either one.[11] Rule C2 forbids both, apart from its two shared wrappers.
- A failure returned as an empty value (C3).
- A `try` that covers more lines than it needs (C1).
- A handler or fallback with no failure code (F1).
- "Never raises", "fail open" or "best-effort" with nothing after it (C5).
- A migration that an older installed build cannot ignore, or that removes something an installed build still uses (C10).

## 6. Failure rules

Every point of failure is loud and distinguishable. The mechanism, including the catalog of failure codes, is in ARCHITECTURE.md section 6.

- **F1. Every failure has a named code from the catalog.** A handler or fallback emits one. There is no bare "error".
- **F2. Failed, empty and degraded are different results.** A failure is never returned as an empty result, and a fallback or a retry that succeeded is recorded as degraded, never as plain success.
- **F3. Every occurrence is logged and recorded.** Each one is logged at WARNING or above and written to the `events` table, through the one recording helper in the failure module. Errors that reach an entry point Claude Code calls, a scheduled job or the startup check are recorded by the boundary wrapper or the job wrapper (ARCHITECTURE.md sections 2 and 6.1), so code never relies on Claude Code to report a failed hook.
- **F4. A failure that changed the answer is shown in the turn.** Every failure, shown or not, appears in the daily failure report.
- **F5. Recording a failure must not lose it.** If the `events` row cannot be written, the failure is logged locally at ERROR with `events.write_failed`.
- **F6. The catalog is enforced.** A test fails the build if a handler or fallback emits no code or a code not in the catalog. A failure that is not in the catalog is a bug, recorded, with its traceback in the log: in a hook or an MCP tool by the boundary wrapper, and in a scheduled job or the startup check by the job wrapper, which records it and then exits non-zero (ARCHITECTURE.md section 6.1).

## 7. Logging rules

Claude Code does not own logging for hook scripts or MCP servers: each is a separate process, and the build owns its log file, rotation and redaction (ARCHITECTURE.md section 7).

- **L1. One logger per module, configured in one place.** Use `logging.getLogger(__name__)`. Only the failure module's setup function adds handlers; no other code adds one or calls `logging.basicConfig()`, because a second handler bypasses the build's redaction. For free-text lines, pass values as `%`-style arguments, not f-strings, so formatting is deferred.[15]
- **L2. One rotating log file per machine**, outside the repository and outside any project directory. Every failure is logged at WARNING or above (F3). Nothing is logged to a hook's stdout or stderr: Claude Code hands stdout to the model as context on some events, and stderr to the model when a hook exits 2,[3] so both are outputs of the guard, not a log.
- **L3. Events go through one helper.** Decisions and failures are logged by the recording helper in the failure module, never by hand. It writes one line whose message is a single JSON object, and the matching `events` row, with the same fields: `code`, `project`, `session_id`, `turn`, `result` and `reason`, plus IDs, counts and scores. Extra fields that have an OpenTelemetry GenAI name (`gen_ai.operation.name`, `gen_ai.tool.name`, `error.type`) use it.[16]
- **L4. Redact before every write, not only before logging.** Every value written to the memory store, to the log or sent off the machine passes through the build's redactor first. A test proves it: a fake key sent through every path that writes to the store or sends text off the machine, the requests to the judge included, never appears in a log line, a stored row or an outgoing request.[17]
- **L5. No content in logs.** Prompts, replies, judge input, and fact or chunk text never appear in a log line. Log IDs, counts, scores and hashes. Untrusted strings appear only as truncated values inside the JSON object.[16][17]

## 8. Repository guards and CI

A rule in this file does not block anything. These guards do. Each is proven with its failing case before it is relied on (rule C6).

| Guard | Rules | Where it lives | The failing case that proves it |
|---|---|---|---|
| Lint over the whole repository: ruff's default rules plus `BLE001` and `S110` | C1, C2 | `pyproject.toml`, the CI workflow | A pull request that adds `except Exception: pass` anywhere fails |
| Suppression ban for `BLE001`, `S110`, `E722` | C2 | The CI workflow | A pull request that adds `# noqa: BLE001`, a bare `# noqa`, a file-level `# ruff: noqa`, ruff configuration that turns one of these rules off, an `exclude`, or a ruff configuration file outside the root `pyproject.toml`, fails |
| Tests in CI | W3 | The CI workflow | A pull request with a failing test fails |
| Actions pinned to a full commit SHA | 8.2 | The test suite | A step or a job in any workflow file whose `uses:` names a tag or a branch fails the build |
| Hook-configuration validation, added with the first hook | ARCHITECTURE.md 5.5 | The test suite | A plugin `hooks/hooks.json` that leaves out a guard's hook, or registers an enforcing guard without the boundary wrapper, fails the build |
| Failure-catalog test | F1, F6 | The test suite | A handler that emits no code, or an unknown one, fails the build |
| Secret-leak test | L4, W13 | The test suite | A fake key that reaches a log line, a stored row or an outgoing request fails the build |
| Secret scan with gitleaks, at a pinned version, as a pre-commit hook and as a CI job, and GitHub's own secret scanning | W13 | A git hook, a workflow, repository settings | A commit carrying a fake key is refused |
| Branch protection on `main` | W8, W9 | Repository settings; needs a public repository or a paid plan (8.1) | A direct push to `main` is rejected |
| An allow-list `.gitignore` | W13, W14 | The repository root | A credential or runtime file placed in the working tree does not show as untracked |
| Separate database roles for the build and for maintenance | C7 | The database | With the build's role, deleting a row or altering a table is refused |

There is no count ratchet: the repository starts with no catch-everything handlers, so the lint fails on the first one.

### 8.1 Repository settings

Branch protection and rulesets are available on public repositories, and on private repositories only with GitHub Pro, Team or Enterprise.[18][25] This repository is public. The first three rows are set in its settings, and the last four as one ruleset on `main` with no bypass list, configured, switched on and read back on 2026-11-17. It refused a direct push to `main` the same day, for the missing `tests` check (rule C6); its other rules have not yet been seen to block. GitHub's secret scanning and push protection are on. The ruleset enforces part of W8 and W9: a change reaches `main` only through a pull request, squash-merged, with the `tests` check passed, and `main` cannot be force-pushed or deleted. It does not enforce who merges, or W9 on other branches, and an account with admin rights can edit it, so those parts still rest on the maintainer and on how each agent is configured (section 9). If the repository is made private on GitHub Free again, the ruleset stops applying.

| Setting | Why |
|---|---|
| Allow squash merging only | Every pull request lands on `main` as one commit.[7] |
| Default squash title: "Pull request title" | One naming rule for commits on `main`.[8] |
| Automatically delete head branches | Removes the remote branch on merge. For a pull request from a fork, the fork's own setting applies.[19] |
| Require a pull request before merging | Nothing lands without one.[25] |
| No bypass list | In a ruleset, only those on its bypass list can bypass it; this one lists no one.[25] |
| Block force pushes and deletion | Protects the history.[25] |
| Require status checks to pass. Add each CI check after it has run once and passed. | Nothing merges on a red check.[20] |

### 8.2 The CI workflow

The workflow is `.github/workflows/ci.yml`, added by the repository's first pull request. Its job is named `tests`, the check the ruleset requires. It runs the same commands a developer runs locally, on every pull request to `main` and on every push to it. This section does not repeat the file, so the two cannot differ; the notes below say why its steps are the way they are.

Lint selection in `pyproject.toml`. Use `extend-select`, which adds to ruff's default rules and keeps `E722` on.[13][14]

```toml
[tool.ruff.lint]
extend-select = ["BLE001", "S110"]
```

- `ruff check .` covers every Python file in the repository, tests included. All ruff configuration lives in the root `pyproject.toml`: no `ruff.toml`, `.ruff.toml` or nested `pyproject.toml`, no `exclude` or `extend-exclude`, and no per-file ignores for these rules. With one configuration file, the probe step proves the rules are on for every file, and the coverage step proves every tracked Python file is linted, whatever would otherwise drop it (`include`, `.gitignore`, or ruff's default excludes such as `dist/`).
- A second `pyproject.toml` anywhere in the repository fails the workflow's configuration step.
- Pin ruff to one version in the project's dev dependencies, so local runs and CI agree.
- Pin every action to a full-length commit SHA, with the released version in a comment beside it: a tag can be moved to other code, a SHA cannot.[26] `.github/dependabot.yml` has Dependabot propose each new release as a pull request, so a pin does not go stale. That pull request is the one exception to W1 and W7: it has no issue and keeps Dependabot's own description. Its diff should change only a pinned SHA and the version comment beside it. The maintainer reads the diff and the checks, then merges it like any other. A test fails on a `uses:` that is not a SHA.
- The job name is what appears as the status check to require.
- Give any step that pipes a tool into a count or a test `shell: bash`, and never end it with `|| true`. Without `shell: bash` a failed command in the middle of a pipe does not stop the step, so a tool that did not start reads as a count of 0 and the check passes.[21] A check must tell "found nothing" from "the tool failed", as the workflow's `noqa` step does. Prove each step with its failing case before relying on it (C6).
- The secret scan runs gitleaks, at a pinned version checked against its published checksum, over the whole history (`fetch-depth: 0`), merge commits included. It fails on a finding and on any error gitleaks reports, because gitleaks exits 0 when git itself fails.
- Add a dependency audit when there are dependencies: `pip-audit` with no arguments.[22]

### 8.3 When a guard or a permission blocks you

Do not look for another route to the same result.

1. Stop. Say what was blocked and that nothing changed.
2. Give a maintainer one command to run, with what it does, the risk, and the rollback.
3. When it is done, read the state back and report it.

## 9. Contributing with an AI coding agent

AI coding agents are welcome contributors, under the same rules as everyone else.

- **Development rules and the build's own instructions never share a file.** When a Claude Code session runs inside a repository, it loads `CLAUDE.md` or `.claude/CLAUDE.md`, `CLAUDE.local.md` and `.claude/rules/` from the project, and the same files in a subdirectory once it works with a file there. From v2.1.277 it reads `AGENTS.md` too, by default only when there is no `CLAUDE.md`.[23] A file of that name in this repository is therefore read as instructions for developing the build. The build installs no instruction file: Claude Code does not load a `CLAUDE.md` from a plugin, so what the build tells the model in other projects is a skill in the plugin or context a hook adds,[27] and a session in this repository loads neither from the working tree. Developing here therefore never loads the build's instructions as development rules. On the default macOS file system `Agents.md` and `AGENTS.md` are the same file, so do not use either spelling.
- **This file is the one source of the rules.** If the repository gains a `CLAUDE.md`, it holds only an import of this file (`@CONTRIBUTING.md`),[23] so the rules are never written twice. Any other coding agent gets them from its own user-level instruction file: point it at this document, and copy in the rules from sections 4.1, 5.1, 6 and 7.
- **End every brief with the error-handling rule.** A brief that says code "must never raise" or must "fail open" invites a silent catch-everything handler (C5). Every brief given to a coding agent ends with this line, verbatim:

  > Error handling rule: catch the specific exception you expect, by name. Never return a value that also means "nothing found" or "skipped" for a failure; log every caught failure with its failure code, and let anything unexpected crash.

  The line restates C1–C4 and F1–F2. It is a request, not a guard: the lint and the tests in section 8 are what enforce the rules.
- **Configure the agent so it cannot do what rules W8 and W9 forbid, or change repository settings (W10).** In Claude Code, that is `permissions.deny` rules in the user's settings, outside this repository, for the commands that merge a pull request, push to `main`, force-push, reset hard, skip hooks, delete a branch, or change the repository's settings or rulesets. A deny rule matches the command text as the agent writes it, and Anthropic's documentation says it is not a security boundary: the same program called in another form gets past it.[24] A rule the agent can break is a request (ARCHITECTURE.md section 5.1). The ruleset in section 8.1 enforces only part of W8 and W9, and an account with admin rights can edit it, so this configuration carries the rest.
- **A person is accountable for every pull request**, whoever wrote the code.

## Sources

- [1] [Setting guidelines for repository contributors (GitHub Docs)](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors)
- [2] [Claude Code overview (Claude Code documentation)](https://code.claude.com/docs/en/overview)
- [3] [Hooks reference (Claude Code documentation)](https://code.claude.com/docs/en/hooks)
- [4] [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0)
- [5] [Linking a pull request to an issue (GitHub Docs)](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue)
- [6] [git-branch documentation](https://git-scm.com/docs/git-branch)
- [7] [About merge methods on GitHub (GitHub Docs)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github)
- [8] [Configuring commit squashing for pull requests (GitHub Docs)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/configuring-commit-squashing-for-pull-requests)
- [9] [Reverting a pull request (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/reverting-a-pull-request)
- [10] [git-push documentation](https://git-scm.com/docs/git-push)
- [11] [blind-except (BLE001) (Ruff documentation)](https://docs.astral.sh/ruff/rules/blind-except)
- [12] [Liu et al., Debt Behind the AI Boom, arXiv 2603.28592 version 3 (preprint)](https://arxiv.org/html/2603.28592v3)
- [13] [try-except-pass (S110) (Ruff documentation)](https://docs.astral.sh/ruff/rules/try-except-pass)
- [14] [Settings (Ruff documentation)](https://docs.astral.sh/ruff/settings)
- [15] [Logging HOWTO (Python 3.14 documentation)](https://docs.python.org/3.14/howto/logging.html)
- [16] [OpenTelemetry semantic conventions for generative AI](https://github.com/open-telemetry/semantic-conventions-genai)
- [17] [Logging Cheat Sheet (OWASP)](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [18] [About protected branches (GitHub Docs)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [19] [Managing the automatic deletion of branches (GitHub Docs)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-the-automatic-deletion-of-branches)
- [20] [About status checks (GitHub Docs)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks)
- [21] [Workflow syntax for GitHub Actions (GitHub Docs)](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
- [22] [pip-audit README](https://github.com/pypa/pip-audit/blob/main/README.md)
- [23] [How Claude remembers your project: CLAUDE.md, AGENTS.md and auto memory (Claude Code documentation)](https://code.claude.com/docs/en/memory)
- [24] [Configure permissions (Claude Code documentation)](https://code.claude.com/docs/en/permissions)
- [25] [About rulesets (GitHub Docs)](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [26] [Secure use reference (GitHub Docs)](https://docs.github.com/en/actions/reference/security/secure-use)
- [27] [Plugins reference (Claude Code documentation)](https://code.claude.com/docs/en/plugins-reference)
