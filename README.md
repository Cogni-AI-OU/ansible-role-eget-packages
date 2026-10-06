# Ansible Role: Template

[![PR Reviews][pr-reviews-image]][pr-reviews-link]
[![License][license-image]][license-link]
[![Check][check-image]][check-link]

This role installs binaries from a configurable list of package names using predefined eget commands.

## Getting Started

🗺️ **New to this repository?** Take the [**CodeTour**](.tours/getting-started.tour) to explore the project structure!

To view the tour:

1. Install the [CodeTour extension](https://marketplace.visualstudio.com/items?itemName=vsls-contrib.codetour) in VS Code
2. Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`)
3. Run "CodeTour: Start Tour"

Or simply open this repository in a devcontainer - the extension is pre-configured!

## Requirements

This role requires:

- Ansible
- Python
- [eget](https://github.com/zyedidia/eget) installed on target hosts and available in `PATH`
- Administrative/root access on target hosts
- One of the following operating systems:
  - Alpine Linux
  - Debian/Ubuntu
  - NixOS or systems with Nix package manager

## Install

To install this role, you can use the following terminal command:

```shell
ansible-galaxy install git+https://github.com/Cogni-AI-OU/ansible-role-template.git
```

## Role Variables

For available variables,
check [`defaults/main.yml`](defaults/main.yml).

- `eget_packages`: List of package names to install (default: `[]`).
- `eget_install_dir`: Destination directory (default: `/usr/local/bin`).

For example:

```yaml
eget_packages:
  - bat
  - rg
```

The command map in [`vars/main.yml`](vars/main.yml) defines how to install each supported package.
Unknown package names are rejected before installation.
The role appends `--to <eget_install_dir>/<package name>` to each predefined command.
Existing binaries are skipped; remove a binary to reinstall it or change its release.
Use play-level `become: true` when the destination requires elevated privileges.
The predefined commands target x86-64 Linux; select packages compatible with your target's runtime.

## Testing

### Docker

Steps to test role on Docker containers.

1. Install the current role by running the following commands in shell:

    ```shell
    ansible-galaxy install -r requirements.yml
    jinja2 requirements-local.yml.j2 -D "pwd=$PWD" -o requirements-local.yml
    ansible-galaxy install -r requirements-local.yml
    ```

    Alternatively, for development purposes, you can consider using symbolic link, e.g.

    ```shell
    ln -vs "$PWD" ~/.ansible/roles/cogni-ai.template
    ```

2. Ensure Docker service (e.g. Docker Desktop) is running.
3. Run playbook from `tests/`:

    ```shell
    ansible-playbook -i tests/inventory/docker-containers.yml tests/playbooks/docker-containers.yml
    ```

4. Run the verify-tag playbook (stops containers after verification):

  ```shell
  ansible-playbook -i tests/inventory/docker-containers.yml tests/playbooks/tags/verify.yml --tags verify
  ```

### Molecule

To test using Molecule, run:

```shell
molecule test
```

Both scenarios install the 21 packages in [`molecule/default/packages.yml`](molecule/default/packages.yml)
and verify that each binary exists, is executable, and can report its version.
These fixtures require x86-64 Linux with glibc 2.39 or newer (Debian latest or Ubuntu 24.04/newer).
They do not target i386 Alpine, Ubuntu 22.04, or NixOS.
The prepare step installs eget and target-side runtime dependencies.
Tests require access to GitHub releases and `downloads.claude.ai`.

## Development

### Setup

```bash
# Install pre-commit hooks
pip install pre-commit
pre-commit install

# Install Python dependencies (for devcontainer)
pip install -r .devcontainer/requirements.txt
```

### Testing and Validation

```bash
# Run all pre-commit checks
pre-commit run -a

# Run specific checks
pre-commit run markdownlint -a
pre-commit run yamllint -a
pre-commit run black -a
pre-commit run flake8 -a
```

## AI Agents

This repository provides AI agent configurations for automated development.

### Agent Configuration Files

| File/Directory | Audience | Purpose |
| -------------- | -------- | ------- |
| [AGENTS.md](AGENTS.md) | All agents | Repository-specific guidance and workflows |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Copilot | Coding standards and project context |
| [.github/FIREWALL.md](.github/FIREWALL.md) | Maintainers | Firewall allowlist guidance for hosted agents |
| [.github/prompts/](.github/prompts/) | All | Prompt templates (`.md` for VS Code, `.yaml` for GitHub Models) |
| [.github/workflows/](.github/workflows/) | Automation | CI, review, and agent workflows including Cogni AI |
| [.github/workflows/cogni-ai-agent.yml](.github/workflows/cogni-ai-agent.yml) | Cogni AI | Event-driven agent workflow for issues, PRs, and discussions |

See also:

- [`AGENTS.md` file format specification](https://agents.md/)
- [Best practices for using GitHub Copilot](https://gh.io/copilot-coding-agent-tips).

## GitHub Actions

For documentation on GitHub Actions workflows, problem matchers, and CI/CD
configuration, see [.github/workflows/README.md](.github/workflows/README.md).

## Contributing

For contribution guidelines, see [CONTRIBUTING.md](https://github.com/Cogni-AI-OU/.github/blob/main/.github/CONTRIBUTING.md).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

<!-- Named links -->

[pr-reviews-image]: https://img.shields.io/github/issues-pr/Cogni-AI-OU/ansible-role-template?label=PR+Reviews&logo=github
[pr-reviews-link]: https://github.com/Cogni-AI-OU/ansible-role-template/pulls
[license-image]: https://img.shields.io/badge/License-MIT-blue.svg
[license-link]: LICENSE
[check-image]: https://github.com/Cogni-AI-OU/ansible-role-template/actions/workflows/check.yml/badge.svg
[check-link]: https://github.com/Cogni-AI-OU/ansible-role-template/actions/workflows/check.yml
