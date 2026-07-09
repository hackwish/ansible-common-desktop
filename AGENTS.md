# AGENTS.md — ansible-common-desktop

## Project type
Standard Ansible role (not a collection). Layout follows the scaffold from `ansible-galaxy init`. Requires `community.general` collection and Ansible ≥ 2.14.

## Quick reference
| Command | Action |
|---|---|
| `ansible-playbook tests/test.yml` | Run role locally (requires root) |
| Conventional commits (feat/fix/…) | Required; `main` + `master` trigger semantic-release |

## Architecture
- **Entrypoint:** `tasks/main.yml` dispatches by `ansible_os_family`:
  - **Debian** → `repos.yml` (PPAs/APT repos) → `packages.yml` (APT + Snap + Flatpak + .deb curls) → `aditional-packages.yml`
  - **RedHat** → `packages-fedora.yml` (DNF + Snap + Flatpak + Chrome repo)
  - **Darwin** → `packages-mac.yml` (Homebrew + Homebrew Cask)
  - All OSes → `upgrade.yml`
- **Three package managers on Linux:** APT (bulk), Snap, Flatpak via `community.general.flatpak`
- **Single var:** `codename` read from env `DISTRIB_CODENAME` (parameterizes PPA repos — Debian only)
- **Target OS:** Ubuntu / Linux Mint / Fedora / macOS

## Known quirks
- File `tasks/aditional-packages.yml` — typo in filename (kept as-is, do not rename)
- Typo in `tests/test.yml`: role referenced as `ansible-commom-desktop` (double m)
- Sync-conflict `*.sync-conflict-*` files committed in `tasks/` — cleanup pending, ignore pattern in `.gitignore`
- Templates `templates/20auto-upgrades.j2`, `jail.local.j2`, `sshd_config.j2` exist but **no task deploys them** (orphaned)
- `files/` is empty
- Task names mix Spanish and English
- Idempotency not guaranteed for shell-based tasks (`apt-get clean`, `apt-get update`, curl pipe-to-shell for Lutris)
- No lint, no pre-commit, no molecule, no test framework — only manual `ansible-playbook` testing

## OS-specific notes
- **Debian (Ubuntu/Mint):** PPAs require `DISTRIB_CODENAME` env var. Kernel headers use `linux-headers-{{ ansible_kernel }}`.
- **Fedora:** Google Chrome repo added via `yum_repository`. Snap needs `snapd` service enabled. RPM Fusion recommended for `rtl8812au-dkms`.
- **macOS:** Requires Homebrew + `community.general` collection. Casks install GUI apps. Some Linux-specific packages silently skipped.

## CI
- GitHub Actions `.github/workflows/actions.yml`: semantic-release only (no test/lint jobs)
- Requires `GITHUB_TOKEN` secret for publishing
