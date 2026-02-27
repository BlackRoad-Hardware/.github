# BlackRoad Architecture Overview

> **The Core Thesis:** BlackRoad is a routing company, not an AI company.

---

## Executive Summary

We don't train models or buy GPUs. We connect users to intelligence that already exists (Claude, GPT, Llama, NumPy, legal databases, etc.) through an orchestration layer we own.

**The insight:** Intelligence is already trained. Libraries already exist. The value is in routing requests to the right tool at the right time—not in building another brain.

---

## Infrastructure (~$40/month recurring)

| Layer | Service | Role |
|-------|---------|------|
| Edge/CDN | Cloudflare | Handles millions of connections, DNS, DDoS |
| CRM/Data | Salesforce (Free Dev Edition) | Customer data, 15K API calls/day |
| Code/CI | GitHub Enterprise | 15 organizations, deployment |
| Mesh Network | Tailscale | Private encrypted connections between nodes |
| Cloud Node | Digital Ocean (Shellfish) | Internet-facing server |

---

## Hardware (Owned, Not Rented)

A Raspberry Pi cluster running specialized roles:

| Node | Hardware | Role |
|------|----------|------|
| **lucidia** | Pi 5 + Pironman + Hailo-8 | Salesforce sync, RoadChain/Bitcoin |
| **octavia** | Pi 5 + Pironman + Hailo-8 | AI routing decisions (26 TOPS), 3D printing |
| **aria** | Pi 5 | Agent orchestration, Cloudflare Workers |
| **alice** | Pi 400 | Kubernetes + VPN hub (mesh root) |
| **shellfish** | Digital Ocean droplet | Public-facing gateway |

Plus dev machines (Mac = "cecilia", iPhone = "arcadia") and edge devices (ESP32s, LoRa modules for future deployment).

### Local Inference Cluster Specifications

The Pi cluster also serves as a local LLM inference engine, replacing centralized API calls with private, on-premises inference.

| Component | Specification | Role |
|-----------|---------------|------|
| Compute Node | Raspberry Pi 5 (8GB LPDDR4X) | General Purpose Inference and Control |
| Inference Accelerator | Raspberry Pi AI Hat 2 (Hailo 10H, 40 TOPS) | Dedicated INT8 LLM Processing |
| Network Layer | Gigabit Ethernet with PoE+ HAT | Synchronized Node Communication |
| Storage | NVMe SSD (M.2 Interface, 256GB+) | Model Weights and Agent Memory |
| Software Stack | LiteLLM Proxy / Ollama / llama.cpp | API Hosting and Load Balancing |

### Copilot Offloading via LiteLLM Proxy

To bypass external rate limits and keep codebase context local, the system redirects GitHub Copilot traffic to the Pi cluster:

```bash
export GH_COPILOT_OVERRIDE_PROXY_URL="http://raspberrypi.local:4000"
```

The LiteLLM proxy translates requests into an OpenAI-compatible format and distributes them across the cluster using round-robin load balancing.

---

## The Control Plane

```
┌─────────────────────────────────────────────────────────────┐
│              BLACKROAD UNIFIED CONTROL                       │
├─────────────────┬─────────────────┬─────────────────────────┤
│   SALESFORCE    │   CLOUDFLARE    │      GITHUB             │
│   CRM + API     │   Edge + DNS    │    Code + CI            │
└────────┬────────┴────────┬────────┴──────────┬──────────────┘
         │                 │                   │
         └────────────────┬┴───────────────────┘
                          ▼
                    ┌──────────┐
                    │ OPERATOR │  ← We own this
                    └────┬─────┘
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    ┌─────────┐    ┌──────────┐    ┌─────────┐
    │ lucidia │    │ octavia  │    │  aria   │
    │ SF/Chain│    │ Hailo-8  │    │ Agents  │
    └────┬────┘    └────┬─────┘    └────┬────┘
         └───────────────┼───────────────┘
                         ▼
                    ┌─────────┐
                    │  alice  │  ← K8s + VPN hub
                    └─────────┘
```

**Key insight:** The OPERATOR sits between us and all external services. Cloudflare, Salesforce, and GitHub are utilities we command—not landlords we rent from. The control plane lives on hardware we own.

---

## The Routing Pattern

```
[User Request] → [Operator] → [Route to Right Tool] → [Answer]
                     │
                     ├── Physics question? → NumPy/SciPy
                     ├── Language task? → Claude/GPT API
                     ├── Customer lookup? → Salesforce API
                     ├── Legal question? → Legal database
                     └── Fast inference? → Hailo-8 local
```

