### Requirement: Remove security-flagged file types from Go install src/
After installing Go to `{{ go_install_path }}`, the role SHALL find and delete all files matching `*_test.go` or `*.pem` recursively under `{{ go_install_path }}/src/`. All other files and directories within `src/` SHALL remain intact.

#### Scenario: test files are removed after fresh install
- **WHEN** Go is installed for the first time and the src/ tree contains `*_test.go` files
- **THEN** no `*_test.go` files SHALL exist anywhere under `{{ go_install_path }}/src/` after the role completes

#### Scenario: pem files are removed after fresh install
- **WHEN** Go is installed for the first time and the src/ tree contains `*.pem` files
- **THEN** no `*.pem` files SHALL exist anywhere under `{{ go_install_path }}/src/` after the role completes

#### Scenario: non-flagged src files are preserved
- **WHEN** Go is installed and cleanup runs
- **THEN** all files under `{{ go_install_path }}/src/` that do not match `*_test.go` or `*.pem` SHALL remain present

#### Scenario: cleanup is idempotent when files already absent
- **WHEN** the role runs again and Go is already installed with no `*_test.go` or `*.pem` files present
- **THEN** the role SHALL succeed without error
