## Requirements

### Requirement: Resolve latest stable patch version for a given minor version
The role SHALL query `https://go.dev/dl/?mode=json` and resolve the latest stable patch release whose version matches the prefix `go<go_version>.` (e.g., `go1.26.`). The resolved full version (e.g., `1.26.3`) SHALL be registered as a fact for use in subsequent tasks.

#### Scenario: Latest patch resolved for a valid minor version
- **WHEN** `go_version` is set to `1.26` and stable releases exist for `go1.26.x`
- **THEN** the role resolves the highest-numbered stable patch (e.g., `go1.26.3`) and registers it as `go_resolved_version`

#### Scenario: Correct minor version prefix matching
- **WHEN** `go_version` is `1.26` and the release list contains both `go1.26.3` and `go1.261.0`
- **THEN** only releases matching `go1.26.\d+` are considered, and `go1.261.0` is excluded

#### Scenario: No matching stable release found
- **WHEN** `go_version` is set to a value with no matching stable release (e.g., `1.99`)
- **THEN** the role fails with a descriptive error message

### Requirement: Exclude release candidates from version resolution
The role SHALL use `https://go.dev/dl/?mode=json` without `include=all` so that only official stable releases are returned. If a resolved version string contains `rc` or `beta`, the role SHALL fail immediately with an error message stating that release candidates and beta versions are not supported.

#### Scenario: RC version encountered after resolution
- **WHEN** the resolved version string contains `rc` (e.g., `go1.26rc1`)
- **THEN** the role SHALL fail with an error message: "Resolved version <version> is a release candidate. RC versions are not supported. Use a stable minor version."

#### Scenario: Beta version encountered after resolution
- **WHEN** the resolved version string contains `beta` (e.g., `go1.26beta1`)
- **THEN** the role SHALL fail with an error message: "Resolved version <version> is a beta release. Beta versions are not supported. Use a stable minor version."

### Requirement: Select OS and architecture-appropriate release asset
The role SHALL select the correct tarball from the resolved release's `files[]` array based on the target host's OS (`ansible_system | lower`) and CPU architecture (mapped from `ansible_architecture` to Go arch strings).

#### Scenario: Linux amd64 asset selected
- **WHEN** the target host is Linux with `ansible_architecture = x86_64`
- **THEN** the file entry with `os: linux` and `arch: amd64` is selected

#### Scenario: macOS arm64 asset selected
- **WHEN** the target host is macOS with `ansible_architecture = arm64`
- **THEN** the file entry with `os: darwin` and `arch: arm64` is selected

#### Scenario: Unsupported architecture
- **WHEN** the target host has an architecture not present in the mapping table
- **THEN** the role fails with a clear error identifying the unmapped architecture
