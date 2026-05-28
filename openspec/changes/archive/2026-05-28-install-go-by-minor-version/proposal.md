## Why

Managing Go installations across machines is tedious when patch versions change frequently. This role lets users pin to a Go minor version (e.g., `1.26`) and automatically resolves and installs the latest patch release, removing the need to manually track `1.26.x` releases.

## What Changes

- Add `go_version` role variable defaulting to `1.26` (comment-documented above each variable in `defaults/main.yml`)
- Add `go_root_dir` variable defaulting to `/usr/local/go-{{ resolved_full_version }}` — the versioned installation directory (mirrors `GOROOT`); comment-documented in defaults
- Add `go_bin_dir` variable defaulting to `/usr/local/bin` — destination for symlinks to all Go binaries; comment-documented in defaults
- Fetch the latest stable patch release for the given minor version from `https://go.dev/dl/?mode=json` (stable releases only — no RCs)
- Detect target OS and CPU architecture to select the correct binary asset
- Download the matching tarball and verify its SHA-256 checksum against the value published in the Go downloads API
- Extract tarball into `go_root_dir`, then symlink all binaries from `{{ go_root_dir }}/bin/` into `go_bin_dir`

## Capabilities

### New Capabilities
- `go-version-resolve`: Queries `https://go.dev/dl/?mode=json` (stable releases only) to resolve the latest patch version matching a given `major.minor` (e.g., `1.26` → `1.26.3`) and returns the download URL, filename, and expected SHA-256 checksum for the detected OS/arch
- `go-binary-install`: Downloads the resolved tarball, verifies its SHA-256 checksum, and installs the Go toolchain on the system

### Modified Capabilities

## Impact

- New role files: `defaults/main.yml`, `tasks/main.yml`, `tasks/resolve_version.yml`, `tasks/install_go.yml`
- Depends on: `ansible.builtin.uri` (HTTP/JSON fetch), `ansible.builtin.get_url` (download with checksum), `ansible.builtin.unarchive` (tarball extraction)
- No breaking changes; this is a new role
