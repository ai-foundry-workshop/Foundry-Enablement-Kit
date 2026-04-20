# Module 02 — Knowledge Sources Deep Dive

## What You Are Building
You will connect **all five types of knowledge sources** available in Foundry to your project. Each source type serves a different enterprise scenario. By the end of this module, you will have data flowing from multiple sources into your Foundry project.

## Scenario Context
> Contoso HR has data scattered across multiple systems:
> - **Company policies** live in Azure Blob Storage (PDFs and markdown files)
> - **Product/benefits FAQs** are indexed in Azure AI Search
> - **Real-time benefit provider info** needs live web lookup
> - **Department handbooks** live in SharePoint (needs governance-preserving access)
> - **Compliance documents** in SharePoint need full-text indexing for deep search

## The Five Knowledge Source Types

| # | Source Type | What It Does | Best For |
|---|---|---|---|
| 1 | **Azure AI Search Index** | Connects to an existing AI Search index | Pre-built search pipelines, custom indexes |
| 2 | **Azure Blob Storage** | Retrieves documents/files directly from Blob | PDFs, Word docs, markdown stored in Azure |
| 3 | **Web (Bing)** | Grounds responses with real-time web content | Current events, external documentation, live data |
| 4 | **Microsoft SharePoint (Remote)** | SharePoint search with M365 governance intact | When you need SharePoint permissions enforced, no re-indexing |
| 5 | **Microsoft SharePoint (Indexed)** | Indexes SharePoint content into AI Search | When you need full-text + vector search over SharePoint docs |

---

## Source 1: Azure AI Search Index

### When to Use
You already have an AI Search index (from an existing pipeline, custom ingestion, or the Import Data wizard) and want to connect it to a Foundry agent.

### Step 2.1: Create a Search Index Using Import Data Wizard

If you already have an index, skip to Step 2.1c.

#### Step 2.1a: Upload Sample Documents to Blob Storage

1. In **Azure Portal** → your resource group (`contoso-hr-rg`)
2. Click **+ Create** → search **Storage account** → **Create**
3. Fill in:

   | Field | Value |
   |---|---|
   | **Name** | `contosohrstorage` (globally unique, lowercase) |
   | **Region** | Same as Foundry |
   | **Performance** | Standard |
   | **Redundancy** | LRS |

4. **Review + create** → **Create** → **Go to resource**
5. Left menu → **Containers** → **+ Container** → Name: `documents` → **Create**
6. Click into `documents` → **Upload** → select the 3 files from `sample-data/` folder → **Upload**

#### Step 2.1b: Run the Import Data Wizard

1. Go to your **AI Search** resource → **Overview** → click **Import data**
2. **Step 1 — Data source:**
   - Data source: **Azure Blob Storage**
   - Name: `blob-datasource`
   - Connection string: **Choose existing connection** → your storage account → `documents` container
   - Click **Next**
3. **Step 2 — Cognitive skills:**
   - Skillset name: `doc-skillset`
   - Attach your AI Services resource
   - (Optional) Enable text-related enrichments
   - Click **Next**
4. **Step 3 — Target index:**
   - Index name: `hr-documents-index`
   - Ensure `content` is **Searchable** ✅ and **Retrievable** ✅
   - Ensure `metadata_storage_name` is **Retrievable** ✅ and **Filterable** ✅
   - Click **Next**
5. **Step 4 — Indexer:**
   - Name: `doc-indexer`
   - Schedule: **Once**
   - Click **Submit**
6. Wait 1–3 minutes

#### Step 2.1c: Verify the Index

1. AI Search → **Indexes** → click `hr-documents-index`
2. Click **Search** → query: `*` → click **Search**
3. You should see your documents with extracted content

#### Step 2.1d: Connect as Knowledge Source in Foundry

1. Go to **https://ai.azure.com** → your project
2. Navigate to **Knowledge** → **Knowledge sources**
3. Click **Create new** → select **Azure AI Search Index**
4. Fill in:
   - **Connection:** Select your AI Search connection (or create new with search URL + API key)
   - **Index:** `hr-documents-index`
