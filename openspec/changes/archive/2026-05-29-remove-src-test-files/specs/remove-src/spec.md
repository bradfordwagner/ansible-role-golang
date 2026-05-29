## MODIFIED Requirements

### Requirement: Strip security-flagged files from Go install dir after installation
After installing Go to `{{ go_install_path }}`, the role SHALL remove all files matching `*_test.go` or `*.pem` recursively under `{{ go_install_path }}/src/`. No top-level directories SHALL be removed; no entries inside `pkg/` SHALL be removed. The full Go install layout is preserved except for the targeted file types.

Retained: all top-level entries (`bin/`, `pkg/`, `src/`, `api/`, `doc/`, `lib/`, `misc/`, `test/`, etc.), all `pkg/` contents
Removed: `*_test.go` files under `src/`, `*.pem` files under `src/`

#### Scenario: full install layout is present after cleanup
- **WHEN** Go is installed for the first time to `{{ go_install_path }}`
- **THEN** `{{ go_install_path }}/bin/`, `{{ go_install_path }}/pkg/`, and `{{ go_install_path }}/src/` SHALL all exist after the role completes

#### Scenario: no test or pem files remain under src after install
- **WHEN** Go is installed and the cleanup step runs
- **THEN** no files matching `*_test.go` or `*.pem` SHALL exist anywhere under `{{ go_install_path }}/src/`

#### Scenario: go binary resolves GOROOT after cleanup
- **WHEN** `go version` is run after cleanup
- **THEN** the command SHALL succeed

#### Scenario: removal is idempotent
- **WHEN** the role runs again and Go is already installed
- **THEN** the role SHALL succeed without error even if no `*_test.go` or `*.pem` files are present
