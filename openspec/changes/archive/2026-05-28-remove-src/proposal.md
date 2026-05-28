## Why

Security scans flag the `src/` subdirectory bundled with Go installations as leaking secrets. The `src/` directory contains the Go stdlib source and is not needed at runtime for pre-compiled binaries. Removing it post-install eliminates the findings without affecting `go`, `gofmt`, or standard toolchain behavior.

## What Changes

- After extracting and moving the Go installation, delete `{{ go_install_path }}/src` to strip the stdlib source directory.
- Update `test.yml` to run `tree` on `{{ go_install_path }}` so the resulting directory structure is visible in test output.

## Capabilities

### New Capabilities

- `remove-src`: Post-install cleanup that removes the `src/` subdirectory from the versioned Go installation directory.

### Modified Capabilities

<!-- No existing spec-level behavior is changing. -->

## Impact

- `tasks/install_go.yml`: Add a task to remove `{{ go_install_path }}/src` after install.
- `test.yml`: Add a `tree` command on the versioned Go install path.
- No changes to role variables, defaults, or public interface.
