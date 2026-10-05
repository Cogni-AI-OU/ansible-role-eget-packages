# Docker Tests

Agent guidance for the standalone Docker test playbooks in `tests/`.

These are the non-Molecule test path: they start real distro containers, install the role into each,
and are what the `Test` CI workflow runs. For the Molecule path, see
[../molecule/AGENTS.md](../molecule/AGENTS.md).

## Layout

| Path | Purpose |
| --- | --- |
| `inventory/docker-containers.yml` | Six-host distro matrix (alpine, debian, nixos, ubuntu jammy/noble/latest) |
| `playbooks/docker-containers.yml` | Builds/pulls distro containers, then installs the role on each |
| `playbooks/tags/verify.yml` | Tag-driven variant that brings the containers up and imports the role with `verify` |
| `playbooks/tasks/install-python.yml` | Bootstraps Python 3 in the container; shared with Molecule `prepare` |
| `test.yml` | Minimal localhost playbook that applies the role from the working tree |
| `vars/main.yml` | Shared variables (`controller_pip`, `nixos_built_image`, Dockerfile paths) |

The inventory uses `ansible_connection: docker`, so the containers are addressed by name over the
Docker socket rather than SSH.

## Running Tests

Run everything through `pipenv`, matching [../molecule/AGENTS.md](../molecule/AGENTS.md):

```bash
# Install/refresh dependencies from Pipfile.lock (first run)
pipenv sync

# Distro matrix (pulls images, builds the NixOS image, ~2 min)
pipenv run ansible-playbook -i tests/inventory/docker-containers.yml tests/playbooks/docker-containers.yml

# Tag-driven verify variant
pipenv run ansible-playbook -i tests/inventory/docker-containers.yml tests/playbooks/tags/verify.yml
```

Syntax check only (what CI runs first):

```bash
pipenv run ansible-playbook --syntax-check \
  -i tests/inventory/docker-containers.yml tests/playbooks/docker-containers.yml
```

`pipenv` auto-detects the project's `.venv` directory (see `is_venv_in_project` in pipenv's
`project.py`), so `pipenv run <cmd>` and `.venv/bin/<cmd>` resolve to the same interpreter. Prefer
`pipenv run` - it does not depend on the caller's `PATH` or on the venv being activated.

### Why pipenv

`Pipfile.lock` pins the exact versions the tests were validated against (`ansible-core 2.17.9`,
`ansible-compat 25.1.4`, `molecule 25.3.1`, `molecule-docker 2.1.0`), so `pipenv sync` reproduces the
environment deterministically. Installing `ansible`/`ansible-lint` ad hoc instead pulls the latest
`ansible-core` (2.21.x), which is a different runtime than CI uses.

`pipenv` does **not** provide `ansible-lint` - it is not in the `Pipfile`. Run lint via
`pre-commit run ansible-lint -a`, or install it separately.

## Prerequisites

- Docker daemon reachable and a working default bridge (see the troubleshooting entry below).
- Ansible collections installed: `ansible-galaxy collection install -r requirements.yml`
  (`community.docker >= 3.10.2`, `community.general >= 8.6.0`).
- The role resolvable as `cogni-ai.template`. The playbooks use `ansible.builtin.import_role`, which
  resolves from `~/.ansible/roles/` - not from the working tree. Either install it
  (`ansible-galaxy install -r requirements-local.yml`) or symlink it for development:

    ```bash
    ln -vs "$PWD" ~/.ansible/roles/cogni-ai.template
    ```

## What the Playbooks Do

`docker-containers.yml` runs two plays:

1. **Configure Docker container** - renders `molecule/resources/playbooks/Dockerfile.j2` to
   `Dockerfile.template-on-nixos` and builds `cogni-ai-template-nixos:latest` (only when
   `template-on-nixos-latest` is in the inventory), then starts each container with
   `recreate: true`, waits for it to be running, bootstraps Python 3, and gathers facts. NixOS
   containers get `privileged: true` and relaxed seccomp/apparmor.
2. **Install cogni-ai.template role** - applies the role to every container, then stops the containers.

`tags/verify.yml` is the same shape, but splits the NixOS build into its own play and imports the role
under the `verify` tag.

## Verifying the Result

This role is a scaffold: `tasks/main.yml` only includes `tasks/verify.yml`, which is empty. There is
nothing functional to assert, so the playbooks only *install* the role and a green run means the role
resolved and ran without error - not that any service came up.

