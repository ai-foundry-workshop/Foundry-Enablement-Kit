# Module 03 — Build Your First Agent

## What You Are Building
A conversational AI agent that answers questions grounded in your enterprise knowledge sources — with citations, refusal for out-of-scope questions, and proper grounding behavior.

## Scenario Context
> Contoso HR wants the onboarding assistant to answer employee questions like "What is the dress code policy?" using only company documents. It should cite sources. It must refuse to answer questions not covered in the knowledge base.

---

## Step 3.1: RBAC Flow 2 — Foundry → Search

Before the agent can query knowledge, grant the Foundry project identity access to AI Search.

1. Azure Portal → your **AI Search** resource → **Access control (IAM)**
2. Click **+ Add** → **Add role assignment**
3. You need to assign **three roles** (repeat steps 3–7 for each):

   | Role | Purpose |
   |---|---|
   | `Search Index Data Reader` | Agent can read index content |
   | `Search Index Data Contributor` | Agent can write to indexes (for Foundry IQ operations) |
   | `Search Service Contributor` | Agent can manage search service resources |

4. **For each role:**
   - **Role tab:** Search the role name → select → **Next**
   - **Members tab:** Select **Managed identity** → **+ Select members**
   - **In the side panel:**

     | Field | Select |
     |---|---|
     | **Managed identity** | System-assigned managed identity |
     | **Resource type** | ⚠️ **Foundry** (or Microsoft Foundry) — NOT Search services |

   - Select your Foundry project → **Select** → **Review + assign**

5. After all three, verify under **Role assignments** tab:
   - Your Foundry project appears **three times** with the three roles above

> 🔴 **Common mistake:** Selecting "Search services" instead of "Foundry" in the resource type dropdown. This assigns the role to the wrong identity. The assignment completes without error but the agent gets 403.

- [ ] **Checkpoint:** Three role assignments visible for the Foundry project on AI Search.

---

## Step 3.2: Create the Agent

1. In **https://ai.azure.com** → your project
2. Click **Agents** in the left menu
3. Click **+ New agent**
4. Fill in:

   | Field | What to Enter |
   |---|---|
   | **Name** | `contoso-hr-onboarding` |
   | **Model** | `gpt-4o` (your deployed model) |
   | **Instructions** | See below |

### Agent Instructions (paste this in the Instructions field)

```
You are the Contoso HR Onboarding Assistant. Your role is to help new employees find information about company policies, benefits, onboarding procedures, and workplace guidelines.

RULES:
1. ALWAYS retrieve information from the attached knowledge base before answering.
2. NEVER answer from your own training data or general knowledge.
3. When you find relevant information, provide a clear answer and cite the source document.
4. If the question is not covered in the knowledge base, respond EXACTLY with:
   "I could not find this information in the Contoso HR knowledge base. Please contact hr@contoso.com for assistance."
5. Be concise, professional, and helpful.
6. If a question is ambiguous, ask for clarification before searching.
```

5. Click **Save** (or the agent is auto-saved)

---

## Step 3.3: Attach Knowledge to the Agent

### Option A: Attach via Knowledge Base (Recommended)
If you have already created a Knowledge Base in Module 04 (or want to create one now):

1. In the agent configuration panel, find **Knowledge** or **Tools** section
2. Click **Add knowledge base** (or **Add tool** → **Foundry IQ**)
3. Select the knowledge base you want to attach
4. Click **Save**

### Option B: Attach Knowledge Source Directly
If you want to connect a knowledge source without a KB wrapper:

1. In the agent configuration, find **Knowledge** section
2. Click **Add knowledge source**
3. Select from your available sources (created in Module 02):
   - `hr-documents-index` (AI Search)
   - Blob Storage source
   - Web (Bing) source
   - SharePoint sources
4. Click **Save**

> **Best practice:** Start with ONE knowledge source (AI Search or Blob) for initial testing. Add more sources after confirming the agent works correctly.

---

## Step 3.4: Test the Agent

### Test 1: In-Scope Question (Should Answer with Citation)

In the agent playground chat box, type:

```
What is the company dress code policy?
```

**Expected behavior:**
- Agent retrieves from knowledge base
- Provides a grounded answer
- Cites the source document (e.g., "According to company-policy.md...")

### Test 2: Specific Content Query (Should Summarize with References)

```
What are the steps for setting up my laptop on day one?
```

**Expected behavior:**
- Agent searches the onboarding guide
- Returns step-by-step info from the document
- References the source

### Test 3: Out-of-Scope Question (Should Refuse)

```
What is the capital of France?
```

**Expected behavior:**
- Agent responds: "I could not find this information in the Contoso HR knowledge base. Please contact hr@contoso.com for assistance."

### Test 4: Ambiguous Question (Should Clarify)

```
Tell me about the policy.
```

**Expected behavior:**
- Agent asks: "Could you specify which policy? For example, dress code, PTO, or remote work?"

### Test Results

| Test | Expected | Actual | Pass? |
|---|---|---|---|
| In-scope question | Grounded answer + citation | ________________ | ☐ |
| Specific content | Summary + source reference | ________________ | ☐ |
| Out-of-scope | Refusal message | ________________ | ☐ |
| Ambiguous | Clarification request | ________________ | ☐ |

---

## Step 3.5: Tune Agent Behavior

### If the Agent Answers from General Knowledge (Not Grounded)

**Check 1:** Knowledge source is attached
- Agent config → Knowledge section → verify a source is listed

**Check 2:** Instructions are strong enough
- The instructions must explicitly say "NEVER answer from your own training data"
- Add: "If no relevant documents are found, say you don't have that information"

**Check 3:** Knowledge base status
- Go to Knowledge → verify the source shows **Active** ✅
- If it shows an error, the search connection may be broken

### If the Agent Refuses Everything

**Check 1:** Search index has documents
- AI Search → Indexes → query `*` → verify documents exist

**Check 2:** RBAC is correct
- AI Search → IAM → verify Foundry project has the three roles from Step 3.1

**Check 3:** Relax the instructions slightly
- Change "EXACTLY" to "something like" in the refusal instruction
- Or add: "Answer questions that are clearly related to HR topics, even if the exact phrasing doesn't match a document"

### If You Get a 403 Error

This means RBAC Flow 2 is missing or wrong. Go back to Step 3.1.

---

## Step 3.6: Add Multiple Knowledge Sources

Once the agent works with one source, add more:

1. In the agent config → **Knowledge** → **Add knowledge source**
2. Add your second source (e.g., if you started with AI Search, now add Blob or Bing)
3. **Save**
4. Test again — ask a question that would require the new source

### Multi-Source Behavior

When an agent has multiple knowledge sources:
- Foundry queries **all attached sources** in parallel
- Results are merged and reranked
- The most relevant content from any source is used for the answer
- Citations show which source the information came from

> **Pro tip:** Use the **Bing/Web** source for questions like "What are the latest tax filing deadlines?" (real-time info) while the Blob/Search sources handle internal policies (static docs).

---

## Verification — You Are Done When

- [ ] Agent exists and is configured with instructions
- [ ] At least one knowledge source is attached
- [ ] In-scope questions return grounded answers with citations
- [ ] Out-of-scope questions are refused appropriately
- [ ] No 403 or access errors

---

## Next Module
→ [04-Foundry-IQ-Knowledge-Bases.md](04-Foundry-IQ-Knowledge-Bases.md) — Build multi-source knowledge bases with reasoning effort and citation modes.
