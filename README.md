# bradfordwagner.golang

Ansible role that installs Go by minor version, automatically resolving and installing the latest stable patch release with SHA-256 checksum verification.

[![Ansible Galaxy](https://img.shields.io/badge/galaxy-bradfordwagner.golang-blue)](https://galaxy.ansible.com/ui/standalone/roles/bradfordwagner/golang/)

## Usage

```yaml
- hosts: all
  roles:
    - role: bradfordwagner.golang
      vars:
        go_version: "1.26"
```

## Variables

| Variable | Default | Description |
|---|---|---|
| `go_version` | `"1.26"` | Go major.minor version line to install |
| `go_root_dir` | `/usr/local` | Base directory for versioned Go installations |
| `go_bin_dir` | `/usr/local/bin` | Directory where `go` and `gofmt` are symlinked |
