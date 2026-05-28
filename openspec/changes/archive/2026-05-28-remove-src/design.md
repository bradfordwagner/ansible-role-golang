## Context

Security scans flag the `src/` directory shipped inside every Go tarball as a secret-leaking path. When the role extracts and moves the Go tarball to `{{ go_install_path }}`, the full stdlib source tree (including embedded test fixtures and vendored data) lands on disk and triggers findings. The fix is a single post-install cleanup task.

## Goals / Non-Goals

**Goals:**
- Remove `{{ go_install_path }}/src` immediately after the Go installation step.
- Make the resulting directory structure visible in test output via `tree`.
- Eliminate security scan findings without changing the public role interface.

**Non-Goals:**
- Removing other non-essential Go subdirectories (e.g., `pkg/`, `test/`, `doc/`).
- Making the removal conditional or opt-out via a variable.
- Changes to role variables or defaults.

## Decisions

**Always remove `src/`, no toggle variable.**
A variable like `go_remove_src: true` adds complexity for a security fix that should always apply. If a caller genuinely needs the stdlib source, they can install it separately via `go env GOROOT`. Security scan findings are the forcing function; there is no valid use case for keeping `src/` in this role.

**Use `ansible.builtin.file` with `state: absent` instead of `command: rm -rf`.**
The `file` module is idempotent, avoids shell injection, and is the Ansible-idiomatic approach. It also handles the case where `src/` is missing gracefully (already absent = no change).

**Place the removal in the `always:` block of the install block.**
The `src/` directory only exists after a successful extraction. Placing removal in `always:` ensures it runs even if a later step fails, preventing a partially-installed `src/` from persisting. Since the cleanup runs under `become: true` alongside the other cleanup tasks, no privilege escalation changes are needed.

**Add `tree` to `test.yml` after install.**
Running `tree {{ go_install_path }}` in the test makes the post-removal directory layout explicit and verifiable in CI output.

## Risks / Trade-offs

- **`go build` for stdlib packages**: Pre-built stdlib `.a` files live in `pkg/`, not `src/`. Removing `src/` does not break normal builds. Tools that recompile the stdlib from source (rare, e.g., `go build -a ./...` on stdlib packages directly) will fail — this is acceptable for the container images this role targets.
- **`tree` not present on all images**: Some minimal images may not have `tree`. The test should use `command` and tolerate absence, or ensure the test image includes `tree`.
