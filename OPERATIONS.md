# Operations

This document describes where the build runs and what must be configured for it to run correctly. It holds no real hostnames, account names, paths under a home directory or credentials: those are deployment values, set outside the repository. The design is in [ARCHITECTURE.md](ARCHITECTURE.md); the rules for changing anything are in [CONTRIBUTING.md](CONTRIBUTING.md).

**Status: nothing is deployed from this repository yet.** The build replaces an earlier pack that runs on the same machine today; until the cutover (section 8), that pack keeps running unchanged. Where a choice has not been made this document says "not decided".

Facts about Claude Code are stated for **Claude Code 2.1.322**, read from its documentation on 2026-11-17. Re-check them before upgrading Claude Code. A statement marked **[unverified]** was not read from the documentation.

## 1. Deployment values

These placeholders appear in the documentation. They are set on the machine, outside this repository.

| Placeholder | What it is |
|---|---|
| `<store>` | The connection string for the local memory database, for the build's own role |
| `<model-digest>` | The pinned build digest of the embedding model |
| `<judge-model>` | The hosted model used as judge, as reranker and to route each prompt. It is set by version, never by an alias that can move to a newer model, because the thresholds depend on its answers. |
| `<extraction-model>` | The model that proposes facts from user prompts and tool results |
| `<digest-model>` | The model that writes session digests |
| `<report-dir>` | The folder the daily failure report is written to |
| `<state-dir>` | The plugin's data directory, `${CLAUDE_PLUGIN_DATA}`, which holds the build's log file and local state database (ARCHITECTURE.md section 2) |

The plugin declares each value it needs as a `userConfig` option, and Claude Code asks for them when the plugin is enabled. It saves a value that is not sensitive in the user's `settings.json` and a value marked `sensitive` in the platform's secure credential store. Hook processes receive each one as an environment variable named `CLAUDE_PLUGIN_OPTION_<KEY>`, and the MCP server's configuration can reference it as `${user_config.KEY}`.[6] The scheduled jobs (section 4) are not started by Claude Code and receive none of this; how they read the values is not decided. Secrets never go in a file this repository tracks.

## 2. Where it runs

Each install runs on one machine, with a store of its own. The first is a Mac, and this document describes that one; installing for another user repeats section 3 on their machine. Everything except the hosted models is local.

| Part | Where |
|---|---|
| Claude Code and the plugin: its hook scripts and its MCP server | The machine |
| Embedding (`qwen3-embedding:4b`) | Locally, through Ollama, as the current pack does |
| Judge, reranker and prompt routing | `<judge-model>`, hosted, as the current pack does |
| Fact extraction and session digests | `<extraction-model>` and `<digest-model>`. The starting point is what the current pack uses, read from the pack in the inventory before this row is filled in. |
| The memory store | Local PostgreSQL 17 with pgvector, in the `claude_mem` database the current pack already uses; that is confirmed on the machine before the cutover (section 8). The build's tables live in a schema of their own, `memory`. The current pack's tables stay where they are and are never altered (CONTRIBUTING.md rule C7). |
| The eval set | Its own throwaway database, never `claude_mem` |
| Scheduled jobs | launchd on the machine (section 4) |

What each hosted part receives is listed in ARCHITECTURE.md section 3.7.

## 3. Machine setup

1. **Runs the pinned embedding build.** Its model digest must equal `<model-digest>`. The build compares its digest with the one recorded in the store and refuses to run recall if they differ (`embed.model_mismatch`). Upgrading the model is a migration: every vector is re-embedded and the recorded digest changes. It is not a file in `migrations/`, and rule C10 (CONTRIBUTING.md section 5.1) does not cover it.
2. **Connects with a role that cannot delete.** The database has one role today, a superuser. The build gets a login role of its own that joins the group role `memory_agent`, which the migrations create and grant: it can read and insert rows in the `memory` tables, update those that take updates, and cannot delete rows or change the schema (ARCHITECTURE.md section 4.3). A maintainer creates it once, outside the repository, with a generated password set through psql's `\password`, so that only the hash is sent.[4] Migrations and retention use other roles.
3. **Uses these Claude Code settings:**

   | Setting | Value | Why |
   |---|---|---|
   | `disableAllHooks` | Never set | With it set, no guard and no recall runs (ARCHITECTURE.md section 5.6).[1] |
   | The plugin | Enabled; never disabled or uninstalled for a session that should be guarded | With it off, no guard and no recall runs. |
   | `--bare`, `--safe-mode` | Never used for a session that should be guarded | Each skips hooks, plugins, skills, MCP servers, auto memory and `CLAUDE.md`.[3] |
   | `permissions.disableBypassPermissionsMode` | `"disable"` | It forbids `bypassPermissions` mode, which skips permission prompts. A hook's `deny` still blocks in that mode,[9] but the prompts Claude Code would raise by itself do not,[2] and whether a hook's `ask` still prompts there is not stated on the pages read. **[unverified]** |
   | `permissions.deny` | The commands CONTRIBUTING.md section 9 lists | A deny rule stops the usual form of a command; it is not a security boundary.[2] |
   | Each guard's hook `timeout`, in the plugin's `hooks/hooks.json` | Longer than the guard's own deadline | A hook cancelled at its timeout lets the action through, so the guard must answer first (ARCHITECTURE.md section 2).[1] |

   Managed settings take precedence over user, project and local settings.[2] If the machine's organisation sets `allowManagedHooksOnly`, the plugin's hooks run only when managed settings force-enable the plugin.[1]
