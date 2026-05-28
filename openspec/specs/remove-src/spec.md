### Requirement: Strip non-essential content from Go install dir after installation
After installing Go to `{{ go_install_path }}`, the role SHALL remove all top-level entries except `bin/` and `pkg/`, then remove all entries inside `pkg/` except `pkg/tool/`. This eliminates security scan findings (from `src/` and other source/doc trees) while preserving the toolchain binaries required for GOROOT detection and compilation.

Retained: `bin/` (go, gofmt), `pkg/tool/` (compile, link, asm, etc.)
Removed: `src/`, `api/`, `doc/`, `lib/`, `misc/`, `test/`, top-level files, `pkg/{os}_{arch}/*.a` precompiled stdlib

#### Scenario: only bin and pkg/tool remain after fresh install
- **WHEN** Go is installed for the first time to `{{ go_install_path }}`
- **THEN** only `{{ go_install_path }}/bin/` and `{{ go_install_path }}/pkg/tool/` SHALL exist; all other entries SHALL be absent

#### Scenario: go binary resolves GOROOT after cleanup
- **WHEN** `go version` is run after cleanup
- **THEN** the command SHALL succeed because `pkg/tool/` is present for GOROOT detection

#### Scenario: removal is idempotent
- **WHEN** the role runs again and Go is already installed (skipping the download/extract block)
- **THEN** the role SHALL succeed without error even if non-essential entries are already absent

### Requirement: Test exposes post-install directory layout
The test playbook SHALL display a directory listing of all `go-*` install paths under `{{ go_root_dir }}` so that the resulting structure is visible in test output and reviewable in CI.

#### Scenario: directory layout appears in test run
- **WHEN** the test playbook executes after role application
- **THEN** the output SHALL include a listing of each `go-*` install dir confirming only `bin/` is present
