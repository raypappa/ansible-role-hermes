# CHANGELOG


## v0.2.2 (2026-09-04)

### Bug Fixes

- **ci**: Skip package build during release
  ([`18a68cd`](https://github.com/raypappa/ansible-role-hermes/commit/18a68cdb1ea71e3cc442c2a06cb19451cdcb80c2))

Disable the package build step because this repository publishes an Ansible role rather than a
  Python package.


## v0.2.1 (2026-09-04)

### Bug Fixes

- **ci**: Fetch full history for semantic release
  ([`599106a`](https://github.com/raypappa/ansible-role-hermes/commit/599106acdd711797af2600814ef9523477579dc6))

Configure the release checkout with full Git history so semantic-release can resolve parent commits
  and tags when generating the changelog.

- **ci**: Correct semantic-release configuration
  ([`83776c8`](https://github.com/raypappa/ansible-role-hermes/commit/83776c876b99fb2f62283f99e5d28221648f4cb6))

Use the supported Angular commit parser and current changelog configuration so semantic-release can
  validate the project configuration.

### Continuous Integration

- Authenticate Molecule git fetches
  ([`07a6583`](https://github.com/raypappa/ansible-role-hermes/commit/07a6583fa7f86fb67eeb0e654383a1b88086f8a2))

Pass the read-only GitHub token into Molecule jobs, retry transient Git operations, and update the
  failure artifact action.

- Update release workflow credentials
  ([`229911b`](https://github.com/raypappa/ansible-role-hermes/commit/229911b27da1a5350a9460328dfc8a8ff92c3c71))

Use the Galaxy token secret name and retain full release metadata behavior for CI publishing.


## v0.2.0 (2026-09-02)

### Bug Fixes

- Install Molecule Ansible collections
  ([`b3a9622`](https://github.com/raypappa/ansible-role-hermes/commit/b3a9622e8b15c1b59266553d42383b030fe50c9e))

Add the Ansible collections required by the Molecule scenario, including community.general for
  conditionally parsed platform tasks and community.docker for custom lifecycle
  management.\n\nRemove the role dependency requirements-file override so Molecule uses
  requirements.yml for roles and collections.yml for collections, preventing CI converge failures
  caused by unresolved modules.

- Make Molecule lifecycle limit-aware
  ([`f7d221d`](https://github.com/raypappa/ansible-role-hermes/commit/f7d221d1682732759a5c39ec6e9e3d5f2c29e103))

Run custom container creation and destruction playbooks against the selected Molecule platform host
  while delegating Docker operations to localhost.\n\nThis prevents matrix jobs using --limit from
  skipping lifecycle tasks and then failing during prepare because their containers were never
  created. Remove unused cleanup phases to avoid missing-playbook warnings.

- Move tooling dependencies to dev group
  ([`25f44b9`](https://github.com/raypappa/ansible-role-hermes/commit/25f44b93082329389622524c4ecbf4f7a2427ada))

Keep the published project free of development-only Ansible, Molecule, linting, and test
  dependencies by moving them into the dev dependency group.\n\nAdd the Docker Molecule plugin to
  the development environment so uv-based CI and local tests install the same provider explicitly,
  and refresh the lockfile.

- Use ansible facts mapping
  ([`8995184`](https://github.com/raypappa/ansible-role-hermes/commit/8995184a0a92b7d72e8807513cb847924a6b2608))

Replace deprecated top-level Ansible fact variables with entries from ansible_facts.\n\nThis
  prevents INJECT_FACTS_AS_VARS deprecation warnings and keeps OS, user, environment, service
  manager, and timestamp lookups compatible with ansible-core 2.24 and later.

- Support Docker Engine in Molecule tests
  ([`900d6a9`](https://github.com/raypappa/ansible-role-hermes/commit/900d6a9c61ef0b85f66a6ca5adc77db6e1fa38de))

Work around the Docker collection's container start request incompatibility with newer Docker Engine
  API versions by creating containers in the present state and starting them through the Docker
  CLI.\n\nPin Docker API negotiation in the mise test task to the client-supported version so the
  Molecule suite runs reliably in the current environment.

### Continuous Integration

- Test all branch pushes
  ([`21f88ec`](https://github.com/raypappa/ansible-role-hermes/commit/21f88ec26bec3bafd3905c931774b0b439f99dd9))

Run the Molecule workflow for pushes to any branch instead of restricting push events to
  main.\n\nKeep pull request validation targeted at main while allowing branch work to receive lint
  and parallel Molecule feedback before a pull request is opened.

- Automate semantic releases
  ([`dccbec1`](https://github.com/raypappa/ansible-role-hermes/commit/dccbec114c797c56298d3c0d9b812e21f606b49c))

Configure python-semantic-release to derive versions from pyproject.toml, use conventional commits,
  and create v-prefixed tags.\n\nAdd a GitHub Actions release workflow that runs on main updates or
  manual dispatch, checks out full history, and grants the release action permission to publish
  repository contents.

- Modernize dependency and validation workflows
  ([`c434bd1`](https://github.com/raypappa/ansible-role-hermes/commit/c434bd1cfd2f9777540df30f897fa9fd161ea81f))

Use uv to install the Python version and project dependencies defined by pyproject.toml in CI. Run
  the existing pre-commit checks before the parallel Molecule distro matrix, while preserving Docker
  API compatibility and tag-gated Galaxy publishing.\n\nAdd Renovate's recommended configuration to
  track project, workflow, and tooling dependencies. Keep pre-commit in the development dependency
  group and refresh the uv lockfile accordingly.

### Features

- Add Hermes Ansible role
  ([`45d95e8`](https://github.com/raypappa/ansible-role-hermes/commit/45d95e8ab0937e7b72d8c5a22d04ac0e8d74ef43))

Add the Hermes Agent role with optional mise-managed tooling, platform prerequisites, configuration,
  launchers, and gateway service support.\n\nUse the published raypappa.mise Galaxy role for
  Molecule dependency setup and expose mise-tasks/test-role so the complete Molecule suite can be
  run consistently through mise.
