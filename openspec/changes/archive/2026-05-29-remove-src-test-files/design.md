## Context

The role currently strips most of the Go install directory after extraction — everything except `bin/` and `pkg/tool/` is deleted — to silence security scanner findings. Investigation narrowed the actual findings to two file types inside `src/`: test files (`*_test.go`) and certificate files (`*.pem`). The broad removal was unnecessary and discards content that may be useful (stdlib source for IDEs, docs, etc.).

## Goals / Non-Goals

**Goals:**
- Remove all `*_test.go` files recursively from `{{ go_install_path }}/src/`
- Remove all `*.pem` files recursively from `{{ go_install_path }}/src/`
- Eliminate the existing broad top-level and `pkg/` directory pruning blocks
- Keep the change idempotent (safe to re-run when files are already absent)

**Non-Goals:**
- Scanning or cleaning any directory other than `src/`
- Removing other file types that have not been confirmed as scanner findings
- Changing version resolution, download, or symlink behavior

## Decisions

**Use `ansible.builtin.find` + loop instead of a shell glob**

`find` with `patterns` is idempotent and returns an empty list cleanly when no files match, avoiding errors on a second run. A shell glob would require `ignore_errors` or `failed_when` workarounds.

**Two separate find tasks (one per pattern) rather than one combined task**

`ansible.builtin.find` accepts a list for `patterns`, so both patterns can be handled in a single task. This is cleaner than two tasks.

**Remove the existing top-level and `pkg/` pruning blocks entirely**

They are no longer needed. Leaving dead cleanup tasks would cause confusion and impose unnecessary filesystem mutations on every install.

## Risks / Trade-offs

- [New scanner findings] If additional file types in `src/` are flagged in future scans, another targeted pattern can be added to the same find task. → Low risk; the pattern is easy to extend.
- [find on large src/ tree] The stdlib `src/` tree is large; find traversal adds a small amount of time to every install run. → Acceptable; `changed_when` and `when: not already installed` guards exist at the block level.

## Migration Plan

No migration needed — this is a tightening of cleanup scope. Hosts that already have Go installed with the old broad cleanup will have a minimal install dir; the new find task will simply find no files to delete and succeed with no changes.
