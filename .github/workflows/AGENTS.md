# GitHub Actions Workflows (Agent Catalog)

Authoritative, agent-facing catalog of workflows in this repository. Use this when loading or modifying
workflows and keep it in sync with the files in this directory.

For a human-readable overview, see [README.md](README.md).

## Workflow catalog

- **[check-pr-comment.yml](check-pr-comment.yml)**: PR feedback/labels for `Check` and devcontainer results.
- **[check.yml](check.yml)**: Linting and quality gates via actionlint and pre-commit.
- **[cogni-ai-agent.yml](cogni-ai-agent.yml)**: Logic for the Cogni AI Agent.
- **[copilot-setup-steps.yml](copilot-setup-steps.yml)**: Environment setup utility.
- **[devcontainer-ci.yml](devcontainer-ci.yml)**: Build/test devcontainer and required tools/packages.
- **[molecule.yml](molecule.yml)**: Molecule test matrix for the default scenario.
- **[test.yml](test.yml)**: Ansible syntax/lint checks and Docker container playbook tests.

## Details

### check.yml

- Purpose: run actionlint and pre-commit to enforce workflow and repo standards.
- Triggers: `push`, `pull_request`, `schedule`, `workflow_call`, `workflow_dispatch`,
  `workflow_run` (after bot workflow completions).
- Bot-PR support: `workflow_run` trigger enables checks on PRs created by bots,
  since normal `pull_request` events don't trigger for bot actors.
- Reusable: `uses: Cogni-AI-OU/.github/.github/workflows/check.yml@main`.
- Jobs: `actionlint`, `link-checker`, `pre-commit`.

### check-pr-comment.yml

- Purpose: extract annotations from failed `Check` and `Development Containers (CI)` runs and post
  them as a PR comment.
- Triggers: `workflow_run` (after `Check` or `Development Containers (CI)` completions).
- Details: Manages the `check-error` label; updates an existing comment via marker detection.
- Reusable: `uses: Cogni-AI-OU/.github/.github/workflows/check-pr-comment.yml@main`.

### cogni-ai-agent.yml

- Purpose: provides the underlying logic to run the Cogni AI Agent.
- Triggers: `issue_comment`, `pull_request_review_comment`, `workflow_dispatch`.
- Details: Installs Python dependencies from `.devcontainer/requirements.txt` and calls the
  `Cogni-AI-OU/cogni-ai-agent-action` to process instructions. A post-run `summary` job generates
  an AI summary of the agent's actions.
- Concurrency: Only one run per issue/PR/branch at a time; new runs are queued (no auto-cancel).
- Permissions: `contents: write`, `id-token: write`, `issues: write`, `pull-requests: write`.

### copilot-setup-steps.yml

- Purpose: utility workflow for setting up the environment.
- Triggers: `push` and `pull_request` on `copilot-setup-steps.yml` or `.devcontainer/requirements.txt`.
- Details: Checks out repo, sets up Python 3.12, restores cache, and installs dependencies.
- Permissions: `contents: read`.

### devcontainer-ci.yml

- Purpose: build and validate the dev container; ensure required tools and Python packages exist.
- Inputs: `required_commands` (defaults to common CLI tools), `required_python_packages`
  (defaults to ansible, ansible-lint, docker, molecule, pre-commit, uv).
- Triggers: pull_request/push affecting `.devcontainer/` or this workflow; weekly schedule;
  `workflow_call`.
- Permissions: callers must grant `packages: write` when pushing images to GHCR.
- Reusable: `uses: Cogni-AI-OU/.github/.github/workflows/devcontainer-ci.yml@main`.

### molecule.yml

- Purpose: run Molecule tests for the default scenario.
- Triggers: `pull_request`, `push`, `workflow_dispatch`.
- Details: Sets up Python 3.12 and runs `molecule test` via `gofrolist/molecule-action@v2`.
- Permissions: `contents: read`.

### test.yml

- Purpose: validate Ansible syntax/lint and run the Docker container test playbook.
- Triggers: `pull_request`, `push`, `workflow_dispatch`.
- Details: Installs Python and Galaxy requirements, runs `ansible-playbook --syntax-check`,
  `ansible-lint`, and the `tests/playbooks/docker-containers.yml` playbook (plus its verify tag).
- Permissions: `contents: read`.

## Notes

- Follow the GitHub workflows instructions (available in the runtime instructions catalog)
  when editing workflow files (ordering, formatting, validation).
- Reference actions by simple major version tags (e.g. `actions/checkout@v6`), not pinned
  patch versions (e.g. `@v6.1.0`), so minor/patch updates apply automatically.
- Keep this catalog updated when workflows are added, removed, or renamed.
