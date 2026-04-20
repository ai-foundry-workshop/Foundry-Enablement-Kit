# Module 06 — Checkpoint Questions and Answer Key

## How to Use
1. Answer these questions at the **end of each module** before proceeding.
2. Verify by checking your own screen against the expected outcomes.
3. If you cannot answer 2 or more questions in a module, revisit the steps before moving on.

---

## Module 00 — Access & Permissions

**Q1: What two RBAC flows are needed in this workshop, and which identity gets the role in each?**

**Expected Answer:**
| Flow | Role Assigned On | Identity Getting Role | Purpose |
|---|---|---|---|
| Flow 1 | Azure OpenAI / AI Services | **AI Search** (system MI) | Search calls OpenAI for embeddings during indexing |
| Flow 2 | Azure AI Search | **Foundry Project** (system MI) | Agent queries KB at runtime |

**Q2: What happens if you forget to enable system-assigned managed identity on AI Search?**

**Expected Answer:**
- The AI Search service will NOT appear in any RBAC role-assignment picker dropdown
- The "Select members" list will be empty when you try to assign a role

**Q3: What is the "identity picker trap"?**

**Expected Answer:**
- When assigning roles, the resource type dropdown in the managed identity picker defaults to the wrong type
- For Flow 1, you must select **Search services** (not Foundry)
- For Flow 2, you must select **Foundry** (not Search services)
- Selecting the wrong one completes without error but the wrong identity gets the role

### Live Demonstration
- [ ] Show Azure subscription with Contributor/Owner role visible
- [ ] Show ai.azure.com is accessible

---

## Module 01 — Foundry Project & Models

**Q1: What two models did you deploy and what is each used for?**

**Expected Answer:**
- **gpt-4o** — Chat/completion model for generating answers
- **text-embedding-3-large** — Embedding model for converting text to 3072-dimension vectors for semantic search

**Q2: Why must the embedding model dimensions match the search index vector field?**

**Expected Answer:**
- Vector indexing and search queries will fail if dimensions mismatch
- `text-embedding-3-large` = 3072 dimensions — the index must be configured for exactly 3072
- Using a different model (e.g., `text-embedding-ada-002` = 1536) requires a different index configuration

**Q3: After creating AI Search, what three configuration steps did you perform?**

**Expected Answer:**
1. **Set API access control to "Both"** — allows both API keys and RBAC authentication
2. **Enable System Assigned Managed Identity** — creates a service principal for RBAC
3. **Record the Admin API Key** — needed for REST calls and Foundry connections

**Q4: What role did you assign in RBAC Flow 1, and to which identity?**

**Expected Answer:**
- Role: `Cognitive Services OpenAI User`
- Assigned on: Azure OpenAI / AI Services resource
- Identity: AI Search system-assigned managed identity
- Resource type in picker: **Search services**

### Live Demonstration
- [ ] Show both model deployments in Succeeded state
- [ ] Show AI Search with MI enabled (Object ID visible)
- [ ] Show RBAC Flow 1 assignment in IAM

---

## Module 02 — Knowledge Sources

**Q1: Name all five knowledge source types and give one use case for each.**

**Expected Answer:**
| Source | Use Case |
|---|---|
| AI Search Index | Pre-built search pipeline with custom index schema |
| Blob Storage | Quick setup for PDFs/docs stored in Azure |
| Web (Bing) | Real-time web info (benefits providers, external docs) |
| SharePoint Remote | Governance-sensitive data, preserves user permissions |
| SharePoint Indexed | Deep search with hybrid + vector over SharePoint content |

**Q2: What is the key difference between SharePoint Remote and SharePoint Indexed?**

**Expected Answer:**
| Aspect | Remote | Indexed |
|---|---|---|
| Security | User-level M365 permissions preserved | Application-level (bypasses user permissions) |
| Search quality | SharePoint search (keyword) | AI Search (hybrid + vector + semantic) |
| Setup | Simple (sign in with M365) | Complex (app registration, admin consent) |

**Q3: For the SharePoint Indexed path, what mistake do people make when copying the client secret?**

