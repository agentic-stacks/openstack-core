# OpenStack Core Agentic Stack Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a comprehensive agentic stack that teaches AI agents to operate, deploy, and extend all 14 major OpenStack services across releases 2025.1, 2025.2, and 2026.1.

**Architecture:** Service-centric skill hierarchy with shared foundation, operations, diagnose, and reference layers. Each service skill follows a consistent template (README, architecture, operations, internals, plus topic-specific deep-dives). Root files (CLAUDE.md, stack.yaml, README.md) provide agent entry points and routing.

**Tech Stack:** Markdown content authored against official OpenStack documentation. Verified CLI commands, exact YAML config fields, and accurate API references for each service.

---

## Research Protocol

Every task that writes skill content MUST follow this research protocol before writing:

1. **Fetch the official docs index** for the service (e.g., `https://docs.openstack.org/keystone/latest/`, `https://docs.openstack.org/nova/latest/`)
2. **Fetch specific topic pages** relevant to the files being written
3. **Extract exact commands** — copy from docs, do not reconstruct from memory
4. **Verify YAML/config field names** — exact casing and spelling from official docs
5. **Check flag names** — exact flags from CLI `--help` output documented in official docs
6. **Note version-specific behavior** — differences between 2025.1, 2025.2, and 2026.1
7. **Cross-reference release notes** for breaking changes

Official docs base URL: `https://docs.openstack.org/`
Release notes: `https://docs.openstack.org/releasenotes/`

## Per-Service File Template

Every service skill follows this template. Adapt section depth to service complexity.

**README.md:**
```markdown
# [Service Name] ([Project Name])

## Overview
[2-3 sentences: what the service does, why it matters]

## When to Read This
- [Operator scenario 1]
- [Operator scenario 2]
- [Developer scenario]

## Files in This Skill

| File | Covers |
|---|---|
| [architecture.md](architecture.md) | Components, data flow, dependencies |
| [operations.md](operations.md) | CLI/API commands, common workflows |
| [internals.md](internals.md) | Implementation details, plugins, extensions |
| [topic.md](topic.md) | [Deep-dive topic] |

## Quick Reference
[Most common 3-5 commands operators will need]

## Dependencies
[Which other OpenStack services this service requires]
```

**architecture.md:**
```markdown
# [Service Name] Architecture

## Components
[Each daemon/process, what it does, how they communicate]

## Data Flow
[Request lifecycle from API to backend]

## Dependencies
[Message queue, database, other services]

## Deployment Topology
[Typical HA layout, which components go where]
```

**operations.md:**
```markdown
# [Service Name] Operations

## Authentication Setup
[clouds.yaml / openrc config for this service]

## Common Operations
### [Operation Category]
[Exact CLI commands with realistic example values]
[API equivalent where useful]

## Configuration Reference
[Key config file sections, important knobs]

## Quotas and Limits
[How to view/set quotas]

## Maintenance Operations
[Restart, log locations, common admin tasks]
```

**internals.md:**
```markdown
# [Service Name] Internals

## Code Architecture
[Major modules, plugin interfaces, extension points]

## [Plugin/Driver] Framework
[How to write custom plugins/drivers, registration]

## Database Schema
[Key tables, relationships, migrations]

## RPC and Messaging
[How the service uses the message bus]

## Extension Points
[Where developers can extend functionality]
```

---

## Task 1: Scaffold Root Files

**Files:**
- Create: `README.md`
- Create: `CLAUDE.md`
- Create: `stack.yaml`
- Create: `.gitignore`

- [ ] **Step 1: Create .gitignore**

```
# Operator project artifacts
*.pyc
__pycache__/
.env
.venv/
*.egg-info/
```

- [ ] **Step 2: Create stack.yaml**

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
    description: "Release upgrades including SLURP paths (2025.1 -> 2026.1)"
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

- [ ] **Step 3: Create CLAUDE.md**

```markdown
# OpenStack Core — Agentic Stack

## Identity

You are an expert OpenStack operator and architect. You understand the internals of every core OpenStack service, can deploy and configure them from packages or source, operate production clouds via CLI and API, and guide developers building on OpenStack. You work across all major releases (2025.1 Epoxy through 2026.1) and help operators regardless of their host OS.

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

## Expected Operator Project Structure

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

- [ ] **Step 4: Create README.md**

```markdown
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
```

- [ ] **Step 5: Commit scaffold**

```bash
git add .gitignore stack.yaml CLAUDE.md README.md
git commit -m "Scaffold openstack-core agentic stack root files"
```

---

## Task 2: Foundation — Architecture

**Files:**
- Create: `skills/foundation/architecture/README.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/install-guide/latest/get-started-with-openstack.html`
- Fetch `https://docs.openstack.org/security-guide/latest/introduction/architecture.html`
- Fetch `https://docs.openstack.org/oslo.messaging/latest/`

- [ ] **Step 1: Research official architecture docs**

Fetch the URLs above. Extract:
- Complete list of OpenStack services and their roles
- Message bus architecture (RabbitMQ/oslo.messaging patterns)
- Database layer (MySQL/MariaDB, SQLAlchemy, oslo.db)
- API patterns (RESTful, versioned APIs, microversions)
- Authentication flow (Keystone token in every request)
- Service catalog and endpoint discovery
- Shared libraries (oslo.* ecosystem)

- [ ] **Step 2: Write `skills/foundation/architecture/README.md`**

Must cover these sections:
- **Overview**: What OpenStack is, how services relate
- **Service Map**: Table of all 14 services — name, project, port, purpose
- **Shared Infrastructure**: Message queue (RabbitMQ), database (MariaDB/PostgreSQL), cache (Memcached/Redis)
- **API Patterns**: RESTful design, microversions, request/response format
- **Authentication Flow**: How every API call goes through Keystone middleware
- **Service Catalog**: Endpoint types (public, internal, admin), service discovery
- **Oslo Libraries**: oslo.config, oslo.messaging, oslo.db, oslo.policy, oslo.log — what each does
- **Typical Deployment Topology**: Controller, compute, network, storage nodes
- **Inter-Service Communication**: Synchronous (REST API) vs asynchronous (RPC over message bus)

Include a full architecture diagram in text/ASCII showing how services connect through the message bus and Keystone.

