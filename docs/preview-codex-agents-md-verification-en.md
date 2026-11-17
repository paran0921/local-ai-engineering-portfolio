# Preview: Codex AGENTS.md Loading and Workflow-Constraint Verification

[한국어 문서 보기](preview-codex-agents-md-verification-ko.md)

- Date: 2026-11-16
- Status: Preview
- Scope: Evidence from two Codex sessions in a Laravel project
- Verification status: Automatic instruction loading and a bounded read/test workflow were observed; compliance during actual code modification remains unverified.

## Why this matters

Repository-level instructions are useful only when an agent receives them and follows the applicable constraints during real work. This preview describes what was observed in two Codex sessions on 2026-11-16. It does not claim that every instruction will always be followed.

## What was verified

### 1. Automatic loading of AGENTS.md

In an initial session, the Codex Rollout (the session's JSONL execution record) contained the project `AGENTS.md` instructions in its session context. The structured metadata identified the content as `agents_md.instructions` (the context item type for AGENTS.md instructions) with provenance `agents_md` (metadata identifying the instruction source); the session world state (`world_state` execution-state metadata) also recorded the applicable instructions.

This is evidence that the instructions were automatically supplied to that session. It is not, by itself, evidence that Codex separately opened the on-disk file or that the injected content was byte-for-byte identical to the file on disk.

### 2. Direct reading and pre-task inspection in a later session

A second session recorded both the automatically supplied instructions and explicit commands to inspect `AGENTS.md`. The session also inspected the project configuration, relevant routes, application code, views, tests, diffs, and Git status before running the approved test.

The reported inspection phase did not modify files or run tests.

### 3. Approved test execution

The approved command was:

```powershell
php artisan test --filter=PostIndexTest
```

The Rollout recorded:

| Item | Result |
|---|---:|
| Exit code | 0 |
| Tests passed | 7 |
| Tests failed | 0 |
| Tests skipped | 0 |
| Assertions | 21 |
| Duration | 16.41 seconds |

The test output included the case that verifies descending ID order when posts share the same `created_at` value.

### 4. Bounded Git-status comparison

The session compared:

```powershell
git status --short -- app tests routes resources
```

before and after the test. The recorded outputs were the same.

This supports preservation of the Git status in those four paths during the recorded session. It is not a byte-for-byte comparison of every file in the repository and does not rule out activity by an external process.

## Evidence-based assessment

| Area | Assessment |
|---|---|
| AGENTS.md automatically supplied to the session | Confirmed |
| Direct AGENTS.md read in the later session | Confirmed |
| Pre-task project inspection | Recorded |
| Approved test execution | Confirmed in the Rollout |
| Test result: 7 tests, 21 assertions | Confirmed in the recorded output |
| Git status unchanged in the four inspected paths | Confirmed in the recorded output |
| Compliance during actual code modification | Not yet verified |
| Byte-for-byte preservation of the entire repository | Not established |

## What this preview does not claim

- It does not establish that Codex will comply with every repository instruction in all future sessions.
- It does not verify instruction compliance while making source-code changes.
- It does not establish that every file in the repository remained byte-identical.
- It does not attribute any behavior to a CLI/background-service version difference; no causal effect was established.

## Next step

The next useful evaluation is a small, explicitly authorized code-change task with a defined file allowlist, success criteria, test command, and stop conditions. The resulting diff, test output, and before/after worktree state should be reviewed as separate evidence.

## Evidence records

The underlying internal verification records are retained separately from this public-facing preview:

- Codex AGENTS.md automatic-loading verification, 2026-11-16.
- Codex AGENTS.md loading and workflow-constraint verification supplement, 2026-11-16.

This preview summarizes those records; it does not publish raw session logs or local machine paths.
