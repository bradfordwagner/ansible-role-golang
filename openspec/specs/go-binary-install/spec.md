## Requirements

### Requirement: Role variables defined with comments in defaults
The role SHALL define the following variables in `defaults/main.yml`, each preceded by a comment explaining its purpose:
- `go_version`: major.minor version to install (default: `"1.26"`)
- `go_root_dir`: base install path; the full versioned directory is `{{ go_root_dir }}/go-{{ go_resolved_version }}` (default base: `/usr/local`)
- `go_bin_dir`: directory where Go binaries are symlinked (default: `/usr/local/bin`)
- `go_arch_map`: dict mapping `ansible_architecture` values to Go arch strings (e.g. `x86_64: amd64`, `aarch64: arm64`); used during version resolution to select the correct release asset

#### Scenario: Defaults applied when no variables overridden
- **WHEN** the role runs with no variable overrides
- **THEN** `go_version` is `"1.26"`, install base is `/usr/local`, and `go_bin_dir` is `/usr/local/bin`

#### Scenario: Variables overridden by caller
- **WHEN** a playbook sets `go_version: "1.25"` and `go_bin_dir: "/home/user/bin"`
- **THEN** the role resolves the latest `1.25.x` patch and symlinks binaries into `/home/user/bin`

### Requirement: Download tarball and verify SHA-256 checksum
The role SHALL download the resolved Go tarball using the URL from the downloads API and verify its SHA-256 checksum (also from the API) before extraction. If the checksum does not match, the role SHALL fail and remove the downloaded file.

#### Scenario: Checksum matches
- **WHEN** the downloaded tarball's SHA-256 matches the value from the API response
- **THEN** the role proceeds to extraction

#### Scenario: Checksum mismatch
- **WHEN** the downloaded tarball's SHA-256 does not match the value from the API response
- **THEN** the role fails with an error and does not extract the tarball

### Requirement: Install Go into versioned directory
The role SHALL extract the verified tarball into `{{ go_root_dir }}/go-{{ go_resolved_version }}` (e.g., `/usr/local/go-1.26.3`). After installation, all content except `bin/` and `pkg/tool/` SHALL be removed (see `remove-src` capability).

#### Scenario: Go installed into versioned path
- **WHEN** the tarball is extracted and post-install cleanup completes
- **THEN** `{{ go_root_dir }}/go-{{ go_resolved_version }}/bin/go` exists and is executable, only `bin/` and `pkg/tool/` remain in the versioned directory

### Requirement: Symlink Go binaries into go_bin_dir
The Go distribution provides exactly two binaries in `bin/`: `go` and `gofmt`. The role SHALL create symlinks for both in `go_bin_dir`, pointing into the versioned install directory. Existing symlinks SHALL be replaced (`force: true`).

#### Scenario: Binaries symlinked after install
- **WHEN** installation completes
- **THEN** `{{ go_bin_dir }}/go` and `{{ go_bin_dir }}/gofmt` are symlinks pointing into the versioned install directory

#### Scenario: Existing symlink replaced on re-run
- **WHEN** the role runs again after a version upgrade (e.g., `1.26.2` → `1.26.3`)
- **THEN** existing symlinks in `go_bin_dir` are updated to point to the new versioned directory

### Requirement: Test playbook installs multiple versions and verifies via PATH
`test.yml` SHALL invoke the role twice — first with `go_version: "1.26"`, then with `go_version: "1.25"`. After both invocations it SHALL run `go version` without a full path (relying on PATH resolution) and assert the output contains `go1.25`, confirming the last-installed version owns the symlinks.

#### Scenario: Last installed version is the active linked binary
- **WHEN** the role is applied for `1.26` then `1.25` in the same play
- **THEN** `go version` invoked via PATH returns a `go1.25.x` version string

#### Scenario: go binary accessible without full path
- **WHEN** `go version` is run without specifying `/usr/local/bin/go`
- **THEN** the command succeeds, confirming `go_bin_dir` is on PATH

### Requirement: README variables table kept in sync with defaults
`README.md` SHALL contain a variables table that matches `defaults/main.yml`. Whenever a variable is added, removed, or changed, the README table SHALL be updated with the correct name, default value, and description.

#### Scenario: New variable added to defaults
- **WHEN** a new variable is added to `defaults/main.yml`
- **THEN** a corresponding row is added to the variables table in `README.md`

### Requirement: Role metadata and README are accurate
`meta/main.yml` SHALL have `role_name: golang`, a meaningful description, and relevant galaxy tags. `README.md` SHALL include a link to the Ansible Galaxy role page and a usage example.

#### Scenario: Galaxy metadata is accurate
- **WHEN** the role is published to Ansible Galaxy
- **THEN** the role page shows the correct name, description, and tags (not template placeholder text)

### Requirement: config.yaml matches reference build matrix
`config.yaml` SHALL be updated to match the reference from `ansible-role-qq`: build matrix includes `archlinux_latest` (amd64 only), `debian_bookworm`, `debian_bookworm-slim`, `debian_trixie`, `debian_trixie-slim`, `ubuntu_noble`, `ubuntu_plucky` (amd64 + arm64 each). `upstream.tag` SHALL be `6.3.1`. `target_repo` SHALL be `ghcr.io/bradfordwagner/ansible-role-golang`.

#### Scenario: config.yaml reflects correct build targets
- **WHEN** the role is built in CI
- **THEN** it is tested against the distros and architectures defined in the reference config, not the stale alpine/rockylinux entries

### Requirement: Idempotent installation
The role SHALL skip download and extraction if `{{ go_root_dir }}/go-{{ go_resolved_version }}/bin/go` already exists.

#### Scenario: No re-download on re-run with same version
- **WHEN** the resolved version is already installed
- **THEN** the download and extraction tasks are skipped and the role completes without changes