- [ ] **Step 3: Validate no placeholders**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/foundation/architecture/
```

- [ ] **Step 4: Commit**

```bash
git add skills/foundation/architecture/
git commit -m "Add foundation architecture skill — OpenStack service map, API patterns, shared infrastructure"
```

---

## Task 3: Foundation — Deployment Overview

**Files:**
- Create: `skills/foundation/deployment-overview/README.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/install-guide/latest/`
- Fetch `https://docs.openstack.org/devstack/latest/`
- Fetch `https://docs.openstack.org/openstack-ansible/latest/`
- Fetch `https://docs.openstack.org/kayobe/latest/`

- [ ] **Step 1: Research deployment methods**

Fetch the URLs above. For each method extract:
- What it is (1-2 sentences)
- When to use it (production, dev, testing)
- Supported platforms
- Maturity and community activity
- Key trade-offs

- [ ] **Step 2: Write `skills/foundation/deployment-overview/README.md`**

Must cover:
- **Manual / Package-Based**: Installing from distro packages or pip, configuring each service individually. Reference official install guide.
- **DevStack**: Development/testing only. Single-machine. Not for production. Link to official docs.
- **Kolla-Ansible**: Containerized deployment via Ansible. Production-grade. Reference `openstack-kolla` agentic stack.
- **OpenStack-Ansible (OSA)**: Ansible-based, bare metal or LXC containers. Production-grade. Link to official docs.
- **TripleO / Director**: Deprecated upstream. Mention for legacy awareness only. Do not recommend for new deployments.
- **Kayobe**: Builds on kolla-ansible with bare metal provisioning via Bifrost. Link to official docs.
- **Charmed OpenStack (Juju)**: Canonical's operator framework. Link to official docs.
- **Comparison Table**: Matrix of all methods — production readiness, complexity, containerized, community activity, supported OSes

End with a clear recommendation: "For new production deployments, kolla-ansible or OSA are the most widely used. See the `openstack-kolla` agentic stack for kolla-ansible guidance."

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/foundation/deployment-overview/
git add skills/foundation/deployment-overview/
git commit -m "Add foundation deployment-overview skill — deployment methods comparison and references"
```

---

## Task 4: Foundation — Configuration

**Files:**
- Create: `skills/foundation/configuration/README.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/oslo.config/latest/`
- Fetch `https://docs.openstack.org/oslo.policy/latest/`
- Fetch `https://docs.openstack.org/keystone/latest/configuration/index.html`

- [ ] **Step 1: Research config patterns**

Extract:
- oslo.config INI file format, sections, option types
- How services discover their config files (`--config-file`, default paths)
- paste.ini / WSGI pipeline configuration
- policy.yaml / policy.json — RBAC policy format, scope concepts
- oslo.log configuration patterns
- Database connection strings
- Message queue connection strings (transport_url)
- Common sections shared across all services: `[DEFAULT]`, `[database]`, `[keystone_authtoken]`, `[oslo_messaging_rabbit]`, `[oslo_concurrency]`

- [ ] **Step 2: Write `skills/foundation/configuration/README.md`**

Must cover:
- **oslo.config**: INI format, section/option syntax, `--config-file` flag, config generators (`oslo-config-generator`)
- **Common Config Sections**: Exact section names and key options for `[DEFAULT]`, `[database]`, `[keystone_authtoken]`, `[oslo_messaging_rabbit]`, `[oslo_concurrency]`, `[cache]`
- **paste.ini / WSGI Pipelines**: What they are, how middleware chains work, Keystone auth middleware
- **policy.yaml**: RBAC rules, scope concepts (system vs project vs domain), `oslopolicy-checker`, generating sample policies
- **Logging**: oslo.log configuration, log levels, format strings, log rotation
- **Service Config File Locations**: Standard paths per-service (`/etc/nova/nova.conf`, `/etc/neutron/neutron.conf`, etc.)
- **Config Validation**: How to validate configs before restarting services

Include realistic config snippets for each section with exact field names.

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/foundation/configuration/
git add skills/foundation/configuration/
git commit -m "Add foundation configuration skill — oslo.config, policy.yaml, common config patterns"
```

---

## Task 5: Identity — Keystone

**Files:**
- Create: `skills/identity/README.md`
- Create: `skills/identity/architecture.md`
- Create: `skills/identity/operations.md`
- Create: `skills/identity/internals.md`
- Create: `skills/identity/federation.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/keystone/latest/`
- Fetch `https://docs.openstack.org/keystone/latest/admin/index.html`
- Fetch `https://docs.openstack.org/keystone/latest/admin/federation/federated_identity.html`
- Fetch `https://docs.openstack.org/api-ref/identity/v3/`
- Fetch release notes for Keystone 2025.1, 2025.2, 2026.1

- [ ] **Step 1: Research Keystone docs**

Extract for each file:
- **architecture.md**: Token providers (fernet, JWS), domains/projects/users/groups/roles model, service catalog, credential storage, application credentials, RBAC enforcement, trust delegation
- **operations.md**: Exact CLI commands for `openstack project create/list/show/delete`, `openstack user create/list/show/delete`, `openstack role add/remove`, `openstack endpoint create/list`, `openstack service create/list`, `openstack token issue`, `openstack application credential create`, `openstack domain create`
- **internals.md**: Auth middleware pipeline, token lifecycle (issue/validate/revoke), identity backends (SQL, LDAP), assignment backends, resource backends, Fernet key rotation mechanism, JWT validation
- **federation.md**: Identity providers, service providers, SAML2 setup, OIDC setup, K2K federation, mapping rules, shadow users

- [ ] **Step 2: Write all 5 files following the per-service template**

README.md routes to the 4 sub-files. Each sub-file follows the template structure from the Research Protocol section above. Include:
- Exact `openstack` CLI commands with realistic values (not `<placeholder>`)
- Exact config file sections (`[keystone_authtoken]`, `[identity]`, `[token]`, `[fernet_tokens]`)
- Fernet key rotation commands: `keystone-manage fernet_setup`, `keystone-manage fernet_rotate`
- Bootstrap command: `keystone-manage bootstrap`
- DB sync: `keystone-manage db_sync`
- federation.md: Complete SAML2 and OIDC setup walkthroughs with realistic example values

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/identity/
git add skills/identity/
git commit -m "Add identity skill — Keystone architecture, operations, internals, federation"
```

---

## Task 6: Compute — Nova + Placement

**Files:**
- Create: `skills/compute/README.md`
- Create: `skills/compute/architecture.md`
- Create: `skills/compute/operations.md`
- Create: `skills/compute/internals.md`
- Create: `skills/compute/placement.md`
- Create: `skills/compute/live-migration.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/nova/latest/`
- Fetch `https://docs.openstack.org/nova/latest/admin/index.html`
- Fetch `https://docs.openstack.org/nova/latest/admin/live-migration-usage.html`
- Fetch `https://docs.openstack.org/nova/latest/reference/scheduler-hints-vs-flavor-extra-specs.html`
- Fetch `https://docs.openstack.org/placement/latest/`
- Fetch `https://docs.openstack.org/api-ref/compute/`
- Fetch release notes for Nova 2025.1, 2025.2, 2026.1

