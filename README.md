# Ansible Role: Hermes Agent

Installs [Hermes Agent](https://github.com/NousResearch/hermes-agent) with optional mise-based tool management.

## Requirements

- Ansible 2.12 or higher
- Target systems: Ubuntu, Debian, Fedora, CentOS, Rocky, Alma, Arch, macOS
- Root access (for system prerequisites and systemd service)
- `mise` installed on the target (only if `hermes_use_mise: true`)

## Role Variables

### Required

None — all variables have sensible defaults.

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `hermes_use_mise` | `false` | Use mise for tool management (requires mise on target) |
| `hermes_create_user` | `true` | Create a dedicated `hermes` system user |
| `hermes_user` | `hermes` | Username for the hermes service account |
| `hermes_user_home` | `/home/hermes` | Home directory for the hermes user |
| `hermes_release_tag` | `v2026.8.19` | Pinned release tag to install |
| `hermes_branch` | `main` | Git branch to clone |
| `hermes_repo_url` | `https://github.com/NousResearch/hermes-agent.git` | Repository URL |
| `hermes_install_gateway` | `true` | Install systemd gateway service |
| `hermes_gateway_autostart` | `true` | Enable gateway service at boot |
| `hermes_install_build_tools` | `true` | Install build dependencies (root) |
| `hermes_install_ffmpeg` | `true` | Install ffmpeg (root) |
| `hermes_install_playwright_deps` | `true` | Install Chromium system deps (root) |
| `hermes_python_version` | `3.11` | Python version (mise only) |
| `hermes_node_version` | `26` | Node.js version (mise only) |
| `hermes_uv_version` | `latest` | uv version (mise only) |

## Dependencies

None by default. When `hermes_use_mise: true`, the role requires:

- `mise` installed on the target system
- `ansible-role-mise` library in the module search path (for the `mise_use` module)

Add to `ansible.cfg`:

```ini
[defaults]
library = /path/to/ansible-role-mise/library
```

## Example Playbook

### Basic usage (no mise)

```yaml
- hosts: servers
  roles:
    - role: ansible-role-hermes
```

### With mise tool management

```yaml
- hosts: servers
  roles:
    - role: ansible-role-hermes
      vars:
        hermes_use_mise: true
        hermes_python_version: "3.12"
        hermes_node_version: "22"
```

### Custom release tag

```yaml
- hosts: servers
  roles:
    - role: ansible-role-hermes
      vars:
        hermes_release_tag: "v2026.8.19"
        hermes_use_mise: true
```

### Without dedicated user (run as current user)

```yaml
- hosts: servers
  roles:
    - role: ansible-role-hermes
      vars:
        hermes_create_user: false
        hermes_use_mise: true
```

## What This Role Does

1. **Creates a dedicated system user** (`hermes`) with optional lingering
2. **Installs system prerequisites** (build tools, ffmpeg, Playwright deps) as root
3. **Clones the hermes-agent repository** and checks out the pinned release tag
4. **Installs Python/Node.js/uv** via mise (when enabled) or expects them on PATH
5. **Creates a Python venv** and installs dependencies via uv
6. **Installs Node.js dependencies** for browser tools and TUI
7. **Creates launcher scripts** (`hermes`, `hermes-agent`, `hermes-acp`) in `~/.local/bin`
8. **Configures the systemd gateway service** (when root and enabled)

## Mise Integration

When `hermes_use_mise: true`:

- The role checks that `mise` is available on the target user's PATH
- Uses the `mise_use` module to install tools locally in the clone directory
- Creates a `.mise.toml` in the installation directory with pinned tool versions
- Launcher scripts and systemd service use `mise exec --` for dynamic tool resolution

When `hermes_use_mise: false`:

- No mise interaction occurs
- Tools (`uv`, `python`, `node`, `npm`) must already be on the target user's PATH
- Launcher scripts use the venv's Python directly

## Systemd Service

The role installs a hardened systemd service at `/etc/systemd/system/hermes-gateway.service`:

- Runs as the `hermes` user
- Starts after `network-online.target`
- Automatically restarts on failure
- Hardened with security directives (NoNewPrivileges, ProtectSystem, etc.)

## Development

### Running Molecule Tests

```bash
# Install dependencies
pip install ansible-core molecule molecule-plugins[docker]

# Run tests
molecule test

# Run on specific platform
molecule test -- --limit ubuntu-noble
```

### Pre-commit Hooks

```bash
# Install pre-commit
pip install pre-commit

# Install hooks
pre-commit install

# Run manually
pre-commit run --all-files
```

## License

MIT

## Author

Nous Research
