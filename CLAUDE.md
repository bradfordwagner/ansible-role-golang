# bradfordwagner.ansible-role-golang

## Testing

Test changes using a local container:

```
task -t /Users/bwagner/.taskfiles/tasks/ansible_role.yml local_container
```

## Variables

Whenever role variables are added, removed, or changed in `defaults/main.yml`, update the variables table in `README.md` with the correct name, default value, and description.

## Renovate

Renovate (`bradfordwagner/renovate`, preset `ansible-role`) opens PRs for the version pins marked `# renovate:`, `meta/requirements.yml`, `config.yaml`'s `upstream.tag` and GitHub Actions. A merged PR labeled `release` is tagged with the next patch version and published to Galaxy by `.github/workflows/release.yml`.
`test.yml`'s `go_latest` must match `go_version`; Renovate bumps both in one PR.