- [ ] **Step 1: Research Nova and Placement docs**

Extract:
- **architecture.md**: nova-api, nova-scheduler, nova-conductor, nova-compute, nova-novncproxy. Cells v2 architecture (cell0, cell1+). How scheduling works (filter → weigh → claim). Interaction with Neutron (port binding), Cinder (volume attach), Glance (image download), Placement (resource claims)
- **operations.md**: `openstack server create/list/show/delete/reboot/resize/rebuild/migrate/evacuate/shelve/unshelve`, `openstack flavor create/list`, `openstack keypair create`, `openstack console url show`, `openstack server add volume/floating ip/security group`, quota commands
- **internals.md**: Scheduler filters (ComputeFilter, RamFilter, DiskFilter, AggregateInstanceExtraSpecsFilter, etc.), weighers, cells v2 DB layout (api DB vs cell DBs), conductor as DB proxy, RPC versioning, compute manager lifecycle hooks
- **placement.md**: Resource providers, resource classes (VCPU, MEMORY_MB, DISK_GB), inventories, allocations, traits, allocation candidates. CLI: `openstack resource provider list`, `openstack allocation candidate list`
- **live-migration.md**: Types (shared storage, block, volume-backed), prerequisites (libvirt, same CPU architecture), tunnelled vs direct, post-copy, monitoring progress, troubleshooting failures

- [ ] **Step 2: Write all 6 files**

Include exact commands, real example IPs/UUIDs, config sections (`[DEFAULT]`, `[scheduler]`, `[filter_scheduler]`, `[compute]`, `[libvirt]`, `[placement]`, `[vnc]`).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/compute/
git add skills/compute/
git commit -m "Add compute skill — Nova architecture, operations, internals, Placement, live migration"
```

---

## Task 7: Networking — Neutron

**Files:**
- Create: `skills/networking/README.md`
- Create: `skills/networking/architecture.md`
- Create: `skills/networking/operations.md`
- Create: `skills/networking/internals.md`
- Create: `skills/networking/ovs.md`
- Create: `skills/networking/ovn.md`
- Create: `skills/networking/advanced.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/neutron/latest/`
- Fetch `https://docs.openstack.org/neutron/latest/admin/index.html`
- Fetch `https://docs.openstack.org/neutron/latest/admin/config-ovs-dpdk.html`
- Fetch `https://docs.openstack.org/neutron/latest/admin/ovn/index.html`
- Fetch `https://docs.openstack.org/api-ref/network/v2/`
- Fetch release notes for Neutron 2025.1, 2025.2, 2026.1

- [ ] **Step 1: Research Neutron docs**

Extract:
- **architecture.md**: neutron-server, ML2 plugin, type drivers (flat, vlan, vxlan, gre, geneve), mechanism drivers (OVS, OVN, Linux Bridge), L3 agent, DHCP agent, metadata agent, L2 agent. Network types: provider, tenant/project. Subnets, ports, routers, floating IPs model.
- **operations.md**: `openstack network create/list/show/delete`, `openstack subnet create`, `openstack router create/add subnet/set --external-gateway`, `openstack floating ip create/associate`, `openstack security group create/rule create`, `openstack port create/list/show`
- **internals.md**: ML2 architecture in depth, type manager, mechanism manager, extension manager. L2 population. Port binding flow. DHCP agent scheduling. Metadata proxy chain. RPC callbacks. DB models.
- **ovs.md**: Open vSwitch integration bridge (br-int), tunnel bridge (br-tun), provider bridge (br-ex). Flow tables. OVS agent. Security group implementation via iptables/conntrack. DPDK support.
- **ovn.md**: OVN architecture (northd, southd, ovn-controller). Integration with Neutron via ML2/OVN. Distributed routing. Native DHCP. Native security groups. OVN metadata agent. Migration from OVS to OVN.
- **advanced.md**: Trunking (VLAN-aware VMs), QoS policies, network segments, BGP dynamic routing, VPNaaS, port forwarding, DNS integration with Designate, RBAC for networks

- [ ] **Step 2: Write all 7 files**

Include exact commands, realistic CIDR values (192.168.1.0/24, 10.0.0.0/8), config sections (`[ml2]`, `[ml2_type_vxlan]`, `[ovs]`, `[ovn]`, `[securitygroup]`, `[l3_agent]`).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/networking/
git add skills/networking/
git commit -m "Add networking skill — Neutron architecture, operations, internals, OVS, OVN, advanced"
```

---

## Task 8: Block Storage — Cinder

**Files:**
- Create: `skills/storage/block/README.md`
- Create: `skills/storage/block/architecture.md`
- Create: `skills/storage/block/operations.md`
- Create: `skills/storage/block/internals.md`
- Create: `skills/storage/block/backends.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/cinder/latest/`
- Fetch `https://docs.openstack.org/cinder/latest/admin/index.html`
- Fetch `https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/index.html`
- Fetch `https://docs.openstack.org/api-ref/block-storage/v3/`

- [ ] **Step 1: Research Cinder docs**

Extract:
- **architecture.md**: cinder-api, cinder-scheduler, cinder-volume, cinder-backup. Volume types, QoS specs, encryption. Interaction with Nova (attach/detach), Glance (volume-backed images).
- **operations.md**: `openstack volume create/list/show/delete/extend`, `openstack volume snapshot create`, `openstack volume backup create/restore`, `openstack volume type create`, QoS commands, volume transfer, volume migration, manage/unmanage
- **internals.md**: Driver interface (create_volume, delete_volume, create_snapshot, etc.), multi-backend config, volume replication (v2.1), active-active HA, scheduler filters/weighers, DB models
- **backends.md**: LVM (reference backend, iSCSI/NFS), Ceph RBD, NFS, Pure Storage, NetApp, Dell EMC — for each: config section, required fields, example config, caveats

