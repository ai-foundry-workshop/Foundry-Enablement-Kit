# Module 04 — Foundry IQ Knowledge Bases

## What You Are Building
A multi-source **Knowledge Base** powered by Foundry IQ — the orchestration layer that connects retrieval to your agent via the MCP protocol. You will configure reasoning effort, citation modes, output formatting, and combine multiple knowledge sources into a single retrieval-ready KB.

## Scenario Context
> Contoso HR wants to unify answers from policies (Blob), FAQs (AI Search), and live benefits info (Bing) into a single knowledge base. The KB should use semantic reranking for high-quality retrieval and always provide document citations.

---

## Understanding the Foundry IQ Stack

```
Agent
  │
  │ asks a question
  ▼
Foundry IQ Knowledge Base
  │
  │ MCP Protocol
  │
  ├── Query Planning (decides which sources to query)
  ├── Retrieval (fetches from all attached sources)
  ├── Reranking (semantic reranking of results)
  └── Synthesis (generates answer with citations using the chat model)
  │
  ▼
Answer + Citations returned to Agent
```

### Key Concepts

| Concept | What It Means |
|---|---|
| **Knowledge Source** | A single connector pointing to a data store (AI Search index, Blob container, etc.) |
| **Knowledge Base** | A retrieval-ready wrapper around one or more sources + settings (model, reasoning, output mode) |
| **Foundry IQ** | The orchestration engine that powers KBs — handles query planning, reranking, synthesis |
| **MCP Protocol** | The open protocol Foundry IQ uses to communicate between agent and KB |

> **Agents connect to Knowledge Bases, not Knowledge Sources.** A source is raw data. A KB adds intelligence.

---

## Step 4.1: Create a Multi-Source Knowledge Base

1. In **https://ai.azure.com** → your project
2. Navigate to **Knowledge** → **Knowledge bases**
3. Click **+ Create knowledge base**
4. Fill in:

   | Field | What to Enter | Notes |
   |---|---|---|
   | **Name** | `hr-unified-kb` | Descriptive name |
   | **Model** | `gpt-4o` | The KB uses this for answer synthesis |
   | **Retrieval reasoning effort** | **Medium** | See comparison table below |
   | **Output mode** | **Answer with citations** | See options below |

5. **Add knowledge sources:**
   - Click **+ Add knowledge source**
   - Select `hr-documents-index` (AI Search) → **Add**
   - Click **+ Add knowledge source** again
   - Select your Blob Storage source → **Add**
   - (Optional) Add the Bing/Web source → **Add**

6. Click **Create**
7. Wait for status: **Active** ✅

---

## Step 4.2: Understanding Reasoning Effort

Reasoning effort controls how much processing Foundry IQ applies to queries.

| Level | What Happens | Best For |
|---|---|---|
| **Low** | Simple keyword matching, minimal reranking | Fast lookups, FAQ-style questions |
| **Medium** | Hybrid search (keyword + vector) + semantic reranking | General-purpose — **recommended default** |
| **High** | Multi-step query planning, deep reasoning, cross-source correlation | Complex questions requiring synthesis from multiple docs |

### When to Change Reasoning Effort

- Start with **Medium** for the workshop
- If answers are too shallow or miss relevant docs → increase to **High**
- If latency is a concern and questions are simple → decrease to **Low**
- You can change this setting anytime without recreating the KB

### Try It: Compare Reasoning Levels

Ask the same question with different reasoning effort settings:

**Question:** "What equipment do I need on my first day and what is the dress code?"

| Reasoning Level | Expected Behavior |
|---|---|
| **Low** | May only find one document (dress code OR equipment, not both) |
| **Medium** | Finds both documents, provides combined answer |
| **High** | Finds both documents, synthesizes a comprehensive answer, may also pull related info (e.g., laptop setup instructions) |

---

## Step 4.3: Understanding Output Modes

