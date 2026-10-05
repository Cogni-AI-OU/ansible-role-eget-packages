# .github Directory

Use this as the entry point for agent work, and follow linked catalogs when relevant.

## Directory-Specific Agent files

Read these Agent files when working in corresponding dirs:

- [`prompts/AGENTS.md`](prompts/AGENTS.md)
- [`workflows/AGENTS.md`](workflows/AGENTS.md)

## Additional key files

- [.github/copilot-instructions.md](copilot-instructions.md): main coding standards for agents.
- [.github/mcp-config.json](mcp-config.json): MCP server configuration for GitHub Copilot.
  Provides access to built-in GitHub tools including:
  - Repository & Code: `get_file_contents`, `search_code`, `search_repositories`, `list_branches`, `list_commits`
  - Issues & PRs: `get_issue`, `list_pull_requests`, `create_pull_request`
  - Actions: `list_workflows`, `list_workflow_runs`, `get_job_logs`
- [.github/actionlint-matcher.json](actionlint-matcher.json): problem matchers used in workflows.
- [.github/pre-commit-matcher.json](pre-commit-matcher.json): problem matchers used in workflows.
- [.github/workflows/README.md](workflows/README.md): documentation for GitHub workflows, agents, and problem matchers.
- [.github/FIREWALL.md](FIREWALL.md): firewall configuration and recommended hosts for agents.

## Hardened NEVER List

- **NEVER create `.github/README.md`**: GitHub renders `.github/README.md` with the highest priority. Creating it will
  override the main `README.md` on the repository homepage and profile page.

## Troubleshooting

### GitHub Actions

If a workflow fails, check the logs for:

- Syntax errors (actionlint)
- Permission issues (GITHUB_TOKEN)
- Missing secrets

## Additional notes

- Keep this Agent file up-to-date.