**Expected Answer:**
- They copy the **Secret ID** column instead of the **Value** column
- The Value is only shown once and disappears after navigating away
- Error: `AADSTS7000215: Invalid client secret provided`

### Live Demonstration
- [ ] Show at least 2 knowledge sources in Active state
- [ ] Run a search query on the AI Search index showing document results

---

## Module 03 — Building the Agent

**Q1: What three roles must the Foundry project have on AI Search, and why?**

**Expected Answer:**
| Role | Why |
|---|---|
| `Search Index Data Reader` | Read index content for queries |
| `Search Index Data Contributor` | Write to indexes for Foundry IQ operations |
| `Search Service Contributor` | Manage search service resources |

**Q2: What is the exact error you get if RBAC Flow 2 is missing?**

**Expected Answer:**
```
ErrorAccess denied when connecting to the MCP server at
https://<search>.search.windows.net:443/knowledgebases/.../mcp
while enumerating tools (HTTP 403 Forbidden)
```

**Q3: How should a properly grounded agent respond to "What is the capital of France?"**

**Expected Answer:**
- The agent should refuse: "I could not find this information in the Contoso HR knowledge base."
- It should NOT answer from general knowledge
- If it does answer, the grounding instructions need to be strengthened

**Q4: When an agent has multiple knowledge sources, how does Foundry handle retrieval?**

**Expected Answer:**
- Queries all attached sources in parallel
- Merges and reranks results
- Uses the most relevant content for the answer
- Citations show which source the information came from

### Live Demonstration
- [ ] Show agent responding with grounded answer + citation
- [ ] Show agent refusing an out-of-scope question

---

## Module 04 — Foundry IQ Knowledge Bases

**Q1: What is the difference between a Knowledge Source and a Knowledge Base?**

**Expected Answer:**
- **Knowledge Source** = a single connector to a data store (AI Search, Blob, etc.)
- **Knowledge Base** = a retrieval-ready wrapper around one or more sources + configuration (model, reasoning effort, output mode)
- Agents connect to Knowledge Bases, not Sources

**Q2: Explain the three reasoning effort levels.**

**Expected Answer:**
| Level | Behavior |
|---|---|
| Low | Simple keyword matching, minimal reranking — fast but shallow |
| Medium | Hybrid search + semantic reranking — balanced (recommended default) |
| High | Multi-step query planning, cross-source correlation — thorough but slower |

**Q3: When would you use "Chunks only" output mode instead of "Answer with citations"?**

**Expected Answer:**
- When you want the **agent's own instructions** to control how the answer is formatted
- The KB returns raw retrieved chunks, and the agent synthesizes the final answer
- Useful when you have complex prompt engineering on the agent side

### Live Demonstration
- [ ] Show KB with multiple sources attached and Active status
- [ ] Show a KB test query returning citations

---

## Module 05 — Work IQ

**Q1: What is the key difference between Foundry IQ and Work IQ?**

**Expected Answer:**
| Aspect | Foundry IQ | Work IQ |
|---|---|---|
| Data | Custom documents (AI Search, Blob, SharePoint) | Microsoft 365 data (email, calendar, Teams, OneDrive) |
| Access | Managed identity / application-level | Delegated (user's own M365 permissions) |
| Security | RBAC on Azure resources | M365 security trimming |

**Q2: Can a Work IQ-enabled agent read another user's email?**

**Expected Answer:**
- **No.** Work IQ respects M365 security trimming
- Users only see their own emails, calendars, and files
- Shared content only appears if the user already has permission

**Q3: What licenses are required for Work IQ?**

**Expected Answer:**
- Microsoft 365 E3/E5 (or Business Premium) for M365 data access
- **Microsoft 365 Copilot** license for Work IQ features
- Azure subscription for Foundry (Foundry IQ side)

**Q4: Give an example of a query that uses BOTH Foundry IQ and Work IQ together.**

**Expected Answer:**
- "What is the PTO policy and do I have any meetings about time-off this week?"
- PTO policy → Foundry IQ (company docs)
- Meetings → Work IQ (user's calendar)
- Agent combines both into a single response

### Live Demonstration
- [ ] Show a People query returning org chart info
- [ ] Show a query combining Foundry IQ + Work IQ results
