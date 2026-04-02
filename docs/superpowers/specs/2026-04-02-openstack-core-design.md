# OpenStack Core — Agentic Stack Design

## Overview

An agentic stack that teaches AI agents to operate, deploy, and extend all major OpenStack services. Covers architecture, CLI/API operations, service internals, and day-two management across OpenStack 2025.1 (Epoxy), 2025.2, and 2026.1.

Targets two personas:
- **Operators** managing production OpenStack clouds
- **Developers** building on OpenStack APIs and extending services

OS-agnostic, with notes on OS-specific differences where they matter.

## Target Releases

- **2025.1 (Epoxy)** — SLURP release
- **2025.2**
- **2026.1**

SLURP upgrade path: 2025.1 → 2026.1 (skipping 2025.2). Standard paths: 2025.1 → 2025.2 → 2026.1.

## Services In Scope

14 major OpenStack projects:

| Service | Project | Domain |
|---|---|---|
| Identity | Keystone | `skills/identity/` |
| Compute | Nova + Placement | `skills/compute/` |
| Networking | Neutron | `skills/networking/` |
| Block Storage | Cinder | `skills/storage/block/` |
| Image | Glance | `skills/storage/image/` |
| Object Storage | Swift | `skills/storage/object/` |
| Shared Filesystems | Manila | `skills/storage/shared-filesystem/` |
| Orchestration | Heat | `skills/orchestration/heat/` |
| Container Orchestration | Magnum | `skills/orchestration/magnum/` |
| Load Balancing | Octavia | `skills/load-balancing/` |
| DNS | Designate | `skills/dns/` |
| Bare Metal | Ironic | `skills/bare-metal/` |
| Key Management | Barbican | `skills/security/` |
| Dashboard | Horizon + Skyline | `skills/dashboard/` |

## Identity

> You are an expert OpenStack operator and architect. You understand the internals of every core OpenStack service, can deploy and configure them from packages or source, operate production clouds via CLI and API, and guide developers building on OpenStack. You work across all major releases (2025.1 Epoxy through 2026.1) and help operators regardless of their host OS.

## Critical Rules

1. **Never delete a project/tenant without explicit operator approval** — cascading deletes remove all resources (VMs, volumes, networks). Irreversible.
2. **Never revoke or delete Keystone admin tokens/users without approval** — can lock out all administrative access to the cloud.
3. **Never force-delete volumes that are attached to instances** — causes data corruption and can crash the instance.
4. **Never modify the database directly** — use service CLI/API. Direct DB edits bypass validation and break state consistency.
5. **Always backup databases before upgrades** — the only recovery path if an upgrade fails.
6. **Always run service prechecks/status before upgrades** — catch issues before they cascade.
7. **Always check known issues for the target version before deploying or upgrading** — read `skills/reference/known-issues/` first.
8. **Never take remediation action without operator approval** — present findings and recommended fix, let the operator decide.
9. **Always verify Keystone endpoint catalog after any service deployment** — misconfigured endpoints silently break inter-service communication.
10. **Never disable or reconfigure Neutron agents on live nodes without a maintenance window** — network disruption affects all tenants.

## Skill Hierarchy

Service-centric organization with shared foundation, operations, and reference layers.

