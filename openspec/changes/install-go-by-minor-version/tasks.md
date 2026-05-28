## 1. Role Scaffolding

- [x] 1.1 Create `defaults/main.yml` with `go_version`, `go_root_dir` base, and `go_bin_dir`, each with an explanatory comment above it
- [x] 1.2 Create `tasks/main.yml` that includes the resolve and install task files in order

## 2. Version Resolution

- [x] 2.1 Create `tasks/resolve_version.yml` — fetch `https://go.dev/dl/?mode=json` with `ansible.builtin.uri` and register the JSON response
- [x] 2.2 Filter the response to find releases matching `^go{{ go_version }}\.\d+$`, take the first (newest), and register `go_resolved_version` (e.g., `1.26.3`)
- [x] 2.3 Fail with a descriptive error if no matching stable release is found
- [x] 2.4 Fail with a descriptive error if the resolved version string contains `rc` or `beta`
- [x] 2.5 Map `ansible_architecture` to Go arch string (e.g., `x86_64` → `amd64`, `aarch64` → `arm64`) and fail clearly if the architecture is unmapped
- [x] 2.6 Select the matching file entry from `files[]` for the target OS and arch; register filename, download URL, and SHA-256 checksum as facts

## 3. Download and Verify

- [x] 3.1 Create `tasks/install_go.yml` — set the full versioned install path fact (`{{ go_root_dir }}/go-{{ go_resolved_version }}`)
- [x] 3.2 Check if `{{ go_install_path }}/bin/go` already exists; set a skip fact to make download/extract idempotent
- [x] 3.3 Download the tarball to a temp path using `ansible.builtin.get_url` with `checksum: sha256:<hash>` — skipped if already installed

## 4. Install and Link

- [x] 4.1 Extract the tarball into the parent of `go_install_path` using `ansible.builtin.unarchive`, renaming the extracted `go/` dir to `go-{{ go_resolved_version }}` — skipped if already installed
- [x] 4.2 Symlink `{{ go_install_path }}/bin/go` into `go_bin_dir` with `force: true`
- [x] 4.3 Symlink `{{ go_install_path }}/bin/gofmt` into `go_bin_dir` with `force: true`

## 5. Role Metadata and Documentation

- [x] 5.0 Update `meta/main.yml` with correct `role_name`, description, and galaxy tags; update `README.md` with Galaxy link and usage example



- [x] 5.1 Update `config.yaml`: set `target_repo: ghcr.io/bradfordwagner/ansible-role-golang`, update `upstream.tag` to `6.3.1`, and replace build matrix with: `archlinux_latest` (amd64), `debian_bookworm`, `debian_bookworm-slim`, `debian_trixie`, `debian_trixie-slim`, `ubuntu_noble`, `ubuntu_plucky` (amd64 + arm64 each)

## 6. Test Playbook

- [x] 6.1 Update `test.yml` to apply the role twice: first with `go_version: "1.26"`, then with `go_version: "1.25"` (last install wins the symlinks)
- [x] 6.2 Add a task in `test.yml` after both role invocations that runs `go version` (no full path — must resolve via PATH) and registers the output
- [x] 6.3 Add an assert task that verifies the output contains `go1.25` (confirming the last-installed version is the linked one)

## 7. Verification

- [x] 7.1 Run `task -t /Users/bwagner/.taskfiles/tasks/ansible_role.yml local_container` and confirm `go version` outputs a `go1.25.x` version string