- [ ] **Step 2: Write all 5 files**

Include exact commands, config sections (`[DEFAULT]`, `[lvm]`, `[ceph]`, `[key_manager]`), driver config examples.

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/storage/block/
git add skills/storage/block/
git commit -m "Add block storage skill — Cinder architecture, operations, internals, backends"
```

---

## Task 9: Image — Glance

**Files:**
- Create: `skills/storage/image/README.md`
- Create: `skills/storage/image/architecture.md`
- Create: `skills/storage/image/operations.md`
- Create: `skills/storage/image/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/glance/latest/`
- Fetch `https://docs.openstack.org/glance/latest/admin/index.html`
- Fetch `https://docs.openstack.org/api-ref/image/v2/`

- [ ] **Step 1: Research Glance docs**

Extract:
- **architecture.md**: glance-api (registry deprecated since Rocky). Image stores (file, swift, ceph, s3, cinder). Image properties, visibility (public/private/shared/community). Interoperable image import. Metadef namespaces.
- **operations.md**: `openstack image create/list/show/delete/set/save`, image import workflow (`openstack image create --import`), image sharing between projects (`openstack image add project`), property management
- **internals.md**: Store backend interface, image caching, task API, import plugins (web-download, glance-direct, copy-image), location strategy, scrubber (delayed delete)

- [ ] **Step 2: Write all 4 files**

Include exact commands, config sections (`[glance_store]`, `[DEFAULT]`, `[image_format]`, `[task]`).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/storage/image/
git add skills/storage/image/
git commit -m "Add image skill — Glance architecture, operations, internals"
```

---

## Task 10: Object Storage — Swift

**Files:**
- Create: `skills/storage/object/README.md`
- Create: `skills/storage/object/architecture.md`
- Create: `skills/storage/object/operations.md`
- Create: `skills/storage/object/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/swift/latest/`
- Fetch `https://docs.openstack.org/swift/latest/admin_guide.html`
- Fetch `https://docs.openstack.org/swift/latest/overview_ring.html`
- Fetch `https://docs.openstack.org/api-ref/object-store/`

- [ ] **Step 1: Research Swift docs**

Extract:
- **architecture.md**: Proxy server, account/container/object servers. Ring architecture (partitions, replicas, zones, regions). Consistent hashing. Eventually consistent model. Storage policies (replication vs erasure coding).
- **operations.md**: `openstack container create/list/show/delete`, `openstack object create/list/show/save/delete`, large object uploads (SLO, DLO), container ACLs, versioning, temp URLs, form POST, bulk operations, `swift-ring-builder` commands
- **internals.md**: Ring building (`swift-ring-builder create/add/rebalance`), replicator, auditor, updater processes, consistency engine, erasure coding (liberasurecode), storage policies configuration, proxy pipeline (middleware chain), rate limiting, encryption at rest (keymaster)

- [ ] **Step 2: Write all 4 files**

Include exact commands, `swift-ring-builder` examples with realistic device/zone/weight values, config sections (`[swift-hash]`, `[storage-policy:0]`, `[filter:*]`).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/storage/object/
git add skills/storage/object/
git commit -m "Add object storage skill — Swift architecture, operations, internals, rings"
```

---

## Task 11: Shared Filesystems — Manila

**Files:**
- Create: `skills/storage/shared-filesystem/README.md`
- Create: `skills/storage/shared-filesystem/architecture.md`
- Create: `skills/storage/shared-filesystem/operations.md`
- Create: `skills/storage/shared-filesystem/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/manila/latest/`
- Fetch `https://docs.openstack.org/manila/latest/admin/index.html`
- Fetch `https://docs.openstack.org/api-ref/shared-file-system/`

- [ ] **Step 1: Research Manila docs**

Extract:
- **architecture.md**: manila-api, manila-scheduler, manila-share, manila-data. Share types, share networks, share groups. Driver modes: with share servers (DHSS=true) vs without (DHSS=false).
- **operations.md**: `openstack share create/list/show/delete/extend/shrink`, `openstack share type create`, `openstack share network create`, `openstack share access create`, share snapshots, share groups, share migration, share replication
- **internals.md**: Driver interface, share server lifecycle, share network management, data copy service, scheduler filters, DB models, share migration workflow

- [ ] **Step 2: Write all 4 files**

Include exact commands, config sections (`[DEFAULT]`, `[generic]`, `[cephfs]`, `[netapp]`).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/storage/shared-filesystem/
git add skills/storage/shared-filesystem/
git commit -m "Add shared filesystem skill — Manila architecture, operations, internals"
```

---

## Task 12: Orchestration — Heat

**Files:**
- Create: `skills/orchestration/heat/README.md`
- Create: `skills/orchestration/heat/architecture.md`
- Create: `skills/orchestration/heat/operations.md`
- Create: `skills/orchestration/heat/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/heat/latest/`
- Fetch `https://docs.openstack.org/heat/latest/template_guide/index.html`
- Fetch `https://docs.openstack.org/api-ref/orchestration/v1/`

- [ ] **Step 1: Research Heat docs**

Extract:
- **architecture.md**: heat-api, heat-api-cfn, heat-engine. HOT template format, resource types, environments, parameters, outputs. Convergence architecture (stack tasks, resource graph). AWS CloudFormation compatibility.
- **operations.md**: `openstack stack create/list/show/delete/update/cancel/suspend/resume/check`, `openstack stack resource list/show/signal`, `openstack stack event list`, `openstack stack template show`, `openstack stack output show`. Complete HOT template examples (server + network + volume).
- **internals.md**: Resource plugin interface, custom resource types, dependency graph resolution, convergence engine (vs legacy stack lifecycle), stack lock, SoftwareConfig/SoftwareDeployment resources, WaitCondition/Handle, autoscaling resources

- [ ] **Step 2: Write all 4 files**

Include complete HOT template examples with `heat_template_version: wallaby` (or latest), exact resource type names (`OS::Nova::Server`, `OS::Neutron::Net`, etc.).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/orchestration/heat/
git add skills/orchestration/heat/
git commit -m "Add orchestration skill — Heat architecture, operations, internals, template examples"
```

---

## Task 13: Orchestration — Magnum

**Files:**
- Create: `skills/orchestration/magnum/README.md`
- Create: `skills/orchestration/magnum/architecture.md`
- Create: `skills/orchestration/magnum/operations.md`
- Create: `skills/orchestration/magnum/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/magnum/latest/`
- Fetch `https://docs.openstack.org/magnum/latest/user/index.html`
- Fetch `https://docs.openstack.org/api-ref/container-infrastructure-management/`

