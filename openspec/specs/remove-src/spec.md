### Requirement: Strip non-bin content from Go install dir after installation
After installing Go to `{{ go_install_path }}`, the role SHALL remove all entries except `bin/` (files and directories such as `src/`, `pkg/`, `api/`, `doc/`, `lib/`, `misc/`, `test/`, and top-level files) to eliminate security scan findings and reduce image size.

#### Scenario: only bin remains after fresh install
- **WHEN** Go is installed for the first time to `{{ go_install_path }}`
- **THEN** only `{{ go_install_path }}/bin/` SHALL exist; all other top-level entries SHALL be absent

#### Scenario: removal is idempotent
- **WHEN** the role runs again and Go is already installed (skipping the download/extract block)
- **THEN** the role SHALL succeed without error even if non-bin entries are already absent

### Requirement: Test exposes post-install directory layout
The test playbook SHALL display a directory listing of all `go-*` install paths under `{{ go_root_dir }}` so that the resulting structure is visible in test output and reviewable in CI.

#### Scenario: directory layout appears in test run
- **WHEN** the test playbook executes after role application
- **THEN** the output SHALL include a listing of each `go-*` install dir confirming only `bin/` is present
