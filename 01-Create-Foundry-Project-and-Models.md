# Module 01 — Create Foundry Project and Deploy Models

## What You Are Building
Your AI workspace — the central hub for models, agents, knowledge bases, and evaluations. Plus the two models every agent needs: a chat model for answers and an embedding model for search.

## Scenario Context
> Contoso HR needs a central AI project where the onboarding assistant will live. This module creates the project and deploys the models it will use.

---

## Step 1.1: Create the Foundry Project

1. Open **https://ai.azure.com** → sign in
2. Click **+ Create project** (blue button, top area)
3. Fill in the form:

   | Field | What to Enter | Notes |
   |---|---|---|
   | **Project Name** | `contoso-hr-assistant` | Or your preferred name |
   | **Hub** | Click **Create New** | A sub-form opens |
   | **Hub Name** | `contoso-ai-hub` | Shared container for projects |
   | **Region** | Your chosen region from Module 00 | Must support Semantic Ranker + Agentic Retrieval |
   | **Subscription** | Your subscription | From Module 00 checklist |
   | **Resource Group** | Click **Create New** → `contoso-hr-rg` | Or use existing |

4. Click **Create**
5. Wait 2–3 minutes for provisioning

### What Gets Created Behind the Scenes
When you create a Foundry project with a new Hub, Azure provisions several resources automatically:

| Resource | Purpose | Where to Find It |
|---|---|---|
| **AI Hub** | Shared container for projects | ai.azure.com → Management → All hubs |
| **AI Project** | Your workspace | ai.azure.com → your project |
| **Azure AI Services** | Hosts model deployments (OpenAI endpoint) | Azure Portal → resource group |
| **Storage Account** | Stores project artifacts | Azure Portal → resource group |
| **Key Vault** | Stores secrets | Azure Portal → resource group |

### Verification — You Are Done When
You see the **Foundry Project Overview page** with these sections in the left menu:
- **Build** (or **Playground**)
- **Agents**
- **Models + endpoints**
- **Data + indexes** (or **Knowledge**)

### Record These Values

| Value | Where to Find It | Your Value |
|---|---|---|
| Project Name | Top of the project page | ________________ |
| Region | Project settings | ________________ |
| Resource Group | Project settings | ________________ |
| Hub Name | Project settings → Hub | ________________ |

- [ ] **Checkpoint:** I can see the project overview with Agents, Models, and Knowledge in the menu.

---

## Step 1.2: Deploy the Chat Model (GPT-4o)

1. In your Foundry project, click **Models + endpoints** in the left menu
2. Click **+ Deploy model** → **Deploy base model**
3. Search for **gpt-4o** → click on it
4. Fill in:

   | Field | What to Enter |
   |---|---|
   | **Deployment name** | `gpt-4o` |
   | **Deployment type** | `GlobalStandard` (if available; else `Standard`) |
   | **Tokens per minute** | `80K` (or whatever is available) |

5. Click **Deploy**
6. Wait for status to show **Succeeded**

### Quick Test

1. Click on the **gpt-4o** deployment name
2. Find the **Playground** or **Test** section
3. Type: `What is an employee onboarding checklist?`
4. Confirm you get a response

---

## Step 1.3: Deploy the Embedding Model

1. Click **+ Deploy model** → **Deploy base model** again
2. Search for **text-embedding-3-large** → click on it
3. Fill in:

   | Field | What to Enter |
   |---|---|
   | **Deployment name** | `text-embedding-3-large` |
   | **Deployment type** | `Standard` |

4. Click **Deploy**
5. Wait for status to show **Succeeded**

> **Why this model?** `text-embedding-3-large` produces 3072-dimension vectors. When we create search indexes later, the vector field dimensions must match exactly. This is the highest-quality embedding model available.

---

## Step 1.4: Record Endpoint Information

1. Under **Models + endpoints** → **Deployments**, click on **gpt-4o**
2. Find the **Target URI** (e.g., `https://contoso-hr-assistant-xxxx.openai.azure.com/`)
3. Record it below

| Value | Your Value |
|---|---|
| OpenAI Endpoint URL | ________________ |
| Chat deployment name | `gpt-4o` |
| Embedding deployment name | `text-embedding-3-large` |

---

## Step 1.5: Create Azure AI Search Service

The retrieval engine that stores document chunks and vectors.

1. Open **https://portal.azure.com** in a new tab
2. Click **+ Create a resource** → search **Azure AI Search** → click **Create**
3. Fill in:

   | Field | What to Enter | Notes |
   |---|---|---|
   | **Subscription** | Your subscription | Same as Foundry project |
   | **Resource group** | `contoso-hr-rg` | Same as Foundry project |
   | **Service name** | `contoso-hr-search` | Must be globally unique |
   | **Location** | Same region as Foundry | Choose nearby if unavailable |
   | **Pricing tier** | **Standard S1** | Free/Basic lack Semantic Ranker and SharePoint indexer |

4. Click **Review + create** → **Create**
5. Wait 1–2 minutes

### Step 1.5a: Enable RBAC Authentication

1. Open the deployed AI Search resource → **Settings** → **Keys**
2. Under **API access control**, select **Both**
3. Click **Save**

### Step 1.5b: Enable Managed Identity

> 🔴 **If you skip this, AI Search will NOT appear in any RBAC picker later.**

1. In AI Search → **Settings** → **Identity**
2. **System assigned** tab → Status: **On** → **Save** → **Yes**
3. Wait for **Object ID** to appear

### Step 1.5c: Record Values

| Value | Your Value |
|---|---|
| Search service name | ________________ |
| Search URL | `https://<name>.search.windows.net` |
| Admin API key | Settings → Keys → Primary admin key: ________________ |
| Object ID (managed identity) | ________________ |

---

## Step 1.6: RBAC Flow 1 — Search → OpenAI

Grant AI Search permission to call the embedding model during indexing.

1. In Azure Portal, find your **Azure AI Services** resource (in the resource group — brain icon)
2. Click **Access control (IAM)** → **+ Add** → **Add role assignment**
3. **Role tab:** Search `Cognitive Services OpenAI User` → select → **Next**
4. **Members tab:** Select **Managed identity** → **+ Select members**
5. **In the side panel:**

   | Field | Select |
   |---|---|
   | **Managed identity** | System-assigned managed identity |
   | **Subscription** | Your subscription |
   | **Resource type** | ⚠️ **Search services** (NOT Foundry) |

6. Select your AI Search service → **Select** → **Review + assign** → **Review + assign**

### Verification
Under Access control (IAM) → Role assignments, you see:
- **Principal:** your search service — **Role:** `Cognitive Services OpenAI User`

- [ ] **Checkpoint:** Both models show Succeeded. AI Search has MI enabled. RBAC Flow 1 is assigned.

---

## Next Module
→ [02-Knowledge-Sources-Deep-Dive.md](02-Knowledge-Sources-Deep-Dive.md) — Connect all 5 knowledge source types.