- [ ] **Step 1: Research Magnum docs**

Extract:
- **architecture.md**: magnum-api, magnum-conductor. Cluster templates, clusters. COE drivers (Kubernetes). Heat stack integration. Cluster drivers architecture.
- **operations.md**: `openstack coe cluster template create/list/show/delete`, `openstack coe cluster create/list/show/delete/resize/upgrade`, `openstack coe cluster config`, getting kubeconfig, node group management
- **internals.md**: Cluster driver interface, Heat template generation, Kubernetes cluster lifecycle, node groups, auto-healing, auto-scaling integration, certificate management

- [ ] **Step 2: Write all 4 files**

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/orchestration/magnum/
git add skills/orchestration/magnum/
git commit -m "Add Magnum skill — container orchestration architecture, operations, internals"
```

---

## Task 14: Load Balancing — Octavia

**Files:**
- Create: `skills/load-balancing/README.md`
- Create: `skills/load-balancing/architecture.md`
- Create: `skills/load-balancing/operations.md`
- Create: `skills/load-balancing/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/octavia/latest/`
- Fetch `https://docs.openstack.org/octavia/latest/admin/index.html`
- Fetch `https://docs.openstack.org/api-ref/load-balancer/v2/`

- [ ] **Step 1: Research Octavia docs**

Extract:
- **architecture.md**: octavia-api, octavia-worker, health-manager, housekeeping. Provider drivers (amphora, OVN). Amphora architecture (HAProxy inside a VM). Spare pool. Anti-affinity. Flavors and flavor profiles.
- **operations.md**: `openstack loadbalancer create/list/show/delete`, `openstack loadbalancer listener create`, `openstack loadbalancer pool create`, `openstack loadbalancer member create`, `openstack loadbalancer healthmonitor create`, `openstack loadbalancer l7policy create/l7rule create`, failover, stats
- **internals.md**: Amphora lifecycle (build, configure, failover, delete), amphora driver interface, provider driver framework, health monitoring (heartbeat UDP), flow-based task execution (Taskflow), database models, certificate management for TLS termination

- [ ] **Step 2: Write all 4 files**

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/load-balancing/
git add skills/load-balancing/
git commit -m "Add load balancing skill — Octavia architecture, operations, internals, amphora"
```

---

## Task 15: DNS — Designate

**Files:**
- Create: `skills/dns/README.md`
- Create: `skills/dns/architecture.md`
- Create: `skills/dns/operations.md`
- Create: `skills/dns/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/designate/latest/`
- Fetch `https://docs.openstack.org/designate/latest/admin/index.html`
- Fetch `https://docs.openstack.org/api-ref/dns/`

- [ ] **Step 1: Research Designate docs**

Extract:
- **architecture.md**: designate-api, designate-central, designate-worker, designate-producer, designate-mdns, designate-sink. Zones (primary, secondary), recordsets, DNS records. Integration with Neutron (auto-create PTR records, FloatingIP DNS).
- **operations.md**: `openstack zone create/list/show/delete/set`, `openstack recordset create/list/show/delete/set`, PTR records, zone transfer, zone import/export, TSIG keys, reverse DNS for floating IPs
- **internals.md**: Backend driver framework (BIND9, PowerDNS, Infoblox), pool configuration, zone management (serial number, SOA), sink (event notification handler for auto-creating DNS records), mini-DNS (internal DNS for zone transfers)

- [ ] **Step 2: Write all 4 files**

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/dns/
git add skills/dns/
git commit -m "Add DNS skill — Designate architecture, operations, internals, backend drivers"
```

---

## Task 16: Bare Metal — Ironic

**Files:**
- Create: `skills/bare-metal/README.md`
- Create: `skills/bare-metal/architecture.md`
- Create: `skills/bare-metal/operations.md`
- Create: `skills/bare-metal/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/ironic/latest/`
- Fetch `https://docs.openstack.org/ironic/latest/admin/index.html`
- Fetch `https://docs.openstack.org/api-ref/baremetal/`
- Fetch `https://docs.openstack.org/ironic-inspector/latest/`

- [ ] **Step 1: Research Ironic docs**

Extract:
- **architecture.md**: ironic-api, ironic-conductor. Hardware types (ipmi, redfish, idrac, ilo, etc.). Deploy interfaces (direct, iscsi, ramdisk). Network interfaces (flat, neutron). Inspection (ironic-inspector or in-band). Cleaning (auto, manual). Provisioning state machine.
- **operations.md**: `openstack baremetal node create/list/show/set/delete`, `openstack baremetal node manage/provide/deploy/undeploy/clean/inspect`, `openstack baremetal port create`, `openstack baremetal driver list`, node enrollment workflow, introspection workflow, BIOS/RAID configuration
- **internals.md**: Driver interface composition (power, management, deploy, boot, inspect, BIOS, RAID, vendor), conductor heartbeat, hash ring for node distribution, deploy steps, clean steps, provisioning state machine transitions, IPA (ironic-python-agent)

- [ ] **Step 2: Write all 4 files**

Include the complete state machine diagram (enroll → verifying → manageable → available → active → etc.).

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/bare-metal/
git add skills/bare-metal/
git commit -m "Add bare metal skill — Ironic architecture, operations, internals, drivers"
```

---

## Task 17: Security — Barbican + Hardening

**Files:**
- Create: `skills/security/README.md`
- Create: `skills/security/architecture.md`
- Create: `skills/security/operations.md`
- Create: `skills/security/internals.md`
- Create: `skills/security/hardening.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/barbican/latest/`
- Fetch `https://docs.openstack.org/barbican/latest/admin/index.html`
- Fetch `https://docs.openstack.org/api-ref/key-manager/`
- Fetch `https://docs.openstack.org/security-guide/latest/`

- [ ] **Step 1: Research Barbican and security docs**

