# Copilot Instructions

## Project Overview

This is an Ansible role template repository. It provides standardized structure,
GitHub Actions workflows, issue/PR templates, and coding standards for creating
reusable Ansible roles.

Key contents:

- **Ansible role structure**: Standard role directories (tasks, handlers, templates, defaults, vars, meta)
- **CI/CD workflows**: Pre-commit checks, linting, Molecule testing
- **Agent configurations**: `AGENTS.md` and workflow metadata for AI coding assistants

### Getting started

- Refer to the `README.md` in the project root for setup and installation instructions.
- Check also `.tours/getting-started.tour` which provides a guided walkthrough of key project features and structure.

## Coding Standards

### Python

- Use **Python 3.11+**.
- Use `uv` script headers for dependency management:

  ```python
  #!/usr/bin/env -S uv run --script
  # /// script
  # requires-python = ">=3.11"
  # dependencies = [
  #     "xero-python",
  #     "PyYAML",
  # ]
  # ///
  ```

- Follow **PEP 8** style guidelines.
- Use `argparse` for CLI argument parsing.
- Handle `BrokenPipeError` for CLI tools that might be piped to `head` or `grep`.

## Formatting Guidelines

### JSON

- Follow `.editorconfig` spacing and trailing-newline conventions.
- Validate JSON with `jq` or the VS Code JSON formatter.

### Markdown

- Keep headings and lists surrounded by blank lines.
- Use fenced code blocks with a language identifier.
- Keep lines at or below 120 characters.
- Validate with `pre-commit run markdownlint -a`.

### YAML

- Use 2-space indentation and keep lines at or below 120 characters.
- Use explicit `true` and `false` values.
- Keep workflow keys and environment variables in lexicographical order when practical.
- Validate with `pre-commit run yamllint -a`; use `actionlint` for workflow-specific checks.

Notes:

- Project utilizes Codespaces with config at `.devcontainer/devcontainer.json` and requirements at `.devcontainer/requirements.txt`.
- GitHub Actions run pre-commit checks (`.pre-commit-config.yaml`).
- To verify locally, run `pre-commit run yamllint -a` from the repo root.

### Devcontainer Guidance

Project utilizes Codespaces with config at `.devcontainer/devcontainer.json`
and requirements at `.devcontainer/requirements.txt`.
These rules apply only when the agent is running inside GitHub Codespaces
or the repository's VS Code devcontainer.
In that environment, the container should be treated as the controller
runtime and as the source of required controller-side dependencies.

- Treat the repository devcontainer as the default controller environment
  for local development and testing.
- Keep controller dependency installation in the devcontainer
  configuration so Molecule scenarios can assume those tools are already
  available.
- Do not install controller-side Python dependencies during Molecule runs
  when the agent is already operating inside Codespaces or the repo
  devcontainer.
- Do not create additional Python virtual environments such as `.venv`
  or `venv`; use the existing container Python environment, which should
  already provide the required dependencies.
- If dependencies are missing in a Codespace or devcontainer, update
  `.devcontainer/requirements.txt` or `.devcontainer/devcontainer.json`
  instead of introducing a per-run Molecule install step or a separate
  virtual environment.
- Use `CODESPACES=true` as the primary quick check for GitHub Codespaces.
- Outside Codespaces, use the presence of `/.dockerenv` together with
  the repository `.devcontainer/devcontainer.json` as a practical check
  for the containerized dev environment.

## Project Structure

```text
.
├── .github/
│   ├── FIREWALL.md          # Firewall allowlist guidance for hosted agents
│   ├── prompts/             # Prompt templates for editors and models
│   ├── workflows/           # GitHub Actions workflows
│   └── copilot-instructions.md
├── .tours/                  # VS Code guided tours
├── defaults/                # Default role variables
├── handlers/                # Handler tasks
├── meta/                    # Role metadata and dependencies
├── molecule/                # Molecule test scenarios
├── tasks/                   # Main role tasks
├── templates/               # Jinja2 templates
├── vars/                    # Role variables
├── AGENTS.md                # AI agent guidance
└── README.md                # Repository documentation
```

### Tours

- Keep the `.tours` folder up-to-date (especially `.tours/getting-started.tour`)
  when making significant changes to the codebase.
  Update existing tours or create new ones to reflect changes in project structure,
  workflows, or key files.

## Git Operations

When working with the user interactively (e.g., in an IDE like VS Code):

- **Never create commits or push changes to branches** without explicit user feedback or requests.
- Present the proposed changes, successfully save the edited files, and allow the user to review the diffs locally.
- Let the user drive the Git staging, committing, and pushing processes,
  or wait for them to explicitly instruct you to perform these operations.

## Troubleshooting

### Finding Build Errors

To identify and diagnose the latest build errors:

1. **Reproduce errors locally:**
   - For pre-commit errors: Run `pre-commit run -a` to check all files
   - For specific hooks: Run `pre-commit run <hook-name> -a` (e.g., `markdownlint`, `yamllint`)
   - For actionlint errors: Install actionlint and run it on workflow files

2. **Common error patterns:**
   - **Ansible missing Python modules:** If a module such as `requests` or
     `docker` is installed for the main container Python but Ansible still
     cannot import it, check `ansible --version` to identify the interpreter in
     use. In Codespaces/devcontainers, Ansible may run from a pipx-managed
     environment, so install controller-side libraries there as well, for
     example with `pipx inject ansible -r .devcontainer/requirements-ansible.txt`.
   - **Markdown linting errors:** Check `.markdownlint.yaml` for rules; errors show line numbers
   - **YAML linting errors:** Check `.yamllint` for rules; verify indentation and structure
   - **JSON formatting errors:** Use `jq . <file>` to validate JSON syntax
