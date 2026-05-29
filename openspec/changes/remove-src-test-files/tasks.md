## 1. Update install_go.yml

- [x] 1.1 Remove the "find non-essential top-level entries" task and its corresponding "remove non-essential top-level entries" task
- [x] 1.2 Remove the "find non-tool entries in go pkg dir" task and its corresponding "remove non-tool entries from go pkg dir" task
- [x] 1.3 Add a find task that searches `{{ go_install_path }}/src/` recursively for files matching `*_test.go` and `*.pem`
- [x] 1.4 Add a loop task that deletes each file found by the find task (become: true, file state: absent)

## 2. Update specs

- [x] 2.1 Rewrite `openspec/specs/remove-src/spec.md` to reflect the new behavior (targeted file removal, full layout retained)
- [x] 2.2 Create `openspec/specs/remove-src-test-files/spec.md` with the new capability requirements

## 3. Verify

- [ ] 3.1 Run `task -t /Users/bwagner/.taskfiles/tasks/ansible_role.yml local_container` and confirm `go version` succeeds
- [ ] 3.2 Confirm no `*_test.go` or `*.pem` files exist under the install dir's `src/` in the test output
- [ ] 3.3 Confirm `src/`, `api/`, `doc/`, and other top-level dirs are present in the directory listing
