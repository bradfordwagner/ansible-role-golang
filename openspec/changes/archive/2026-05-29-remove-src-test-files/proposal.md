## Why

Security scans failed specifically because of `_test.go` files inside the Go standard library `src/` tree. The previous approach of removing entire top-level directories (`src/`, `api/`, `doc/`, etc.) was overly aggressive. Targeting only `_test.go` files satisfies the scanner while preserving the full Go install layout.

## What Changes

- The top-level find/remove block is removed entirely — no top-level directories are deleted
- The `pkg/` pruning block is also removed — `pkg/` is kept as-is
- After installation, all `_test.go` files under `{{ go_install_path }}/src/` are found and deleted recursively
- After installation, all `*.pem` files under `{{ go_install_path }}/src/` are found and deleted recursively
- All other Go install content (`api/`, `doc/`, `lib/`, `misc/`, `test/`, `src/`, `pkg/`) is retained

## Capabilities

### New Capabilities

- `remove-src-test-files`: Find and delete all `_test.go` and `*.pem` files recursively under the Go install `src/` directory after installation

### Modified Capabilities

- `remove-src`: The requirement changes substantially — instead of stripping top-level dirs and `pkg/`, the role now only removes `_test.go` files from `src/`; the retained/removed inventory and all scenarios must be updated

## Impact

- `tasks/install_go.yml`: remove the top-level find/remove block and the `pkg/` find/remove block; add find-and-delete steps targeting `*_test.go` and `*.pem` files under `{{ go_install_path }}/src/`
- `openspec/specs/remove-src/spec.md`: rewrite retained/removed inventory and scenarios to match the new minimal cleanup approach
- `test.yml`: directory listing assertion must accept the full Go install layout
