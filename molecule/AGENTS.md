# Molecule Testing

Molecule and Ansible are installed via the project `Pipfile`, so run every command through `pipenv`
(they are not on `PATH`).

```bash
# Run Molecule tests
pipenv run molecule test

# Syntax check
pipenv run molecule syntax
```

## Molecule Platforms

Molecule platform names follow the `<role>-<scenario>-<platform>` convention, e.g.
`template-default-debian-latest`. Molecule's Docker driver names each container
exactly after its platform, so generic names such as `debian-latest` would collide
with concurrent Molecule runs of other roles or scenarios.

## Sandboxed / firewalled environments

Molecule defaults work on GitHub Actions runners with direct internet access. In sandboxed or
firewalled environments (no outbound NAT on the default Docker bridge, or a resolver that returns
non-routable IPv6 addresses), opt in with these environment variables:

- `MOLECULE_DOCKER_NETWORK=host` - Docker network used for both containers and image builds.
  Required where the default bridge has no outbound NAT.
- `MOLECULE_DOCKER_FORCE_IPV4=true` - prefer IPv4 for DNS resolution inside containers. Required
  where the resolver returns IPv6 addresses that are not routable.
- `MOLECULE_XVFB_DISPLAY_BASE` - only applies to repos that run the xvfb role; ignored elsewhere.

```bash
# Sandboxed run with host networking and IPv4 DNS preference
MOLECULE_DOCKER_NETWORK=host MOLECULE_DOCKER_FORCE_IPV4=true pipenv run molecule test
```

## Troubleshooting

### Molecule prepare fails with DNS resolution errors

> `Temporary failure resolving 'deb.debian.org'` (or `azure.archive.ubuntu.com`), followed by
> `E: Unable to locate package python3` during the `prepare` step.

- **Root cause**: `docker0` has lost the `bridge` network's configured gateway address, so containers
  on the default bridge have no working gateway and cannot resolve DNS or reach the network.
- **Check**: `ip -4 addr show docker0` vs
  `docker network inspect bridge --format '{{range .IPAM.Config}}{{.Gateway}}{{end}}'`.
  If the gateway address is missing from `docker0`, this is the cause.
- **Fix (durable)**: `sudo systemctl restart docker` recreates `docker0` with the configured gateway.
  This restarts the daemon and stops any running containers.
- **Fix (non-disruptive, not persistent)**: `sudo ip addr add 172.17.0.1/16 dev docker0`
  (use the gateway reported by the check above).
- **Workaround (no sudo)**: run the tests on the host network, which bypasses the broken bridge.

```bash
MOLECULE_DOCKER_NETWORK=host \
MOLECULE_DOCKER_FORCE_IPV4=true \
MOLECULE_XVFB_DISPLAY_BASE=90 \
pipenv run molecule test
```
