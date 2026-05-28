## ADDED Requirements

### Requirement: Remove src directory after Go installation
After installing Go to `{{ go_install_path }}`, the role SHALL remove the `src/` subdirectory to eliminate security scan findings caused by secrets detected in the stdlib source tree.

#### Scenario: src removed after fresh install
- **WHEN** Go is installed for the first time to `{{ go_install_path }}`
- **THEN** `{{ go_install_path }}/src` SHALL NOT exist on disk after the role completes

#### Scenario: removal is idempotent
- **WHEN** the role runs again and Go is already installed (skipping the download/extract block)
- **THEN** the role SHALL succeed without error even if `{{ go_install_path }}/src` is already absent

### Requirement: Test exposes post-install directory layout
The test playbook SHALL run `tree` on `{{ go_install_path }}` so that the resulting directory structure is visible in test output and reviewable in CI.

#### Scenario: tree output appears in test run
- **WHEN** the test playbook executes after role application
- **THEN** the output SHALL include a directory listing of `{{ go_install_path }}` showing `bin/` and confirming `src/` is absent
