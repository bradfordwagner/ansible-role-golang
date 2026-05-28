## Context

The `bradfordwagner.ansible-role-golang` role (https://github.com/bradfordwagner/ansible-role-golang.git) needs to install Go on target hosts. Go releases follow `major.minor.patch` versioning; operators typically care about staying on a minor version line (e.g., `1.26`) and want the latest patch automatically. The official Go download index at `https://go.dev/dl/?mode=json` (stable releases only — no `include=all`) provides a machine-readable list of all official releases with per-file checksums, download URLs, OS, and architecture — eliminating the need for scraping or hardcoded version pinning.

## Goals / Non-Goals

**Goals:**
- Accept a `go_version` (major.minor) variable and resolve it to the latest `major.minor.patch`
- Select the correct tarball for the target host's OS and CPU architecture
- Verify the downloaded tarball's SHA-256 checksum before extraction
- Install Go into a versioned directory (`go_root_dir`) and symlink binaries into `go_bin_dir`
- Each variable has an explanatory comment above it in `defaults/main.yml`

**Non-Goals:**
- Managing `GOPATH`, `GOENV`, or user-level Go toolchains
- Installing multiple Go versions simultaneously
- Building Go from source
- Managing Go modules or project dependencies

## Decisions

### 1. Use `https://go.dev/dl/?mode=json` as the version source
Omitting `include=all` returns **only stable, official releases** — no release candidates or betas. The API returns releases newest-first, each with `version` (e.g., `go1.26.3`), `stable: true`, and a `files[]` array containing `os`, `arch`, `sha256`, and `filename` per asset.

Version resolution: filter for entries where `version` matches `^go<major.minor>\.\d+$` (e.g., `^go1\.26\.\d+$`) — the trailing `\.` prevents `go1.261.x` from matching `go1.26`. Take the first result (newest stable patch for that minor line).

**Alternative considered:** Use `include=all` and filter by `stable: true`. Rejected — unnecessary data; the default endpoint already excludes RCs.

**Alternative considered:** Hardcode the full version in the variable. Rejected — defeats the purpose of minor-version pinning.

### 2. Resolve OS/arch via Ansible facts
Use `ansible_system | lower` for OS (maps to `linux`, `darwin`) and a lookup table for `ansible_architecture` → Go arch string (`x86_64` → `amd64`, `aarch64` → `arm64`, etc.). This avoids shell-specific commands and works across all Ansible connection types.

### 3. Checksum verification via `ansible.builtin.get_url` `checksum` parameter
`get_url` supports `sha256:<hash>` natively — it downloads and verifies atomically, failing the task if the checksum does not match. No separate `stat`/`sha256sum` step needed.

**Alternative considered:** Download with `uri` then verify with `stat`. Rejected — more tasks, more failure modes.

### 4. Install into versioned directory, symlink binaries
Installing to `/usr/local/go-1.26.3` (rather than a fixed `/usr/local/go`) allows multiple minor versions to coexist on the same host without collision. Symlinking `{{ go_root_dir }}/bin/*` into `go_bin_dir` provides a stable PATH entry regardless of the resolved version.

### 5. `go_root_dir` default is computed from the resolved full version
The default in `defaults/main.yml` is set to a base path (`/usr/local`) and the full versioned path (`/usr/local/go-{{ go_resolved_version }}`) is assembled in the task after resolution. This keeps `defaults/main.yml` declarative while still supporting override.

## Risks / Trade-offs

- **go.dev API unavailability** → Tasks will fail on hosts without internet access or if go.dev is down. Mitigation: document the network requirement; consider a future `go_download_mirror` variable.
- **Architecture mapping gaps** → Less common architectures (e.g., `mips`, `s390x`) may need additions to the mapping table. Mitigation: fail fast with a clear error if the arch is unmapped.
- **Symlink conflicts** → If another package owns a `go` binary in `go_bin_dir`, the symlink task will fail or silently overwrite. Mitigation: use `force: true` on symlinks and document that this role takes ownership of Go binaries in `go_bin_dir`.
- **Re-running idempotency** → If the resolved version hasn't changed, re-running should be a no-op. Mitigation: check for existence of `{{ go_root_dir }}/bin/go` before downloading; skip if present.
