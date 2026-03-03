<div align="center">

# TechConsult GmbH
### Secure Client Portal Infrastructure

**Infrastructure Engineering Bootcamp &nbsp;·&nbsp; 8 Sprints &nbsp;·&nbsp; 115 Hours**

[![Status](https://img.shields.io/badge/Status-In_Progress-f59e0b?style=flat-square)](.)
[![Sprint](https://img.shields.io/badge/Current-Sprint_1-4a9eed?style=flat-square)](.)
[![Stack](https://img.shields.io/badge/Stack-KVM_·_Docker_·_Ansible_·_Ollama-8b5cf6?style=flat-square)](.)
[![Host](https://img.shields.io/badge/Host-Fedora_GNOME-22c55e?style=flat-square)](.)

</div>

---

## What This Is

This repository documents the end-to-end design and build of a production-grade infrastructure for a fictional German IT consulting firm. It is not a tutorial walkthrough. It is a working, deployable system built from first principles — version-controlled, documented, and explainable at every layer.

Every architectural decision maps to a numbered principle from the [Engineering Principles Framework](docs/engineering-principles.pdf). Every sprint produces a tangible deliverable committed to this repository. By the end, this infrastructure runs a secure client portal with network segmentation, hardware authentication, full observability, Ansible automation, and a locally hosted AI model — all on a single ThinkPad.

> **"The lab IS the interview."**

---

## Architecture

![Network Topology](diagrams/network-topology.png)

The lab simulates a standard 3-tier production network with a dedicated management plane. Four isolated KVM subnets enforce security boundaries at the network level — not the application level.

| VM | Hostname | IP | Subnet | Role |
|:---|:---|:---|:---|:---|
| VM1 | `tc-bastion` | `10.0.1.10` | Management | SSH jump host · Yubikey FIDO2 |
| VM2 | `tc-proxy` | `10.0.2.10` | DMZ | Nginx · TLS termination |
| VM3 | `tc-app` | `10.0.3.10` | Application | Flask in Docker · Ollama AI |
| VM4 | `tc-db` | `10.0.4.10` | Database | PostgreSQL 16 |
| VM5 | `tc-monitor` | `10.0.1.20` | Management | Prometheus · Grafana |

```
CLIENT  ──HTTPS:443──►  tc-proxy (DMZ)
                             │
                        HTTP:8080
                             │
                             ▼
                        tc-app (Application)
                             │
                       PostgreSQL:5432
                             │
                             ▼
                        tc-db (Database)

Fedora Host ──SSH+Yubikey──► tc-bastion ──ProxyJump──► all VMs
tc-monitor  ──metrics:9100──► all 5 VMs
```

---

## Engineering Principles Applied

All design decisions reference the [Engineering Principles Framework](docs/engineering-principles.pdf) — a set of 60 agnostic infrastructure principles organized across 5 pillars.

| Principle | Implementation in this lab |
|:---|:---|
| **#12 — Default-Deny Networking** | UFW on every VM. Only explicitly documented ports are open. |
| **#14 — Addressing is Architecture** | Subnets and IPs designed and documented before any VM was created. |
| **#18 — Least Privilege Always** | No sudo for VM management. No root SSH. libvirt group for hypervisor ops. |
| **#21 — Segment Everything** | 4 isolated subnets. A compromised web server cannot reach the database. |
| **#22 — Defence-in-Depth** | Yubikey + network segmentation + OS hardening + TLS. Multiple layers. |
| **#27 — If You Can't See It, You Can't Secure It** | Prometheus scrapes all 5 VMs. Grafana provides real-time visibility. |
| **#32 — Infrastructure is Code** | Everything in Git. Ansible rebuilds the full lab from scratch in under 30 minutes. |

---

## Sprints

| # | Sprint | Hours | Status |
|:---:|:---|:---:|:---:|
| 0 | Git + Network Architecture | 12 | `COMPLETED` |
| 1 | Linux + KVM Virtualization | 16 | `IN PROGRESS` |
| 2 | Docker Fundamentals | 15 | `NOT STARTED` |
| 3 | Network Segmentation + Firewalls | 15 | `NOT STARTED` |
| 4 | Full Lab: TechConsult Portal | 22 | `NOT STARTED` |
| 5 | Yubikey Hardware Authentication | 8 | `NOT STARTED` |
| 6 | Ansible Automation | 12 | `NOT STARTED` |
| 7 | AI Infrastructure + Final Docs | 15 | `NOT STARTED` |
| | **Total** | **115** | **8–10 weeks** |

---

## Sprint 0 — Git + Network Architecture
`12 hours` · `COMPLETED`

Established version control and designed the full network topology before touching a single VM. The architecture diagram, subnet plan, and IP assignments were committed to Git first — this is how production infrastructure is built.

**Deliverables**
- Public GitHub repository with clean, descriptive commit history
- [`docs/network-plan.md`](docs/network-plan.md) — subnet design, VM inventory, traffic flows
- [`diagrams/network-topology.png`](diagrams/network-topology.png) — architecture diagram
- Project directory structure for all future sprints

*Engineering Principles: #6 Know the OS · #10 Follow the Packet · #14 Addressing is Architecture*

---

## Sprint 1 — Linux Server Administration + KVM Virtualization
`16 hours` · `IN PROGRESS`

Building the first VM from the command line, establishing SSH key-based access, hardening the server before deploying any services, and producing a professional deployment report with real system metrics.

**Deliverables**
- `practice-vm` — Ubuntu Server 24.04 LTS, headless, SSH-only
- ed25519 SSH key authentication (no passwords)
- UFW default-deny firewall with documented ruleset
- [`sprint-1-server-report.md`](sprint-1-server-report.md) — real metrics, before/after hardening comparison

*Engineering Principles: #6 · #7 Know What Healthy Looks Like · #8 Design for Failure · #9 Repeatable Configs · #23 Reduce Attack Surface*

---

## Sprint 2 — Docker Fundamentals
`15 hours` · `NOT STARTED`

Builds the TechConsult Flask portal application, containerizes it, and wires it to PostgreSQL using Docker Compose. This compose file becomes the prototype deployed to production VMs in Sprint 4.

**Deliverables**
- Custom Docker image of the TechConsult Flask portal
- Working `docker-compose.yml` (Flask + PostgreSQL)
- Documentation of container networking model and lifecycle

*Engineering Principles: #9 · #32 Infrastructure is Code · #37 Prefer Managed Services When Understood*

---

## Sprint 3 — Network Segmentation + Firewalls
`15 hours` · `NOT STARTED`

Creates the four isolated KVM networks, assigns static IPs via netplan, and implements role-specific UFW rulesets. Proves segmentation actually works — blocked paths are tested and confirmed blocked.

**Deliverables**
- 4 isolated KVM virtual networks with static IP assignment
- UFW ruleset per VM with documented business justification for every rule
- Connectivity test report proving segmentation is enforced
- SSH ProxyJump bastion configuration

*Engineering Principles: #10 · #11 Debug in Layers · #12 · #13 Tools Give Visibility · #21*

---

## Sprint 4 — Full Lab Deployment: TechConsult Portal
`22 hours` · `NOT STARTED`

Deploys all five VMs simultaneously and connects the full application stack end-to-end. Nginx terminates TLS and proxies to Flask. Flask queries PostgreSQL. Prometheus scrapes everything.

**Deliverables**
- End-to-end HTTPS portal accessible through Nginx reverse proxy
- PostgreSQL backend storing real application data
- Prometheus + Grafana monitoring dashboards with live metrics from all 5 VMs

*Engineering Principles: #1 Business Over Puzzles · #3 Think in Systems · #7 · #15 Assume Breach · #21 · #27*

---

## Sprint 5 — Yubikey Hardware Authentication
`8 hours` · `NOT STARTED`

Adds physical hardware as an authentication factor. SSH to the bastion host requires a Yubikey touch — a software credential alone is no longer sufficient.

**Deliverables**
- FIDO2 ed25519-sk SSH key pair bound to Yubikey 5C NFC
- Hardware-gated bastion access (physical touch required)
- Security architecture documentation with trust boundary diagrams

*Engineering Principles: #15 · #17 Never Trust Always Verify · #18 · #22 · #24 Identity is the Perimeter*

---

## Sprint 6 — Ansible Automation
`12 hours` · `NOT STARTED`

Converts every manual configuration step into idempotent Ansible playbooks. A single `site.yml` rebuilds the entire lab from bare VMs — proof that all configurations are repeatable.

**Deliverables**
- Full Ansible inventory + role-based playbooks + `group_vars`
- Master `site.yml` orchestrating complete lab rebuild
- Documented full rebuild from bare VMs in under 30 minutes

*Engineering Principles: #9 · #30 The Rule of Two · #31 Standardise Before Automating · #32 · #33 Automation Must Be Safe*

---

## Sprint 7 — AI Infrastructure + Final Documentation
`15 hours` · `NOT STARTED`

Deploys Mistral Nemo 12B via Ollama on the app server — an EU-developed model for GDPR-compliant local AI inference. Adds AI deployment to Ansible automation. Produces final portfolio documentation.

**Deliverables**
- Local AI inference endpoint (Ollama + Mistral Nemo 12B)
- Ansible-automated AI service deployment integrated into `site.yml`
- Portfolio-ready documentation, final architecture diagram, Grafana screenshots

*Engineering Principles: #1 · #32 · #37 · #49 One Evolving Project · #51 Communication is Engineering · #52 Documentation Trinity*

---

## Technology Stack

| Layer | Technology |
|:---|:---|
| **Host** | Fedora GNOME · ThinkPad T14 G1 · Ryzen 7 Pro · 32GB RAM |
| **Virtualization** | KVM / QEMU · libvirt · virsh CLI |
| **Guest OS** | Ubuntu Server 24.04 LTS (headless, SSH-only) |
| **Containers** | Docker · Docker Compose |
| **Web / Proxy** | Nginx (reverse proxy + TLS termination) |
| **Application** | Flask (Python) |
| **Database** | PostgreSQL 16 |
| **Monitoring** | Prometheus · Grafana · node_exporter |
| **Automation** | Ansible |
| **Security** | UFW · ed25519 SSH keys · Yubikey 5C NFC (FIDO2) |
| **AI Runtime** | Ollama · Mistral Nemo 12B |
| **Version Control** | Git · GitHub CLI (`gh`) |

---

## Repository Structure

```
The-Lab/
├── diagrams/
│   └── network-topology.png        # Architecture diagram
├── docs/
│   ├── engineering-principles.pdf  # Engineering Principles Framework
│   └── network-plan.md             # Subnet design and VM inventory
├── ansible/
│   ├── inventory/
│   ├── playbooks/
│   └── group_vars/
├── docker/
├── scripts/
└── sprint-1-server-report.md       # Added each sprint
```

---

## Post-Bootcamp Path

```
NOW      →  CCNA 200-301
             Lab covers ~60-65% of exam content already.
             │
             ▼
PHASE 1  →  Junior Infrastructure Engineer
             International companies / MSPs in Germany
             Bechtle · Computacenter · NTT Data · T-Systems
             │
             ▼
PHASE 2  →  CCNP + 2 years experience
             Gulf markets — Dubai · Abu Dhabi
             │
             ▼
PHASE 3  →  AWS / Azure + CKA
             USA · AI Infrastructure / Platform Engineering
```

---

<div align="center">

*Built on the Engineering Principles Framework.*
*Tools change. Principles don't.*

</div>