### Idempotency

`AGENTS.md` requires idempotent tasks. The playbooks recreate containers on every run, so a second
run is not a valid idempotency check. Run the role twice against a container that is already
converged instead - the second run must report `changed=0`:

```bash
docker run -d --name template-on-package-image ubuntu:latest sleep infinity
pipenv run ansible-playbook -i tests/inventory/docker-containers.yml \
  --limit template-on-package-image tests/playbooks/docker-containers.yml
```

## Troubleshooting Matrix

### `ansible-lint` cannot resolve `community.docker.*`

> `syntax-check[unknown-module]: couldn't resolve module/action 'community.docker.docker_image'`

- **Root cause**: a stale, empty `.ansible/collections/ansible_collections/community/docker`
  directory shadows the real collection. This is the blocker documented in the root `AGENTS.md`.
- **Check**: `ls -la .ansible/collections/ansible_collections/community/docker/` - if it contains
  only an empty `roles/` directory, this is the cause.
- **Fix**: `rm -rf .ansible/collections/ansible_collections/community/docker`. `.ansible` is
  gitignored and holds no tracked files, so this is safe.

### Docker bridge has no gateway (containers cannot reach the network)

> `apk update` / `apt-get update` fails, or `getent hosts` returns nothing, while the host resolves
> and routes fine.

- **Root cause**: `docker0`'s address does not match the `bridge` network's configured gateway, so
  containers get a default route pointing at an address that is not on the bridge.
- **Check**: `ip -4 addr show docker0` vs
  `docker network inspect bridge --format '{{range .IPAM.Config}}{{.Subnet}} {{.Gateway}}{{end}}'`.
  If the gateway from the second command is missing from the first, the bridge is broken.
- **Fix**: `sudo systemctl restart docker` recreates `docker0` with the configured gateway. This
  stops running containers, including an in-flight test run.
- **Note**: `docker0` may carry more than one address. As long as the gateway reported by
  `docker network inspect bridge` is present on `docker0`, the bridge works even if the subnet
  differs from `bip` in `/etc/docker/daemon.json`.

### Running the tests dirties the working tree

> `git status` reports `M molecule/resources/playbooks/Dockerfile.template-on-nixos` after a run.

- **Root cause**: `docker-containers.yml` renders that file from
  `molecule/resources/playbooks/Dockerfile.j2` on every run. The committed copy is stale relative to
  the template, so the render always produces a diff.
- **Impact**: cosmetic for the test result, but it makes the tree dirty and can mask real changes.
- **Fix**: regenerate and commit the file, or stop tracking it. Until then, revert with
  `git checkout -- molecule/resources/playbooks/Dockerfile.template-on-nixos`.

### `community.general does not support Ansible version 2.17.9`

> `[WARNING]: Collection community.general does not support Ansible version 2.17.9`

- **Root cause**: the installed `community.general` expects a newer `ansible-core` than the one
  pinned in `Pipfile.lock` (2.17.9).
- **Impact**: warning only; the tests pass. Do not "fix" it by upgrading `ansible-core` ad hoc - that
  diverges from the pinned environment CI uses.

### `molecule` fails with `FileNotFoundError: 'ansible-config'`

> Running a venv binary directly (`.venv/bin/molecule`) raises
> `FileNotFoundError: [Errno 2] No such file or directory: 'ansible-config'`.

- **Root cause**: `ansible_compat` shells out to `ansible-config` by name. Invoking the binary
  directly does not put the venv's `bin` on `PATH`, so the lookup fails.
- **Fix**: use `pipenv run molecule ...` - pipenv prepends the venv's `bin` to `PATH`. Activating
  the venv (`source .venv/bin/activate`) works too. This is the main reason to prefer `pipenv run`
  over calling `.venv/bin/*` directly.

### `pipenv` warns that Python 3.10 was not found

> `Warning: Python 3.10 was not found on your system...`

- **Root cause**: `Pipfile` pins `python_version = "3.10"`, but the project `.venv` was created with
  a different Python version.
- **Impact**: harmless when `.venv` already exists, because pipenv reuses it. Only relevant when
  creating a fresh environment.
- **Fix**: install Python 3.10 (`pyenv`/`asdf`), or create the venv explicitly with
  `python3 -m venv .venv` and use `pipenv run` / `.venv/bin` directly.