5. Click **Create**
6. Wait for status: **Active** ✅

### Record Value

| Item | Your Value |
|---|---|
| Knowledge source name (AI Search) | ________________ |

- [ ] **Checkpoint:** AI Search index knowledge source shows Active.

---

## Source 2: Azure Blob Storage

### When to Use
You want Foundry to directly read files from a Blob container without going through AI Search. Simpler setup, good for smaller document sets.

### Step 2.2: Create Blob Storage Knowledge Source

1. In Foundry → **Knowledge** → **Knowledge sources**
2. Click **Create new** → select **Azure Blob Storage**
3. Fill in:
   - **Connection:** Select your storage account connection (or create new)
   - **Container:** `documents`
   - (Optional) Specify a folder path within the container
4. Click **Create**
5. Wait for status: **Active** ✅

> **Note:** The Blob source uses Foundry's built-in parsing and chunking. It supports PDF, DOCX, PPTX, TXT, MD, and HTML files.

### How This Differs from AI Search Index Source

| Aspect | Blob Storage Source | AI Search Index Source |
|---|---|---|
| **Indexing** | Foundry handles parsing/chunking | You control the index schema and pipeline |
| **Search type** | Basic retrieval | Full hybrid (keyword + vector + semantic) |
| **Best for** | Quick setup, small doc sets | Production, large scale, custom relevance tuning |
| **Update frequency** | Re-syncs when KB refreshes | Controlled by indexer schedule |

- [ ] **Checkpoint:** Blob Storage knowledge source shows Active.

---

## Source 3: Web (Bing Grounding)

### When to Use
Your agent needs to answer questions using real-time web information — current events, external documentation, product pages, competitor info.

### Step 2.3: Create Web Knowledge Source

1. In Foundry → **Knowledge** → **Knowledge sources**
2. Click **Create new** → select **Web**
3. This creates a Bing-grounded knowledge source
4. Configuration options:
   - **Bing search resource:** Select existing or create new (a Bing Search resource in Azure)
   - **Search scope:** (Optional) Restrict to specific domains if needed
5. Click **Create**
6. Wait for status: **Active** ✅

### Creating a Bing Search Resource (if you don't have one)

1. Azure Portal → **+ Create a resource** → search **Bing Search** → **Create**
2. Fill in:
   - Resource group: `contoso-hr-rg`
   - Name: `contoso-bing-search`
   - Pricing tier: **S1** (or Free for testing)
3. **Create** → copy the API key from **Keys and Endpoint**
4. Return to Foundry and use this key when creating the Web source connection

> **When NOT to use this:** If your agent should ONLY answer from internal documents. Bing grounding can introduce external information. Control this via agent instructions in Module 03.

- [ ] **Checkpoint:** Web (Bing) knowledge source shows Active.

---

## Source 4: Microsoft SharePoint (Remote)

### When to Use
You want your agent to search SharePoint content **while preserving Microsoft 365 security trimming**. Documents are NOT copied into AI Search — the query is passed to SharePoint search, and only results the user has permission to see are returned.

### Prerequisites
- Microsoft 365 tenant with SharePoint Online
- User must have access to the target SharePoint site
- No Entra app registration needed (uses delegated access)

### Step 2.4: Create SharePoint Remote Knowledge Source

1. In Foundry → **Knowledge** → **Knowledge sources**
2. Click **Create new** → select **Microsoft SharePoint (Remote)**
3. Fill in:
   - **M365 connection:** Sign in with your M365 account (or select existing connection)
   - **SharePoint site URL:** The URL of your SharePoint site (e.g., `https://contoso.sharepoint.com/sites/HR`)
   - (Optional) **Document library:** Restrict to a specific library
4. Click **Create**
5. Wait for status: **Active** ✅

### How Remote Differs from Indexed

