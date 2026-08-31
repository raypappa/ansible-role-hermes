# AGENTS.md — ansible-role-hermes

## What This Is

Ansible role that installs [Hermes Agent](https://github.com/NousResearch/hermes-agent) with optional mise-based tool management. Targets Ubuntu, Debian, Fedora, CentOS/Rocky/Alma, Arch, macOS.

## Key Commands

```bash
# Lint (run before committing)
yamllint .
ansible-lint

# Molecule tests (requires Docker)
pip install ansible-core molecule molecule-plugins[docker]
molecule test
molecule test -- --limit ubuntu-noble   # single distro

# Pre-commit
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

## Architecture

```
tasks/main.yml          → entry point, includes others conditionally
tasks/user.yml          → creates hermes system user
tasks/mise_check.yml    → validates mise, installs tools via mise_use module
tasks/prerequisites.yml → build tools, ffmpeg, Playwright deps (root only)
tasks/clone.yml         → git clone + release tag checkout
tasks/venv.yml          → Python venv via uv
tasks/node_deps.yml     → npm install + Playwright
tasks/launcher.yml      → hermes/hermes-agent/hermes-acp scripts in ~/.local/bin
tasks/config.yml        → .env, config.yaml, SOUL.md, skills
tasks/gateway.yml       → systemd service (hardened, runs as hermes user)
```

## Conventions

- **Mise is optional** (`hermes_use_mise: false` default). When enabled, uses `mise_use` module from `ansible-role-mise` library — not inline `mise use` commands.
- **`_hermes_mise_exec`** in `vars/main.yml` is empty when mise is disabled, `mise exec --` when enabled. Use this prefix in tasks instead of duplicating mise/system variants.
- **`become_user`** for tool tasks: use `_hermes_mise_user` (hermes user when creating, current user otherwise).
- **Two systemd unit templates**: one for mise (`mise exec -- python ...`), one for system (`venv/bin/python ...`). ExecStart differs fundamentally, so separate tasks are correct.
- **Internal variables** are prefixed with `_hermes_` (computed in vars/main.yml, not overridable).
- **Tags** follow `hermes_<task>` pattern: `hermes_user`, `hermes_mise`, `hermes_clone`, etc.

## Testing Notes

- Molecule converge runs twice: once without mise, once with mise.
- `molecule/default/requirements.yml` installs the published `raypappa.mise`
  role from Ansible Galaxy.
- Converge must reference the FQRN `raypappa.hermes`, not `hermes` — molecule's
  prerun links the local role under its fully qualified name.
- `molecule/default/molecule.yml` adds the Galaxy-installed mise role's
  `library/` to `defaults.library` so the `mise_use` module resolves.
- Prepare installs mise to `/usr/local/bin` in containers — the role itself
  never installs mise (by design).
- `idempotence` is omitted from test_sequence: uv/npm/mise shell tasks cannot
  report reliable change detection on rerun.
- `hermes_skip_browser: true` in converge skips the ~170MB Chromium download;
  without it tests take hours on slow networks.
- Verify checks: user exists, home dir, agent dir, launcher executable, venv,
  python in venv.

## Publishing

- Galaxy workflow triggers on `v*.*.*` tags
- Requires `GALAXY_API_KEY` secret in GitHub repo settings
- Role name on Galaxy: `hermes`