```
skills/
├── foundation/
│   ├── architecture/              # Overall OpenStack architecture, message bus, DB, API patterns
│   ├── deployment-overview/       # Deployment methods overview, references to other stacks
│   └── configuration/            # Common config patterns (oslo.config, paste.ini, policy.yaml)
├── identity/
│   ├── README.md                 # Keystone overview + routing
│   ├── architecture.md           # Token providers, federation, fernet, domains/projects/users
│   ├── operations.md             # CLI/API: manage users, projects, roles, endpoints, catalog
│   ├── internals.md              # Token lifecycle, auth plugins, middleware pipeline
│   └── federation.md             # SAML, OIDC, K2K, identity providers
├── compute/
│   ├── README.md
│   ├── architecture.md           # Nova components: scheduler, conductor, compute agent, cells
│   ├── operations.md             # CLI/API: instances, flavors, keypairs, migrations, quotas
│   ├── internals.md              # Scheduler filters/weighers, cells v2, placement integration
│   ├── placement.md              # Placement service: resource providers, inventories, allocations
│   └── live-migration.md         # Deep-dive: live migration types, requirements, troubleshooting
├── networking/
│   ├── README.md
│   ├── architecture.md           # Neutron: server, agents, ML2, network types
│   ├── operations.md             # CLI/API: networks, subnets, routers, floating IPs, security groups
│   ├── internals.md              # ML2 drivers, L2/L3 agents, DHCP agent, metadata agent
│   ├── ovs.md                    # Open vSwitch deep-dive
│   ├── ovn.md                    # OVN deep-dive
│   └── advanced.md               # Trunking, QoS, segments, BGP, VPNaaS
├── storage/
│   ├── block/
│   │   ├── README.md             # Cinder overview
│   │   ├── architecture.md       # Cinder: API, scheduler, volume service, backup service
│   │   ├── operations.md         # CLI/API: volumes, snapshots, backups, types, QoS
│   │   ├── internals.md          # Driver architecture, multi-backend, replication
│   │   └── backends.md           # LVM, Ceph, NFS, iSCSI, vendor drivers
│   ├── image/
│   │   ├── README.md             # Glance overview
│   │   ├── architecture.md       # Glance: API, registry (deprecated), stores
│   │   ├── operations.md         # CLI/API: images, import, tasks, properties
│   │   └── internals.md          # Store backends, image caching, interoperable import
│   ├── object/
│   │   ├── README.md             # Swift overview
│   │   ├── architecture.md       # Swift: proxy, account/container/object servers, rings
│   │   ├── operations.md         # CLI/API: containers, objects, ACLs, versioning, large objects
│   │   └── internals.md          # Ring building, replication, consistency engine, erasure coding
│   └── shared-filesystem/
│       ├── README.md             # Manila overview
│       ├── architecture.md
│       ├── operations.md
│       └── internals.md
├── orchestration/
│   ├── heat/
│   │   ├── README.md             # Heat overview
│   │   ├── architecture.md       # Heat engine, API, resource types, convergence
│   │   ├── operations.md         # CLI/API: stacks, templates, environments, parameters
│   │   └── internals.md          # Resource plugins, dependency graph, convergence engine
│   └── magnum/
│       ├── README.md             # Magnum overview
│       ├── architecture.md
│       ├── operations.md
│       └── internals.md
├── load-balancing/
│   ├── README.md                 # Octavia overview
│   ├── architecture.md           # Amphora, provider drivers, health manager
│   ├── operations.md             # CLI/API: LBs, listeners, pools, members, health monitors
│   └── internals.md              # Amphora lifecycle, failover, provider framework
├── dns/
│   ├── README.md                 # Designate overview
│   ├── architecture.md           # API, central, worker, producer, mini-DNS, sink
│   ├── operations.md             # CLI/API: zones, recordsets, PTR, reverse DNS
│   └── internals.md              # Backend drivers (BIND9, PowerDNS), pool management
├── bare-metal/
│   ├── README.md                 # Ironic overview
│   ├── architecture.md           # Conductor, drivers, deploy interfaces, cleaning
│   ├── operations.md             # CLI/API: nodes, ports, provisioning, inspection
│   └── internals.md              # Driver interfaces, deploy steps, BIOS/RAID config
├── security/
│   ├── README.md                 # Barbican overview + cross-cutting security
│   ├── architecture.md           # Barbican: API, worker, HSM plugins
│   ├── operations.md             # CLI/API: secrets, containers, orders, CAs
│   ├── internals.md              # Crypto plugins, certificate management
│   └── hardening.md              # Cross-service: TLS everywhere, policy.yaml, RBAC patterns
├── dashboard/
│   ├── README.md                 # Dashboard overview — Horizon vs Skyline comparison
│   ├── horizon/
│   │   ├── README.md
│   │   ├── operations.md         # Configuration, customization, panel management
│   │   └── internals.md          # Django architecture, extending panels, themes
│   └── skyline/
│       ├── README.md
│       ├── operations.md         # Configuration, deployment, API console
│       └── internals.md          # Vue.js frontend, Skyline-apiserver, Nginx
├── operations/
│   ├── health-check/
│   │   └── README.md             # Cross-service health verification
│   ├── upgrades/
│   │   ├── README.md             # Upgrade strategy overview + SLURP explanation
│   │   ├── 2025.1-to-2025.2.md
│   │   ├── 2025.1-to-2026.1.md  # SLURP upgrade path
│   │   └── 2025.2-to-2026.1.md
│   ├── backup-restore/
│   │   └── README.md             # Database + config backup strategies
│   └── capacity-planning/
│       └── README.md             # Quota management, resource planning
├── diagnose/
│   └── README.md                 # Symptom-based troubleshooting decision trees
└── reference/
    ├── decision-guides/
    │   ├── README.md
    │   ├── networking.md          # OVS vs OVN vs other backends
    │   ├── storage-block.md       # Cinder backend selection
    │   ├── storage-object.md      # Swift vs Ceph RadosGW
    │   ├── dashboard.md           # Horizon vs Skyline
    │   └── deployment-tools.md    # Kolla vs OSA vs Charmed vs manual + stack references
    ├── compatibility/
    │   └── README.md              # Service version matrix, Python version, DB versions
    └── known-issues/
        ├── README.md
        ├── 2025.1.md
        ├── 2025.2.md
        └── 2026.1.md
```

