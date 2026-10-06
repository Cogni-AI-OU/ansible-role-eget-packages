# Molecule Testing

## Running Tests

Run these commands from the repository root:

```shell
pipenv run molecule test
pipenv run molecule test -s devel
```

## Coverage and Requirements

Both scenarios install the packages in [`default/packages.yml`](default/packages.yml)
and verify that each binary exists, is executable, and can report its version.
The package list selects predefined eget commands from [`vars/main.yml`](../vars/main.yml).
The test sequence also checks that a second convergence makes no changes.

These fixtures require x86-64 Linux with glibc 2.39 or newer (Debian latest or Ubuntu 24.04/newer).
They do not target i386 Alpine, Ubuntu 22.04, or NixOS.
The prepare step installs eget and target-side runtime dependencies.
Tests require Docker and access to GitHub releases and `downloads.claude.ai`.

Set `EGET_GITHUB_TOKEN` (or `GITHUB_TOKEN`) in the controller environment to authenticate GitHub release requests
and avoid the unauthenticated API rate limit. Convergence forwards it to eget in the test containers;
the GitHub Actions workflow already supplies `GITHUB_TOKEN`.

See [`AGENTS.md`](AGENTS.md) for sandbox networking options and troubleshooting.
