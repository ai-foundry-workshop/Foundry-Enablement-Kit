# Azure AI Foundry — Customer Enablement Kit

## Disclaimer

This repository contains reference architecture, workshop material, and sample code
intended for learning and enablement purposes only.

This is **not** an officially supported Microsoft product or service.
The content is provided as a reusable blueprint and may require adaptation
before use in any production or customer environment.

All customers are expected to deploy this solution in their **own Azure tenant**,
using their **own subscriptions, data sources, identities, and security controls**.
No Microsoft internal data or customer-specific data is included in this repository.

---

## Build Your First Enterprise AI Agent with Microsoft Foundry

### Purpose
A hands-on workshop that takes you from **zero Foundry experience** to a **working AI agent grounded in enterprise data** — using multiple knowledge source types, Foundry IQ, and Work IQ.

### Audience
Cloud engineers, solution architects, and technical decision-makers who want to evaluate or adopt Azure AI Foundry for enterprise AI use cases. No prior Foundry experience required.

### Scenario
> **Contoso HR** is building an internal assistant for employee onboarding. The assistant must answer questions from company policies (stored in Blob), product FAQs (indexed in AI Search), live web content (Bing grounding), and SharePoint document libraries — all through a single conversational agent.

### Duration
| Format | Time | Modules Covered |
|---|---|---|
| **Full day** | ~6 hours | All modules (00–05) |
| **Half day** | ~3.5 hours | Modules 00–03 (skip Work IQ and advanced KB) |
| **Self-paced** | 1–2 days | All modules at own pace |

### Prerequisites
- Azure subscription with **Contributor** or **Owner** role
- Browser access to **https://ai.azure.com** and **https://portal.azure.com**
- (For SharePoint modules) Microsoft 365 tenant with SharePoint Online
- (For Work IQ module) Microsoft 365 Copilot license

### File Map

| File | Purpose |
|---|---|
| [00-Access-Permissions-Checklist.md](00-Access-Permissions-Checklist.md) | **Start here.** Get all access, roles, and permissions sorted before touching anything else. |
| [01-Create-Foundry-Project-and-Models.md](01-Create-Foundry-Project-and-Models.md) | Create your Foundry project and deploy chat + embedding models. |
| [02-Knowledge-Sources-Deep-Dive.md](02-Knowledge-Sources-Deep-Dive.md) | Connect all 5 knowledge source types: AI Search, Blob, Web, SharePoint Remote, SharePoint Indexed. |
| [03-Build-Your-First-Agent.md](03-Build-Your-First-Agent.md) | Create an agent, attach knowledge, configure grounding, and test. |
| [04-Foundry-IQ-Knowledge-Bases.md](04-Foundry-IQ-Knowledge-Bases.md) | Build multi-source knowledge bases with Foundry IQ — reasoning effort, citation modes, reranking. |
| [05-Extend-to-Work-IQ.md](05-Extend-to-Work-IQ.md) | Extend your agent to Microsoft 365 data via Work IQ — emails, calendar, Teams, OneDrive. |
| [06-Checkpoint-Questions.md](06-Checkpoint-Questions.md) | Module-by-module comprehension checks with answer keys. |
| [07-Troubleshooting-Playbook.md](07-Troubleshooting-Playbook.md) | Every error you will hit, exact root cause, exact portal fix. |
| [workshop-flow.mmd](workshop-flow.mmd) | Visual workshop flow (Mermaid diagram). |
| [sample-data/](sample-data/) | 3 sample documents for consistent indexing and testing. |

### Architecture at a Glance

```
                    ┌──────────────────────────────────────────────┐
                    │           Azure AI Foundry Project           │
                    │                                              │
                    │   ┌──────────┐    ┌───────────────────────┐  │
                    │   │  Agent   │───▶│  Foundry IQ (KB)      │  │
                    │   │ (GPT-4o) │    │  ├─ AI Search Index    │  │
                    │   └──────────┘    │  ├─ Blob Storage       │  │
                    │                   │  ├─ Web (Bing)         │  │
                    │                   │  ├─ SharePoint Remote  │  │
                    │                   │  └─ SharePoint Indexed │  │
                    │                   └───────────────────────┘  │
                    │                                              │
                    │   ┌──────────────────────────────────────┐   │
                    │   │  Work IQ (M365 Graph)                │   │
                    │   │  ├─ Emails   ├─ Calendar             │   │
                    │   │  ├─ Teams    └─ OneDrive             │   │
                    │   └──────────────────────────────────────┘   │
                    └──────────────────────────────────────────────┘
```

### Delivery Rhythm (Per Module)
1. **Explain** — 5 min overview of concepts
2. **Demo** — 5–10 min walkthrough
3. **Build** — 15–25 min hands-on following the guide
4. **Checkpoint** — 5 min verification questions

### How to Start
**Go to [00-Access-Permissions-Checklist.md](00-Access-Permissions-Checklist.md) first.** Nothing else works until access is sorted.