~80 files across 25 skills.

### Per-Service Template

Each service follows a consistent structure:

- **README.md** — overview, when to read, routing to sub-files
- **architecture.md** — components, how they interact, data flow
- **operations.md** — CLI/API commands, common workflows, quotas
- **internals.md** — deep implementation details, plugin architecture, extension points
- **Topic-specific deep-dives** where warranted (federation, live-migration, OVS/OVN, etc.)

## Routing Table

| Operator Need | Skill | Entry Point |
|---|---|---|
| Understand OpenStack architecture | architecture | `skills/foundation/architecture/` |
| Learn about deployment methods | deployment-overview | `skills/foundation/deployment-overview/` |
| Understand common config patterns | configuration | `skills/foundation/configuration/` |
| Manage users, projects, auth, federation | identity | `skills/identity/` |
| Launch and manage instances | compute | `skills/compute/` |
| Create and manage networks | networking | `skills/networking/` |
| Manage block storage volumes | block-storage | `skills/storage/block/` |
| Manage images | image | `skills/storage/image/` |
| Manage object storage | object-storage | `skills/storage/object/` |
| Manage shared filesystems | shared-filesystem | `skills/storage/shared-filesystem/` |
| Orchestrate with Heat templates | heat | `skills/orchestration/heat/` |
| Manage Kubernetes clusters | magnum | `skills/orchestration/magnum/` |
| Configure load balancers | load-balancing | `skills/load-balancing/` |
| Manage DNS zones and records | dns | `skills/dns/` |
| Provision bare metal nodes | bare-metal | `skills/bare-metal/` |
| Manage secrets and certificates | security | `skills/security/` |
| Harden the cloud | security-hardening | `skills/security/` |
| Configure the web dashboard | dashboard | `skills/dashboard/` |
| Check cloud health | health-check | `skills/operations/health-check/` |
| Upgrade OpenStack releases | upgrades | `skills/operations/upgrades/` |
| Backup and restore | backup-restore | `skills/operations/backup-restore/` |
| Plan capacity and quotas | capacity-planning | `skills/operations/capacity-planning/` |
| Troubleshoot a problem | diagnose | `skills/diagnose/` |
| Choose between component options | decision-guides | `skills/reference/decision-guides/` |
| Check version compatibility | compatibility | `skills/reference/compatibility/` |
| Look up known bugs and workarounds | known-issues | `skills/reference/known-issues/` |

## Workflows

### New Deployment

1. Understand architecture → `skills/foundation/architecture/`
2. Review deployment methods → `skills/foundation/deployment-overview/`
3. Make decisions (networking backend, storage backend, dashboard) → `skills/reference/decision-guides/`
4. Check known issues for target version → `skills/reference/known-issues/`
5. Deploy Keystone first → `skills/identity/`
6. Deploy core services (Nova, Neutron, Glance, Cinder, Placement)
7. Deploy additional services as needed
8. Verify health → `skills/operations/health-check/`

### Existing Deployment

- Something is broken → `skills/diagnose/`
- Manage a specific service → jump to that service's skill directly
- Upgrade → `skills/operations/upgrades/`
- Check health → `skills/operations/health-check/`

### Developer Building on OpenStack

1. Understand architecture → `skills/foundation/architecture/`
2. Jump to specific service `operations.md` for CLI/API usage
3. Read `internals.md` for extension points and plugin development

## stack.yaml

