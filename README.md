# ansible-common-desktop

Ansible role for setting up common desktop packages across multiple operating systems.

## Supported OS

| OS Family | Distributions |
|---|---|
| Debian | Ubuntu (focal, jammy, noble), Linux Mint (vanessa, vera, victoria) |
| RedHat | Fedora (all) |
| Darwin | macOS (all) |

## Requirements

- Ansible ≥ 2.14
- `community.general` collection (for Homebrew, Snap, Flatpak modules)
- **macOS:** Homebrew must be installed
- **Debian:** `DISTRIB_CODENAME` env var must be set (e.g., `jammy`, `noble`) for PPA repos

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `codename` | `lookup('env','DISTRIB_CODENAME')` | Ubuntu codename for PPA repos (Debian only) |

## Tags

| Tag | Description |
|---|---|
| `repos` | APT repository setup (Debian only) |
| `packages` | Package installation across all OSes |
| `aditional-packages` | Linux Mint specific packages |
| `upgrade` | System upgrade |
| `common` | Common package installation subtag |

## Example Playbook

```yaml
- hosts: localhost
  connection: local
  roles:
    - ansible-common-desktop
```

## Architecture

`tasks/main.yml` dispatches by `ansible_os_family`:

- **Debian** → `repos.yml` (PPAs/APT repos) → `packages.yml` (APT + Snap + Flatpak + .deb curls) → `aditional-packages.yml`
- **RedHat** → `packages-fedora.yml` (DNF + Snap + Flatpak + Chrome repo)
- **Darwin** → `packages-mac.yml` (Homebrew + Homebrew Cask)
- **All** → `upgrade.yml`

## Caveats

- Some shell-based tasks (Lutris curl install, `apt-get clean`, `apt-get update`) are not idempotent
- Task names mix Spanish and English
- `rtl8812au-dkms` may fail on newer kernels — use with `ignore_errors`
- Templates in `templates/` (`20auto-upgrades.j2`, `jail.local.j2`, `sshd_config.j2`) exist but are not deployed by any task

## License

BSD