Extract:
- **architecture.md**: barbican-api, barbican-worker, barbican-keystone-listener. Secrets, containers, orders, CAs, transport keys. Secret store plugins (PKCS#11, KMIP, Vault, simple crypto).
- **operations.md**: `openstack secret store/list/get/delete`, `openstack secret order create`, `openstack secret container create`, `openstack ca list`, certificate management workflow
- **internals.md**: Secret store plugin interface, crypto plugin interface, certificate plugin interface, transport key wrapping, secret metadata, per-project quotas, DB models
- **hardening.md**: TLS everywhere (internal API endpoints), oslo.policy scope enforcement, service user accounts (least privilege), Keystone federation for centralized auth, Barbican integration with other services (Nova encrypted volumes, Octavia TLS termination, Swift at-rest encryption), security groups best practices, network segmentation, audit logging

- [ ] **Step 2: Write all 5 files**

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/security/
git add skills/security/
git commit -m "Add security skill — Barbican architecture, operations, internals, hardening guide"
```

---

## Task 18: Dashboard — Horizon + Skyline

**Files:**
- Create: `skills/dashboard/README.md`
- Create: `skills/dashboard/horizon/README.md`
- Create: `skills/dashboard/horizon/operations.md`
- Create: `skills/dashboard/horizon/internals.md`
- Create: `skills/dashboard/skyline/README.md`
- Create: `skills/dashboard/skyline/operations.md`
- Create: `skills/dashboard/skyline/internals.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/horizon/latest/`
- Fetch `https://docs.openstack.org/horizon/latest/admin/index.html`
- Fetch `https://docs.openstack.org/skyline-apiserver/latest/`
- Fetch `https://docs.openstack.org/skyline-console/latest/`
- Fetch `https://opendev.org/openstack/skyline-apiserver`

- [ ] **Step 1: Research Horizon and Skyline docs**

Extract:
- **dashboard/README.md**: Comparison table — Horizon (mature, Django, full feature parity) vs Skyline (modern, Vue.js, lighter, newer)
- **horizon/operations.md**: `local_settings.py` configuration, enabling/disabling panels, session backend, caching, multi-domain support, branding/theming, SSO configuration
- **horizon/internals.md**: Django architecture, panel/dashboard/panel group registration, custom panels, Angular (legacy) vs Django templates, horizon plugin system (`enabled/` directory)
- **skyline/operations.md**: `skyline.yaml` configuration, deployment (container or systemd), Nginx reverse proxy setup, Keystone integration, custom settings
- **skyline/internals.md**: Vue.js 3 frontend (skyline-console), FastAPI backend (skyline-apiserver), architecture (frontend → apiserver → OpenStack APIs), extending with custom pages

- [ ] **Step 2: Write all 7 files**

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/dashboard/
git add skills/dashboard/
git commit -m "Add dashboard skill — Horizon and Skyline architecture, operations, internals"
```

---

## Task 19: Operations — Health Check

**Files:**
- Create: `skills/operations/health-check/README.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/nova/latest/admin/manage-the-cloud.html`
- Fetch `https://docs.openstack.org/neutron/latest/admin/config-service-subnets.html`

- [ ] **Step 1: Write `skills/operations/health-check/README.md`**

Must cover a complete health check procedure:
- **Service Status**: Check each service is running (`openstack compute service list`, `openstack network agent list`, `openstack volume service list`, `openstack loadbalancer provider list`)
- **Endpoint Verification**: `openstack endpoint list`, `openstack catalog list`, verify each endpoint responds
- **Keystone**: Token issuance test (`openstack token issue`)
- **Nova**: Hypervisor status (`openstack hypervisor list`), compute service status, verify instance launch
- **Neutron**: Agent status, DHCP agent network hosting, L3 agent router hosting, verify network connectivity
- **Cinder**: Volume service status, verify volume create/attach/delete cycle
- **Glance**: Verify image list and download
- **Database**: Connection check for each service database
- **Message Queue**: RabbitMQ queue status, unacknowledged messages
- **Quick Health Script**: A combined bash script that checks all services in sequence

- [ ] **Step 2: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/operations/health-check/
git add skills/operations/health-check/
git commit -m "Add health-check skill — cross-service health verification procedure"
```

---

## Task 20: Operations — Upgrades

**Files:**
- Create: `skills/operations/upgrades/README.md`
- Create: `skills/operations/upgrades/2025.1-to-2025.2.md`
- Create: `skills/operations/upgrades/2025.1-to-2026.1.md`
- Create: `skills/operations/upgrades/2025.2-to-2026.1.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/nova/latest/admin/upgrades.html`
- Fetch `https://docs.openstack.org/keystone/latest/admin/upgrades.html`
- Fetch `https://governance.openstack.org/tc/resolutions/20220210-release-cadence-adjustment.html` (SLURP resolution)
- Fetch release notes for each service across 2025.1, 2025.2, 2026.1

- [ ] **Step 1: Research upgrade procedures**

Extract:
- SLURP (Skip Level Upgrade Release Process) explanation
- General upgrade procedure: backup DBs → stop services → update packages → db_sync → start services → verify
- Rolling upgrade support per service
- Online data migrations (`nova-manage db online_data_migrations`, etc.)
- Deprecation warnings to address before upgrading
- Breaking changes per version

- [ ] **Step 2: Write README.md**

Cover:
- **SLURP Explained**: What SLURP is, which releases are SLURP (2025.1 Epoxy), how it enables 2025.1 → 2026.1 direct upgrade
- **General Upgrade Strategy**: Pre-upgrade checklist, backup procedure, service-by-service upgrade order (infrastructure first, then Keystone, then compute/network/storage), post-upgrade validation
- **Rolling Upgrades**: Which services support rolling upgrades, RPC version pinning, online data migrations
- **Upgrade Order**: Keystone → Glance → Cinder → Neutron → Nova → (remaining services)

- [ ] **Step 3: Write version-specific upgrade files**

Each file covers:
- **Pre-upgrade**: Deprecations to address, known issues to check, backup commands
- **Breaking Changes**: Config options removed/renamed, API changes, behavior changes
- **Per-Service Steps**: Exact `db_sync` and `db online_data_migrations` commands for each service
- **Post-upgrade**: Verification steps, config cleanup
- **2025.1-to-2026.1.md** must note this is the SLURP path and cover any additional considerations for skipping 2025.2

- [ ] **Step 4: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/operations/upgrades/
git add skills/operations/upgrades/
git commit -m "Add upgrades skill — SLURP explanation, version-specific upgrade guides"
```

---

## Task 21: Operations — Backup/Restore

**Files:**
- Create: `skills/operations/backup-restore/README.md`

**Depends on:** Task 1

- [ ] **Step 1: Write `skills/operations/backup-restore/README.md`**

Must cover:
- **Database Backup**: `mysqldump` / `mariadb-dump` per-service databases, consistent backup across services, backup scheduling
- **Configuration Backup**: `/etc/<service>/` directories, what to include/exclude
- **Keystone Data**: Fernet keys backup (`/etc/keystone/fernet-keys/`), credential keys
- **Cinder Volumes**: `openstack volume backup create`, backup drivers (swift, NFS, Ceph, GCS)
- **Glance Images**: Image data backup strategy (depends on backend)
- **Swift Rings**: Ring file backup (critical — lost rings = lost data access)
- **Secrets**: Barbican database backup (encrypted secrets), master key backup
- **Restore Procedures**: Per-component restore steps, order of restoration
- **Disaster Recovery**: Full-cloud recovery procedure, minimum required backups

- [ ] **Step 2: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/operations/backup-restore/
git add skills/operations/backup-restore/
git commit -m "Add backup-restore skill — database, config, and data backup strategies"
```

---

## Task 22: Operations — Capacity Planning

**Files:**
- Create: `skills/operations/capacity-planning/README.md`

**Depends on:** Task 1

- [ ] **Step 1: Write `skills/operations/capacity-planning/README.md`**

Must cover:
- **Quota Management**: `openstack quota show/set` for compute, network, volume quotas per project. Default quotas. Quota classes.
- **Resource Usage**: `openstack usage show`, `openstack hypervisor stats show`, `openstack limits show --absolute`
- **Placement Queries**: `openstack resource provider inventory list/show`, `openstack resource provider usage show`, finding available capacity
- **Overcommit Ratios**: CPU (`cpu_allocation_ratio`), RAM (`ram_allocation_ratio`), disk (`disk_allocation_ratio`) — how to tune, consequences
- **Growth Planning**: Monitoring utilization trends, when to add compute/storage/network nodes
- **Flavor Design**: Sizing flavors for workload types, NUMA topology, CPU pinning, huge pages considerations

- [ ] **Step 2: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/operations/capacity-planning/
git add skills/operations/capacity-planning/
git commit -m "Add capacity-planning skill — quotas, resource usage, overcommit, growth planning"
```

---

## Task 23: Diagnose

**Files:**
- Create: `skills/diagnose/README.md`

**Depends on:** Task 1

- [ ] **Step 1: Write `skills/diagnose/README.md`**

Must cover symptom-based troubleshooting decision trees:

- **Instance won't launch**: Check scheduler logs → check Placement allocations → check compute node capacity → check Neutron port binding → check Glance image availability → check Cinder volume (if boot from volume)
- **Instance has no network**: Check security groups → check port status → check DHCP agent → check L2 agent → check bridge/OVS config → check MTU
- **API returns 401/403**: Check Keystone token expiry → check endpoint catalog → check `keystone_authtoken` config → check policy.yaml
- **API returns 500**: Check service logs → check DB connectivity → check RabbitMQ connectivity → check oslo.config
- **Volume attach fails**: Check Cinder volume status → check Nova instance status → check target protocol (iSCSI/RBD) → check multipath → check Cinder driver logs
- **Slow API responses**: Check DB query performance → check RabbitMQ queue depth → check service worker count → check Keystone token validation cache
- **Service won't start**: Check config file syntax → check DB connectivity → check DB migration status → check log file permissions → check port conflicts
- **RabbitMQ issues**: Queue depth, connection count, memory/disk alarms, cluster partition
- **Database issues**: Connection pool exhaustion, slow queries, replication lag

Each tree: symptom → diagnostic commands → likely causes → resolution steps.

- [ ] **Step 2: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/diagnose/
git add skills/diagnose/
git commit -m "Add diagnose skill — symptom-based troubleshooting decision trees"
```

---

## Task 24: Reference — Decision Guides

**Files:**
- Create: `skills/reference/decision-guides/README.md`
- Create: `skills/reference/decision-guides/networking.md`
- Create: `skills/reference/decision-guides/storage-block.md`
- Create: `skills/reference/decision-guides/storage-object.md`
- Create: `skills/reference/decision-guides/dashboard.md`
- Create: `skills/reference/decision-guides/deployment-tools.md`

**Depends on:** Task 1

- [ ] **Step 1: Write README.md**

Index of all decision guides with one-line descriptions.

- [ ] **Step 2: Write `networking.md`**

OVS vs OVN vs Linux Bridge:

| Feature | OVS | OVN | Linux Bridge |
|---|---|---|---|
| Performance | High | High | Medium |
| Distributed routing | No (L3 agent) | Yes (native) | No |
| Security groups | iptables/conntrack | Native ACLs | iptables |
| DPDK support | Yes | Yes | No |
| Complexity | Medium | Medium-High | Low |
| Community direction | Maintenance | Active development | Legacy |

Recommendation by use case. Migration path (OVS → OVN).

- [ ] **Step 3: Write `storage-block.md`**

LVM vs Ceph RBD vs NFS vs vendor:

| Feature | LVM | Ceph RBD | NFS | Vendor |
|---|---|---|---|---|
| HA | No | Yes | Depends | Yes |
| Performance | Local disk | Distributed | Network | Varies |
| Snapshots | LVM snapshots | Copy-on-write | Backend | Native |
| Replication | No | Built-in | Backend | Yes |
| Complexity | Low | High | Low | Medium |

Recommendation by use case.

- [ ] **Step 4: Write `storage-object.md`**

Swift vs Ceph RadosGW:

| Feature | Swift | Ceph RadosGW |
|---|---|---|
| S3 compatibility | Limited | Full |
| Standalone deployment | Yes | Needs Ceph cluster |
| Erasure coding | Yes | Yes |
| Multi-site | Yes (native) | Yes |
| Best when | Dedicated object storage | Already running Ceph |

- [ ] **Step 5: Write `dashboard.md`**

Horizon vs Skyline:

| Feature | Horizon | Skyline |
|---|---|---|
| Maturity | Very mature | Newer |
| Framework | Django/Python | Vue.js/FastAPI |
| Plugin ecosystem | Large | Growing |
| Performance | Heavier | Lighter/faster |
| Service coverage | Complete | Core services |
| Customization | Extensive | Moderate |

- [ ] **Step 6: Write `deployment-tools.md`**

Comparison matrix of kolla-ansible, OSA, Charmed, Kayobe, DevStack, manual. Reference `openstack-kolla` agentic stack for kolla-ansible. Note TripleO deprecation.

- [ ] **Step 7: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/reference/decision-guides/
git add skills/reference/decision-guides/
git commit -m "Add decision guides — networking, storage, dashboard, deployment tool selection"
```

---

## Task 25: Reference — Compatibility

**Files:**
- Create: `skills/reference/compatibility/README.md`

**Depends on:** Task 1

**Research:**
- Fetch `https://docs.openstack.org/latest/` (release matrix)
- Fetch `https://governance.openstack.org/tc/reference/release-naming.html`

- [ ] **Step 1: Write `skills/reference/compatibility/README.md`**

Must cover:
- **Release Matrix**: Table mapping release name → date → Python version → DB versions → message queue versions
- **Service Version Matrix**: Which service versions ship with each release (all services same release name but verify)
- **Python Compatibility**: Python 3.10, 3.11, 3.12 support per release
- **Database Compatibility**: MariaDB 10.x/11.x, MySQL 8.x, PostgreSQL support status
- **Message Queue**: RabbitMQ version requirements
- **OS Compatibility**: Ubuntu 22.04/24.04, Rocky Linux 9/10, Debian — note which are tested in CI
- **Client Compatibility**: python-openstackclient version requirements
- **SLURP Release Markers**: Which releases are SLURP-capable

- [ ] **Step 2: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/reference/compatibility/
git add skills/reference/compatibility/
git commit -m "Add compatibility skill — release matrix, Python, DB, OS compatibility"
```

---

## Task 26: Reference — Known Issues

**Files:**
- Create: `skills/reference/known-issues/README.md`
- Create: `skills/reference/known-issues/2025.1.md`
- Create: `skills/reference/known-issues/2025.2.md`
- Create: `skills/reference/known-issues/2026.1.md`

**Depends on:** Task 1

**Research:**
- Fetch release notes for all 14 services across all 3 releases
- Check Launchpad/StoryBoard for high-impact bugs

- [ ] **Step 1: Write README.md**

Explain the format:
```markdown
### [Short Description]
**Symptom:** What the operator sees
**Cause:** Why it happens
**Workaround:** Exact steps to fix
**Affected versions:** x through y
**Status:** Open / Fixed in z
```

- [ ] **Step 2: Write version-specific known issues files**

For each version, research and document:
- High-impact bugs across all services
- Upgrade blockers
- Config changes that break existing deployments
- Deprecations that became removals
- Security advisories (OSSAs)

Each entry follows the format from README.md. Include exact commands for workarounds.

- [ ] **Step 3: Validate and commit**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/reference/known-issues/
git add skills/reference/known-issues/
git commit -m "Add known issues — version-specific bugs and workarounds for 2025.1, 2025.2, 2026.1"
```

---

## Task 27: Final Validation

**Depends on:** All previous tasks

- [ ] **Step 1: Verify all skill directories have README.md**

```bash
grep "entry:" stack.yaml | awk '{print $2}' | while read dir; do
  [ -d "$dir" ] && [ -f "$dir/README.md" ] && echo "OK: $dir" || echo "MISSING: $dir"
done
```

- [ ] **Step 2: Verify routing table matches skills**

```bash
grep 'skills/' CLAUDE.md | grep -o 'skills/[^ |`]*' | sed 's|`||g' | sort -u | while read path; do
  path="${path%/}"
  [ -d "$path" ] || [ -f "$path" ] && echo "OK: $path" || echo "BROKEN: $path"
done
```

