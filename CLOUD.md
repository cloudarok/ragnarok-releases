# Ragnarok Cloud · DigitalOcean

Ragnarok can configure and create DigitalOcean VMs, prepare a verified runtime installer, and request start, graceful shutdown, or reboot for machines it created. This is a bring-your-own-account integration. A provisioned machine is not automatically a connected scheduler worker.

## Create a machine

1. Open **Cloud** in the local Web UI or desktop app.
2. Connect a DigitalOcean API token with `account:read`, `size:read`, `region:read`, `image:read`, `ssh_key:read`, `droplet:read`, and `droplet:create`. Lifecycle controls additionally need `droplet:update`; action reconciliation may require `action:read` according to your token scope settings.
3. Choose an available VM size, region and an account SSH key. Password-only access is not offered.
4. Review the current compute estimate and confirm creation. Size, region, public key and price are checked again before sending the paid request.
5. Refresh the machine list for the provider's actual state and IP.

Machines shown belong to the connected DigitalOcean account and were created by this Ragnarok installation. Provider API access is restricted to the official HTTPS endpoint, without redirects. Tokens stay in the desktop encrypted vault when available, otherwise in runtime memory; `DIGITALOCEAN_TOKEN` is also supported. Local cloud receipts contain identifiers/configuration, never the token.

## Install the supervised runtime

Choose setup for the created machine and supply the public Cloud runtime asset URL and its SHA256 from the same release. Only the `cloudarok/ragnarok-releases` Cloud runtime asset is accepted. Administrators can preconfigure `RAGNAROK_CLOUD_BUNDLE_URL` and `RAGNAROK_CLOUD_BUNDLE_SHA256` in the local runtime environment instead.

Ragnarok generates a **reviewable Bash installer** and its SHA256. Download it as `ragnarok-cloud-install.sh`, read it, then use the displayed SCP/SSH command to run it on the VM. Generating a plan does not execute SSH, upload credentials or source repositories, charge a provider, or claim installation succeeded.

The installer:

- Targets Ubuntu 24.04 x64 and refuses to overwrite an existing runtime installation.
- Downloads the public runtime-only archive, verifies its SHA256 and rejects unexpected archive paths, links and excessive expanded size.
- Installs Node.js **24.21.0** from the official distribution with its pinned [upstream SHA256](https://nodejs.org/dist/v24.21.0/SHASUMS256.txt).
- Runs the app under a dedicated unprivileged `ragnarok` user with a systemd service, restart limits, graceful shutdown, memory/CPU/process limits, read-only system paths and a private temp directory.
- Stores state/workspaces under `/var/lib/ragnarok`, keeps app code root-owned under `/opt/ragnarok`, and binds the runtime only to loopback on port 4173.
- Checks runtime readiness without copying its session token into logs.

The service has outbound network access for model providers and authorized tools. This is a single-tenant VM boundary; tasks are not isolated from one another within the service account. CPU is initially limited to two cores and memory to 80% of the machine. Review systemd overrides for larger machines and workloads.

## Connect privately

The generated tunnel command uses your SSH client and normal host-key verification:

```sh
ssh -o ExitOnForwardFailure=yes -o ServerAliveInterval=30 -N -L 127.0.0.1:14173:127.0.0.1:4173 root@YOUR_VM_IP
```

Keep the tunnel open, then visit `http://127.0.0.1:14173`, or run `rag terminal --port 14173`. The forwarded port is bound only to your computer's loopback. Do not expose the runtime port through a public proxy or firewall rule. This daemon's local bootstrap endpoint assumes a trusted single-user host; it is not an Internet authentication service.

Clone/transfer your chosen repository to `/var/lib/ragnarok/workspaces` with ownership assigned to `ragnarok`. Local computer paths do not refer to VM directories. Model setup in the remote Web UI is memory-only without a desktop vault. For persistent provider configuration, edit the root-only `/etc/ragnarok/environment` file on the VM, then restart the service. Avoid placing provider keys in shell history, cloud-init user-data, screenshots, or project files. Keys and repositories are not automatically copied from desktop.

## Operate and recover

```sh
systemctl status ragnarok --no-pager
journalctl -u ragnarok -n 100 --no-pager
systemctl restart ragnarok
```

Start, graceful shutdown and reboot requests each require confirmation and a fresh request ID. Ragnarok writes a receipt before contacting DigitalOcean, checks current machine state, and does not automatically retry uncertain results. Refresh actions to reconcile known provider action IDs. For an uncertain request without an action ID, inspect DigitalOcean before attempting another mutation. A graceful shutdown is a request; verify that the machine actually reached the powered-off state.

**Powering off a machine does not stop DigitalOcean compute charges.** Destroy it in DigitalOcean to end its compute charges; associated resources may have separate billing. Ragnarok intentionally has no destructive delete or hard-power-off control in this release. See [DigitalOcean pricing](https://docs.digitalocean.com/products/droplets/details/pricing/).

Back up `/var/lib/ragnarok` and protect `/etc/ragnarok/environment` before upgrades or machine destruction. Managed backups, restore drills, automatic upgrades, remote queue registration and multi-machine dispatch remain separate work. The setup page reports `installationVerified: false` until independent remote health verification exists; an accepted Droplet request or generated script is not proof the harness is running.

## Build the public runtime asset

```sh
npm run build:command
python3 scripts/package-cloud-runtime.py
```

The output is `outputs/cloud/Ragnarok-<version>-cloud-runtime.tar.gz` plus a checksum file. Its allowlist is `server.mjs`, `packages/core/src`, `dist/command`, and `package.json`. It excludes the source repository, Git metadata, source maps, `.env`, credentials, local databases and private source archives. Publish only after testing the corresponding release.

## Validation boundary

Cloud tests use injected DigitalOcean fixtures to cover creation, confirmations, account ownership, lifecycle idempotency, uncertain-result recovery, credential filtering and bootstrap generation. The bootstrap is syntax checked, and the runtime-only bundle has been built locally. No real DigitalOcean account, paid VM, remote systemd installation or live SSH tunnel was exercised.

API behavior follows the official [Droplet creation schema](https://github.com/digitalocean/openapi/blob/master/specification/resources/droplets/droplets_create.yml) and [Droplet actions documentation](https://docs.digitalocean.com/reference/api/reference/droplet-actions/).