4. **Has the build installed as a Claude Code plugin at user scope**, so it applies in every project and is not loaded from this repository's working tree. The plugin is listed in a marketplace whose entry pins its source to a full commit `sha`.[7] Claude Code copies that commit into its plugin cache and loads the copy.[8] Auto-update is off by default for a marketplace that is not one of Anthropic's own,[10] and it stays off, so the installed build changes only when the pin does. Where the marketplace file lives, and how the plugin's Python environment is created, are not decided. Installing or updating changes the user's Claude Code settings (`enabledPlugins`, and the saved `userConfig` values of section 1), so a maintainer does it: with what changes, the risk and the rollback stated first, a backup taken, and the settings read back afterwards.
5. **Runs the startup check in every session.** It runs on `SessionStart` and cannot stop a session; it warns in the session and records its failure code (ARCHITECTURE.md section 5.5).

## 4. Scheduled jobs

Each job is a launchd job on the machine. Loading or changing one is a maintainer's step. launchd runs a job that came due while the machine was asleep when it wakes, and skips one that came due while it was off. **[unverified]**

| Job | Runs | What it does | Its own failure |
|---|---|---|---|
| Retention | Daily | Deletes `events` rows and session chunks older than 90 days. It connects with the maintenance role, the only one that can delete (ARCHITECTURE.md section 4.3). | Exits non-zero and records its failure code; a day with no run is `job.missed` |
| Job-health check | Daily, before the report | Flags any job with no fresh success (`job.missed`) and any job that took input but produced nothing for more than one run (`job.no_output`) | Exits non-zero and records its failure code. A day it does not run records no `job.succeeded` for it, for the report to notice |
| Daily failure report | Daily | Writes the last 24 hours' failures from `events`, grouped by code, with each code's severity, as a dated Markdown file in `<report-dir>`. With nothing to report it writes a one-line all-clear that names the jobs that ran. | A missing file |
| Backup | Nightly | Dumps the `claude_mem` database with `pg_dump` to a folder outside the repository. Whether a backup already runs today was not checked. | Exits non-zero and records its failure code |

Every job runs through the job wrapper (ARCHITECTURE.md section 6.1), so a crash is recorded in `events` as `unhandled` and `job.failed` before the job exits non-zero.

**A missing report is an alarm, and a file does not raise one.** Nothing tells anyone that a report was not written, or that one lists failures. So the startup check also warns, in the session, when the newest report is more than two days old or lists a high-severity failure.

## 5. Upgrading Claude Code

A native install of Claude Code updates itself in the background, so "before upgrading" is not always available; a Homebrew install does not update itself.[11] Whether the machine pins its version is not decided. On each new version:

1. Read the [changelog](https://code.claude.com/docs/en/changelog) for changes to hooks, permissions, settings, MCP and instruction files.
2. Re-check every Claude Code fact that ARCHITECTURE.md and this document state, and update them, with the version they were checked against, in one pull request.
3. Run the full test suite, including the tests that feed each hook a recorded event and check its answer, and the test that reads a recorded transcript (ARCHITECTURE.md section 2).

The same applies to every other direct dependency: read its release notes in its official documentation, below, and re-check what the documents state about it. Versions are pinned in `pyproject.toml` and the CI workflow, not in this list.

- Claude Code: [documentation](https://code.claude.com/docs/en/overview), [changelog](https://code.claude.com/docs/en/changelog), [plugins reference](https://code.claude.com/docs/en/plugins-reference)
- Python: [documentation](https://docs.python.org/3.14/)
- PostgreSQL: [documentation](https://www.postgresql.org/docs/17/)
- pgvector: [README](https://github.com/pgvector/pgvector)
- Ollama and `qwen3-embedding`: [documentation](https://docs.ollama.com/), [model page](https://ollama.com/library/qwen3-embedding)
- GitHub Actions: [documentation](https://docs.github.com/en/actions)
- ruff: [documentation](https://docs.astral.sh/ruff/)
- pytest: [documentation](https://docs.pytest.org/)
- LongMemEval (eval data): [dataset card](https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned)

The judge, the extraction and digest models and the database driver are added to this list when each is confirmed from the inventory.

## 6. Components from outside this repository

An MCP server, a plugin or a skill from outside this repository runs with the agent's access, so each one the build depends on is listed here with its pinned version, what it can access, what it sends off the machine, and how it behaves when it fails.

| Component | Pin | Access | Sends off the machine | When it fails |
|---|---|---|---|---|
| None yet | | | | |

Other MCP servers, plugins and skills installed on the machine are not part of the build. One that adds context to a prompt or answers a permission request can change what a guard sees; the guards' tests run without them.

## 7. Applying a migration

The files in `migrations/` are applied to the `claude_mem` database in file-name order, each once, after its pull request merges. A maintainer applies each one, or approves applying it, with `psql`, as the database's owner, and reads the schema level back afterwards (ARCHITECTURE.md section 4.3). A backup is taken first. A migration creates and changes objects in the `memory` schema only; a test fails a migration that names an object outside it.

Apply a migration before updating the installed build to a commit that needs it. Rule C10 (CONTRIBUTING.md section 5.1) keeps every migration working with the build installed before it, the expand-and-contract pattern,[5] so the build can be rolled back to its previous commit without touching the store. The tests apply every file to an empty Postgres 17.

## 8. Cutover from the current pack

The current pack keeps running, with its own hooks, memory server and tables, until the new build has passed its tests and its guards have run in `log` mode. The two are never registered at once: with both, every prompt would be searched twice and every guard would fire twice.

This document assumes the current pack keeps its memory in the local `claude_mem` database. That has not been checked on the machine. It is confirmed there, with what the pack registers in the Claude Code settings, before the cutover is written out.

The cutover is one maintainer's step, written out in full before it is run:

1. Take a backup of `claude_mem` and of the Claude Code settings files.
2. Copy the current pack's facts into `memory.facts`, once. Which rows are copied, how each is given an origin and a basis, and whether they are embedded again are decided from the inventory. The current pack's tables are read and never changed.
3. Remove the current pack's hook and MCP server entries from the Claude Code settings, then install and enable the plugin (section 3).
4. Start a session and read the startup check's result.
5. To roll back, disable the plugin and restore the settings files. The `memory` schema can stay; the current pack does not read it.

## Sources

- [1] [Hooks reference (Claude Code documentation)](https://code.claude.com/docs/en/hooks)
- [2] [Configure permissions (Claude Code documentation)](https://code.claude.com/docs/en/permissions)
- [3] [CLI reference (Claude Code documentation)](https://code.claude.com/docs/en/cli-reference)
- [4] [psql: `\password` (PostgreSQL 17 documentation)](https://www.postgresql.org/docs/17/app-psql.html)
- [5] [Parallel Change, also known as expand and contract (martinfowler.com)](https://martinfowler.com/bliki/ParallelChange.html)
- [6] [Plugins reference: user configuration and environment variables (Claude Code documentation)](https://code.claude.com/docs/en/plugins-reference)
- [7] [Plugin marketplace reference: plugin sources (Claude Code documentation)](https://code.claude.com/docs/en/plugins/marketplace-reference)
- [8] [How plugins load: the cache, versions and updates (Claude Code documentation)](https://code.claude.com/docs/en/plugins/loading)
- [9] [Hooks guide (Claude Code documentation)](https://code.claude.com/docs/en/hooks-guide)
- [10] [Discover and install plugins: auto-update (Claude Code documentation)](https://code.claude.com/docs/en/discover-plugins)
- [11] [Set up Claude Code: updates (Claude Code documentation)](https://code.claude.com/docs/en/setup)