- [ ] **Step 3: Check for placeholders**

```bash
grep -ri "tbd\|todo\|fixme\|placeholder" skills/ CLAUDE.md stack.yaml README.md
```

- [ ] **Step 4: Verify no empty files**

```bash
find skills/ -name "*.md" -empty
```

- [ ] **Step 5: Count files and verify against spec (~80 expected)**

```bash
find skills/ -name "*.md" | wc -l
find skills/ -name "*.md" | sort
```

- [ ] **Step 6: Final commit if any fixes were needed**

```bash
git add -A
git commit -m "Fix validation issues from final review"
```

---

## Task Dependency Graph

```
Task 1 (scaffold) ──┬── Task 2 (foundation/architecture)
                     ├── Task 3 (foundation/deployment-overview)
                     ├── Task 4 (foundation/configuration)
                     ├── Task 5 (identity/Keystone)
                     ├── Task 6 (compute/Nova+Placement)
                     ├── Task 7 (networking/Neutron)
                     ├── Task 8 (storage/block/Cinder)
                     ├── Task 9 (storage/image/Glance)
                     ├── Task 10 (storage/object/Swift)
                     ├── Task 11 (storage/shared-filesystem/Manila)
                     ├── Task 12 (orchestration/Heat)
                     ├── Task 13 (orchestration/Magnum)
                     ├── Task 14 (load-balancing/Octavia)
                     ├── Task 15 (dns/Designate)
                     ├── Task 16 (bare-metal/Ironic)
                     ├── Task 17 (security/Barbican+hardening)
                     ├── Task 18 (dashboard/Horizon+Skyline)
                     ├── Task 19 (operations/health-check)
                     ├── Task 20 (operations/upgrades)
                     ├── Task 21 (operations/backup-restore)
                     ├── Task 22 (operations/capacity-planning)
                     ├── Task 23 (diagnose)
                     ├── Task 24 (reference/decision-guides)
                     ├── Task 25 (reference/compatibility)
                     └── Task 26 (reference/known-issues)
                                        │
                          All ──────── Task 27 (final validation)
```

**Tasks 2-26 are all independent** and can be executed in parallel after Task 1 completes. Task 27 runs after all others.

## Parallelization Groups

For subagent-driven execution, these groups can run simultaneously:

- **Group A (Foundation):** Tasks 2, 3, 4
- **Group B (Core Services):** Tasks 5, 6, 7
- **Group C (Storage):** Tasks 8, 9, 10, 11
- **Group D (Additional Services):** Tasks 12, 13, 14, 15, 16
- **Group E (Cross-Cutting):** Tasks 17, 18
- **Group F (Operations):** Tasks 19, 20, 21, 22
- **Group G (Reference):** Tasks 23, 24, 25, 26
