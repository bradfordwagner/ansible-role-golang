## 1. Remove src after install

- [x] 1.1 In `tasks/install_go.yml`, add an `ansible.builtin.file` task with `state: absent` targeting `{{ go_install_path }}/src` in the `always:` block, after the existing tarball and temp-dir cleanup tasks — run under `become: true`

## 2. Update test

- [x] 2.1 In `test.yml`, add a task that runs `tree {{ go_install_path }}` (or `find {{ go_install_path }} -maxdepth 1`) to display the post-install directory layout in test output