| Aspect | SharePoint Remote | SharePoint Indexed |
|---|---|---|
| **Security** | M365 permission trimming preserved | Application-level access (bypasses user permissions) |
| **Search quality** | SharePoint search (keyword-based) | AI Search (hybrid + vector + semantic) |
| **Latency** | Real-time from SharePoint | Pre-indexed, faster retrieval |
| **Setup** | Simple (sign in with M365) | Complex (app registration, admin consent, indexer) |
| **Best for** | Governance-sensitive data, quick setup | Deep search, large document sets |

> **Choose Remote when:** Data governance and user-level permissions matter more than search quality.

- [ ] **Checkpoint:** SharePoint Remote knowledge source shows Active.

---

## Source 5: Microsoft SharePoint (Indexed)

### When to Use
You need full-text + vector search over SharePoint documents with AI Search's hybrid retrieval capabilities. Content is indexed into AI Search for maximum search quality.

### Prerequisites
- Entra app registration with `Sites.Read.All` and `Files.Read.All` (Application permissions)
- Admin consent granted
- See Module 00, Part 5 for Entra role requirements

### Step 2.5a: Create Entra App Registration

1. Open **https://entra.microsoft.com**
2. **Identity** → **Applications** → **App registrations** → **+ New registration**
3. Fill in:
   - Name: `foundry-sp-indexer`
   - Supported account types: **Accounts in this organizational directory only**
   - Redirect URI: leave blank
4. Click **Register**
5. Record the **Application (client) ID** and **Directory (tenant) ID**

### Step 2.5b: Create Client Secret

1. In the app registration → **Certificates & secrets** → **+ New client secret**
2. Description: `foundry-workshop` → Expiry: **6 months** → **Add**
3. **IMMEDIATELY copy the Value column** (NOT the Secret ID)

> ⚠️ The Value disappears forever once you navigate away from this page.

### Step 2.5c: Add API Permissions

1. In the app → **API permissions** → **+ Add a permission**
2. Select **Microsoft Graph** → **Application permissions**
3. Search and check:
   - `Sites.Read.All`
   - `Files.Read.All`
4. Click **Add permissions**
5. Click **Grant admin consent for \<your tenant\>** → **Yes**
6. Verify green checkmarks appear next to both permissions

### Step 2.5d: Create SharePoint Indexed Knowledge Source

1. In Foundry → **Knowledge** → **Knowledge sources**
2. Click **Create new** → select **Microsoft SharePoint (Indexed)**
3. Fill in:
   - **SharePoint site URL:** `https://contoso.sharepoint.com/sites/HR`
   - **Tenant ID:** from Step 2.5a
   - **Client ID:** from Step 2.5a
   - **Client Secret:** the Value from Step 2.5b
   - **AI Search connection:** your existing AI Search service
4. Click **Create**
5. Wait for indexing to complete (may take several minutes depending on document volume)
6. Status should show **Active** ✅

### Record Values

| Item | Your Value |
|---|---|
| App registration Client ID | ________________ |
| Tenant ID | ________________ |
| SharePoint site URL | ________________ |

- [ ] **Checkpoint:** SharePoint Indexed knowledge source shows Active.

---

## Summary: All Five Sources Connected

After completing this module, your Foundry project should have:

| # | Knowledge Source | Status |
|---|---|---|
| 1 | AI Search Index (`hr-documents-index`) | ✅ Active |
| 2 | Azure Blob Storage (`documents` container) | ✅ Active |
| 3 | Web / Bing Grounding | ✅ Active |
| 4 | SharePoint Remote | ✅ Active |
| 5 | SharePoint Indexed | ✅ Active |

> **Note:** You don't need ALL five for the workshop to continue. **At minimum, complete Source 1 (AI Search) or Source 2 (Blob)**. The others demonstrate breadth and can be added later.

---

## Next Module
→ [03-Build-Your-First-Agent.md](03-Build-Your-First-Agent.md) — Create an agent, attach knowledge, and test grounded responses.