```yaml
name: openstack-core
namespace: agentic-stacks
version: "0.1.0"
description: >
  Comprehensive operational knowledge for all major OpenStack services.
  Covers architecture, deployment, CLI/API operations, service internals,
  and day-two management across OpenStack 2025.1 (Epoxy) through 2026.1.
  Targets both operators managing production clouds and developers building
  on OpenStack APIs.

repository: https://github.com/agentic-stacks/openstack-core

target:
  software: openstack
  versions:
    - "2025.1"
    - "2025.2"
    - "2026.1"

skills:
  - name: architecture
    entry: skills/foundation/architecture
    description: "Overall OpenStack architecture, message bus, DB, API patterns"
  - name: deployment-overview
    entry: skills/foundation/deployment-overview
    description: "Deployment methods overview with references to deployment stacks"
  - name: configuration
    entry: skills/foundation/configuration
    description: "Common config patterns: oslo.config, paste.ini, policy.yaml"
  - name: identity
    entry: skills/identity
    description: "Keystone: authentication, authorization, federation, catalog"
  - name: compute
    entry: skills/compute
    description: "Nova + Placement: instances, scheduling, migrations, resource tracking"
  - name: networking
    entry: skills/networking
    description: "Neutron: networks, routers, security groups, ML2, OVS, OVN"
  - name: block-storage
    entry: skills/storage/block
    description: "Cinder: volumes, snapshots, backups, multi-backend, replication"
  - name: image
    entry: skills/storage/image
    description: "Glance: image management, stores, import workflows"
  - name: object-storage
    entry: skills/storage/object
    description: "Swift: containers, objects, rings, replication, erasure coding"
  - name: shared-filesystem
    entry: skills/storage/shared-filesystem
    description: "Manila: shared filesystem management, share types, drivers"
  - name: heat
    entry: skills/orchestration/heat
    description: "Heat: stack orchestration, templates, environments, convergence"
  - name: magnum
    entry: skills/orchestration/magnum
    description: "Magnum: Kubernetes cluster provisioning on OpenStack"
  - name: load-balancing
    entry: skills/load-balancing
    description: "Octavia: load balancers, listeners, pools, health monitors, amphora"
  - name: dns
    entry: skills/dns
    description: "Designate: DNS zones, recordsets, reverse DNS, backend drivers"
  - name: bare-metal
    entry: skills/bare-metal
    description: "Ironic: bare metal provisioning, inspection, cleaning, drivers"
  - name: security
    entry: skills/security
    description: "Barbican + cross-cutting: secrets, certificates, TLS, RBAC, hardening"
  - name: dashboard
    entry: skills/dashboard
    description: "Horizon and Skyline: web dashboards, configuration, customization"
  - name: health-check
    entry: skills/operations/health-check
    description: "Cross-service health verification"
  - name: upgrades
    entry: skills/operations/upgrades
    description: "Release upgrades including SLURP paths (2025.1 → 2026.1)"
  - name: backup-restore
    entry: skills/operations/backup-restore
    description: "Database and config backup strategies"
  - name: capacity-planning
    entry: skills/operations/capacity-planning
    description: "Quota management and resource planning"
  - name: diagnose
    entry: skills/diagnose
    description: "Symptom-based troubleshooting decision trees"
  - name: decision-guides
    entry: skills/reference/decision-guides
    description: "Choose between networking, storage, dashboard, and deployment options"
  - name: compatibility
    entry: skills/reference/compatibility
    description: "Service version matrix, Python versions, DB compatibility"
  - name: known-issues
    entry: skills/reference/known-issues
    description: "Version-specific bugs and workarounds"

project:
  structure:
    - clouds.yaml
    - openrc
    - templates/
    - scripts/

requires:
  tools:
    - name: openstack
      description: "OpenStack unified CLI client"
    - name: python-openstackclient
      description: "Python OpenStack client library"

depends_on: []
```

## Deployment Method References

This stack does not cover deployment toolkits in depth. Instead, it references dedicated stacks:

- **kolla-ansible** → `openstack-kolla` agentic stack
- **DevStack** — development/testing only, reference official docs
- **OpenStack-Ansible (OSA)** — future agentic stack candidate
- **TripleO / Director** — deprecated upstream, mention for legacy awareness
- **Kayobe** — future agentic stack candidate
- **Charmed OpenStack (Juju)** — future agentic stack candidate

The `skills/foundation/deployment-overview/` skill covers what each method is, when to use it, and where to find the dedicated stack or documentation.

## Composability

- **Depends on:** nothing (standalone)
- **Pairs well with:** `openstack-kolla` (deployment), future deployment stacks
- **Domain boundary:** this stack owns service knowledge; deployment stacks own the "how to install" workflows
- **No conflicting outputs:** this stack generates no config files in the operator project beyond `clouds.yaml` and `openrc`

## Expected Operator Project Structure

```
my-openstack/
├── clouds.yaml              # OpenStack client auth config
├── openrc                   # Shell-based auth (source openrc)
├── templates/               # Heat templates, environment files
├── scripts/                 # Operational scripts
├── CLAUDE.md                # Operator's own context
├── stacks.lock
└── .stacks/
    └── openstack-core/      # This stack
```