| Mode | What the KB Returns | Best For |
|---|---|---|
| **Answer with citations** | Synthesized answer + inline citations to source docs | End-user-facing agents (recommended) |
| **Answer only** | Synthesized answer without source references | Chatbots where citation isn't needed |
| **Chunks only** | Raw retrieved chunks, no synthesis | When the agent (not the KB) should generate the answer |

### When to Use Each

- **Answer with citations** — Default for this workshop. Users can verify information.
- **Chunks only** — Advanced scenario: you want the agent's own prompt engineering to control the final answer, using the KB only for retrieval.
- **Answer only** — Simplest output, useful for casual chatbots.

---

## Step 4.4: Test the Knowledge Base Directly

Before connecting to an agent, test the KB standalone:

1. In **Knowledge** → **Knowledge bases** → click `hr-unified-kb`
2. Find the **Test** or **Query** section
3. Type: `What is the company policy on remote work?`
4. Review the response:
   - Is the answer grounded in your documents?
   - Are citations present (if using "Answer with citations")?
   - Does the answer pull from the correct source?

### Test with Multi-Source Query

```
What is the dress code and what are the latest benefits enrollment deadlines?
```

**Expected behavior with multiple sources:**
- Dress code answer comes from Blob or AI Search (internal docs)
- Benefits deadlines come from Bing/Web source (if attached) or internal docs
- Both are cited separately

---

## Step 4.5: Connect the KB to Your Agent

1. Go to **Agents** → click your `contoso-hr-onboarding` agent
2. In the **Knowledge** or **Tools** section:
   - Remove any directly-attached knowledge sources (from Module 03)
   - Click **Add knowledge base** (or **Add tool** → **Foundry IQ**)
   - Select `hr-unified-kb`
3. **Save**

### Re-Run the Agent Tests

Run the same four tests from Module 03:

| Test | Question | Expected |
|---|---|---|
| In-scope | "What is the dress code?" | Grounded answer + citation |
| Specific | "Steps for day one laptop setup?" | Detailed answer from onboarding guide |
| Out-of-scope | "Capital of France?" | Refusal message |
| Multi-source | "Dress code AND latest benefits deadlines?" | Combined answer from multiple sources |

---

## Step 4.6: Advanced — Multiple Knowledge Bases

You can create specialized KBs for different domains and attach multiple KBs to one agent:

### Example: Domain-Specific KBs

| KB Name | Sources | Reasoning | Use Case |
|---|---|---|---|
| `hr-policies-kb` | AI Search (policies index) | Medium | Internal policy questions |
| `benefits-live-kb` | Bing/Web + SharePoint | High | Benefits with real-time data |
| `it-onboarding-kb` | Blob (IT docs) | Low | Simple IT setup FAQs |

### How the Agent Uses Multiple KBs

When an agent has multiple KBs:
1. Agent receives user query
2. Foundry routes the query to **all attached KBs**
3. Each KB retrieves and synthesizes independently
4. Agent merges and presents the best answer

> **Trade-off:** More KBs = broader coverage but higher latency. For most use cases, one well-configured multi-source KB is better than multiple single-source KBs.

---

## Step 4.7: KB Configuration Cheat Sheet

| Setting | Recommended Default | When to Change |
|---|---|---|
| **Model** | gpt-4o | Use gpt-4o-mini for cost savings on simple queries |
| **Reasoning effort** | Medium | High for complex cross-doc queries; Low for FAQ bots |
| **Output mode** | Answer with citations | Chunks only for custom synthesis; Answer only for casual bots |
| **Sources** | 1–3 focused sources | More sources = broader but slower retrieval |

---

## Verification — You Are Done When

- [ ] Knowledge base `hr-unified-kb` shows **Active** with multiple sources
- [ ] Direct KB test returns grounded answers with citations
- [ ] Agent connected to KB passes all four tests
- [ ] (Bonus) Compared reasoning effort levels on the same query

---

## Next Module
→ [05-Extend-to-Work-IQ.md](05-Extend-to-Work-IQ.md) — Extend your agent to Microsoft 365 data via Work IQ.
