# OpenStack Core — Agentic Stack

Comprehensive operational knowledge for all major OpenStack services. This stack teaches AI agents to deploy, operate, troubleshoot, and extend OpenStack clouds.

## What This Covers

| Domain | Services | Depth |
|---|---|---|
| Identity | Keystone | Architecture, CLI/API, internals, federation |
| Compute | Nova, Placement | Architecture, CLI/API, internals, scheduling, live migration |
| Networking | Neutron | Architecture, CLI/API, internals, OVS, OVN, advanced networking |
| Block Storage | Cinder | Architecture, CLI/API, internals, backends |
| Image | Glance | Architecture, CLI/API, internals, import workflows |
| Object Storage | Swift | Architecture, CLI/API, internals, rings, erasure coding |
| Shared Filesystems | Manila | Architecture, CLI/API, internals |
| Orchestration | Heat, Magnum | Architecture, CLI/API, internals, templates |
| Load Balancing | Octavia | Architecture, CLI/API, internals, amphora |
| DNS | Designate | Architecture, CLI/API, internals, backend drivers |
| Bare Metal | Ironic | Architecture, CLI/API, internals, drivers |
| Key Management | Barbican | Architecture, CLI/API, internals, HSM |
| Dashboard | Horizon, Skyline | Configuration, customization, internals |

## Supported Releases

- **2025.1 (Epoxy)** — SLURP release
- **2025.2**
- **2026.1**

SLURP upgrade path: 2025.1 → 2026.1 (skipping 2025.2).

## Skills

| Skill | Entry Point | Description |
|---|---|---|
| architecture | `skills/foundation/architecture/` | Overall OpenStack architecture |
| deployment-overview | `skills/foundation/deployment-overview/` | Deployment methods and references |
| configuration | `skills/foundation/configuration/` | Common config patterns |
| identity | `skills/identity/` | Keystone |
| compute | `skills/compute/` | Nova + Placement |
| networking | `skills/networking/` | Neutron |
| block-storage | `skills/storage/block/` | Cinder |
| image | `skills/storage/image/` | Glance |
| object-storage | `skills/storage/object/` | Swift |
| shared-filesystem | `skills/storage/shared-filesystem/` | Manila |
| heat | `skills/orchestration/heat/` | Heat |
| magnum | `skills/orchestration/magnum/` | Magnum |
| load-balancing | `skills/load-balancing/` | Octavia |
| dns | `skills/dns/` | Designate |
| bare-metal | `skills/bare-metal/` | Ironic |
| security | `skills/security/` | Barbican + hardening |
| dashboard | `skills/dashboard/` | Horizon + Skyline |
| health-check | `skills/operations/health-check/` | Cross-service health |
| upgrades | `skills/operations/upgrades/` | Release upgrades + SLURP |
| backup-restore | `skills/operations/backup-restore/` | Backup strategies |
| capacity-planning | `skills/operations/capacity-planning/` | Quotas and planning |
| diagnose | `skills/diagnose/` | Troubleshooting |
| decision-guides | `skills/reference/decision-guides/` | Component selection |
| compatibility | `skills/reference/compatibility/` | Version matrix |
| known-issues | `skills/reference/known-issues/` | Bugs and workarounds |

## Quick Start

```bash
agentic-stacks init my-openstack
cd my-openstack
agentic-stacks pull openstack-core
```

Then describe your cloud to the agent — it will guide you from architecture through operations.

## Composability

This stack covers service knowledge. For deployment tooling, pair with:
- [openstack-kolla](https://github.com/agentic-stacks/openstack-kolla) — kolla-ansible deployment

## Requirements

- OpenStack CLI (`python-openstackclient`)