The agent doesn't need to be smart. It needs to know **who to call.**

---

## The Business Model

| What We Own | What We Don't Need |
|-------------|-------------------|
| Customer relationships | Training models |
| Routing/orchestration logic | GPUs |
| Data and state | Data centers |
| The Operator | The intelligence itself |

When better models come out, we add them to the router. Infrastructure stays the same.

---

## The Math

At $1/user/month:

- 1M users = $12M/year
- 100M users = $1.2B/year
- 1B users = $12B/year

Ceiling: everyone who ever talks to AI.

---

## Organization Structure

BlackRoad operates across 15 specialized GitHub organizations within the [BlackRoad OS Enterprise](https://github.com/enterprises/blackroad-os):

| Organization | Primary Responsibility | Repository Examples |
|--------------|----------------------|---------------------|
| **Blackbox-Enterprises** | Corporate and Enterprise Integrations | blackbox-api, enterprise-bridge |
| **BlackRoad-AI** | Core LLM and Reasoning Engine Development | lucidia-core, blackroad-reasoning |
| **BlackRoad-Archive** | Long-term Data Persistence and Documentation | blackroad-os-docs, history-ledger |
| **BlackRoad-Cloud** | Infrastructure as Code and Orchestration | cloud-orchestrator, railway-deploy |
| **BlackRoad-Education** | Onboarding and Documentation Frameworks | br-help, onboarding-portal |
| **BlackRoad-Foundation** | Governance and Protocol Standards | protocol-specs, governance-rules |
| **BlackRoad-Gov** | Regulatory Compliance and Policy Enforcement | compliance-audit, regulatory-tools |
| **BlackRoad-Hardware** | SBC and IoT Device Management | blackroad-agent-os, pi-firmware |
| **BlackRoad-Interactive** | User Interface and Frontend Systems | blackroad-os-web, interactive-ui |
| **BlackRoad-Labs** | Experimental R&D and Prototyping | experimental-agents, quantum-lab |
| **BlackRoad-Media** | Content Delivery and Public Relations | media-engine, pr-automation |
| **BlackRoad-OS** | Core System Kernel and CLI Development | blackroad-cli, kernel-source |
| **BlackRoad-Security** | Auditing, Cryptography, and Security | security-audit, hash-witnessing |
| **BlackRoad-Studio** | Production Assets and Creative Tooling | lucidia-studio, creative-assets |
| **BlackRoad-Ventures** | Strategic Growth and Ecosystem Funding | tokenomics-api, venture-cap |

Cross-organization access is managed via GitHub Apps (preferred over PATs for their short-lived, granular permissions).

---

## Domain Architecture

All domains are orchestrated via Cloudflare. Cloudflare Tunnels securely expose local Pi nodes to the public internet for local inference without opening internal ports.

| Domain | Use Case | Organization |
|--------|----------|-------------|
| blackboxprogramming.io | Developer Education and APIs | Blackbox-Enterprises |
| blackroad.io | Core Project Landing Page | BlackRoad-OS |
| blackroad.company | Corporate and HR Operations | BlackRoad-Ventures |
| blackroad.me | Personal Agent Identity Nodes | BlackRoad-AI |
| blackroad.network | Distributed Network Interface | BlackRoad-Cloud |
| blackroad.systems | Infrastructure and System Ops | BlackRoad-Cloud |
| blackroadai.com | AI Research and API Hosting | BlackRoad-AI |
| blackroadinc.us | US-based Governance and Legal | BlackRoad-Gov |
| blackroadqi.com | Quantum Intelligence Research | BlackRoad-Labs |
| blackroadquantum.com | Primary Quantum Lab Interface | BlackRoad-Labs |
| lucidia.earth | Memory Layer and Personal AI | BlackRoad-AI |
| lucidia.studio | Creative and Asset Management | BlackRoad-Studio |
| roadchain.io | Blockchain and Witnessing Ledger | BlackRoad-Security |
| roadcoin.io | Tokenomics and Financial Interface | BlackRoad-Ventures |

---

## @blackroad-agents Deca-Layered Scaffold

Every task entered into the system passes through a ten-step scaffold:

1. **Initial Reviewer** — A Layer 6 (Lucidia Core) agent reviews the request for clarity, security compliance, and resource availability, generating a preliminary execution plan.
2. **Task Distributor to Organization** — The task is routed to one of the 15 BlackRoad organizations based on functional domain.
3. **Task Distribution to Team** — Within the organization, the task is refined and assigned to a team. High-risk operations pause for human-in-the-loop (HITL) approval.
4. **Task Update to Project** — The task is recorded on a GitHub Project board and synchronized with Salesforce for enterprise audit trailing.
5. **Task Distribution to Agent** — A specialized autonomous agent is assigned (e.g., `fastapi-coder-agent`, `doctl-infrastructure-agent`) using the Planner-Executor-Reflector pattern.
6. **Task Distribution to Repository** — The agent targets a specific repository and creates an isolated branch following GitHub Flow.
7. **Task Distribution to Device** — If the task requires physical execution (firmware deploy, droplet rebuild), it is routed to the device layer for local hardware offloading.
8. **Task Distribution to Drive** — Artifacts (logs, reports, documentation) are distributed to Google Drive via a Service Account (GSA) with Content Manager role.
9. **Task Distribution to Cloudflare** — Network changes (new tunnels, DNS record updates) are applied to ensure new services are immediately reachable.
10. **Task Distribution to Website Editor** — The presentation layer is updated via AI-driven tools (headless CMS, Wix Harmony, Elementor) for autonomous content management.

---

## @BlackRoadBot Routing Matrix

When `@BlackRoadBot` is mentioned on a GitHub issue or PR, it identifies target platforms from natural language intent:

- **Salesforce** — Maps GitHub events to Salesforce Cases/Tasks via Apex middleware; ingests telemetry via Data Cloud.
- **Hugging Face** — Routes specialized reasoning tasks to Inference Endpoints or locally via Ollama through Cloudflare Tunnels.
- **DigitalOcean** — Manages droplet lifecycle via `doctl` in GitHub Actions (rebuild, scale).
- **Railway** — Deploys ephemeral test environments for feature branches via Railway CLI.
- **Google Drive** — Archives artifacts to shared drives via GSA pattern.

---

## BlackRoad CLI v3 Layered Architecture

| Layer | Name | Role |
|-------|------|------|
| 3 | Agents/System | Autonomous agent lifecycle management and system processes |
| 4 | Deploy/Orchestration | Infrastructure provisioning across cloud and local nodes |
| 5 | Branches/Environments | Ephemeral environment and git-branching logic for agentic tests |
| 6 | Lucidia Core/Memory | Long-term context, state transitions, and simulation data |
| 7 | Orchestration | High-level task distribution powering the @blackroad-agents scaffold |
| 8 | Network/API | REST and GraphQL endpoints for the @BlackRoadBot routing matrix |

Each layer is isolated; a failure in Layer 8 (Network) does not affect state persistence in Layer 6 (Memory).

---

## roadchain Witnessing Architecture

Every state transition (agent commit, bot task routing, deployment) is hashed with SHA-256 and appended to a non-terminating ledger. This creates an immutable record of **what happened** (witnessing) rather than consensus-based proof. The roadchain is hosted at [roadchain.io](https://roadchain.io).

---

## Rate Limit Mitigation

| Provider | Observed Limit | Mitigation |
|----------|---------------|------------|
| GitHub Copilot | RPM / Token Exhaustion | Redirect to local Raspberry Pi LiteLLM proxy |
| Hugging Face Hub | IP-based Rate Limit | Rotate HF_TOKEN or use authenticated SSH keys |
| Google Drive | Individual User Quota | Use Shared Drives with GSA Content Manager role |
| DigitalOcean API | Concurrent Build Limits | Queue tasks via Layer 7 Orchestration |
| Salesforce API | Daily API Request Cap | Batch updates via Data Cloud Streaming Transforms |

If a task fails at any scaffold layer, the system creates a GitHub Issue with Layer 6 (Lucidia Core) logs. A human reviewer or Reflect-and-Retry plugin then assesses and resolves the failure.

---

## Future Scaling: 30k Agents

Active development on `blackroad-30k-agents` targets 30,000 autonomous agents using Kubernetes auto-scaling and self-healing, transitioning from Pi clusters to larger ARM-based data centers.

---

## Implementation Guide

The FastAPI pattern is the starting point:

1. **Expose endpoints** (`/physics/hydrogen`, `/relativity/time-dilation`)
2. **Operator routes** (keyword matching → right function)
3. **Log everything** (JSON audit trail → future ledger)

This is the Operator pattern in miniature. Start with physics, extend to every domain.

---

## Verification

- **Source of Truth:** GitHub (BlackRoad-OS) + Cloudflare
- **Hash Verification:** PS-SHA-∞ (infinite cascade hashing)
- **Authorization:** Alexa's pattern via Claude/ChatGPT

---

*Last Updated: 2026-02-27*
*BlackRoad OS, Inc. - Proprietary and Confidential*
