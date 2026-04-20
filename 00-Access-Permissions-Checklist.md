# Module 00 — Access, Permissions, and RBAC Checklist

## Why This Module Exists
This is the **#1 blocker** in every Foundry workshop. If access and permissions are not sorted before you start building, you will hit errors in every subsequent module. Complete every item in this checklist before moving to Module 01.

> **Note:** Complete this checklist **at least 48 hours before starting the workshop**. Several items require admin action or propagation time.

---

## Part 1: Azure Subscription Access

### 1.1 Verify You Have an Azure Subscription

1. Open **https://portal.azure.com**
2. Sign in with your work account
3. In the top search bar, type **Subscriptions** → click **Subscriptions**
4. You should see at least one subscription listed

> **If no subscriptions appear:** You need an Azure subscription. Ask your Azure admin to either assign you to an existing subscription or create a trial at https://azure.microsoft.com/free.

### 1.2 Verify Your Role on the Subscription

1. Click on your subscription name
2. In the left menu, click **Access control (IAM)**
3. Click the **Role assignments** tab
4. Search for your email address
5. Confirm you see **Contributor** or **Owner** next to your name

> **If you see Reader or no role:** Ask your Azure admin to assign **Contributor** on the subscription. This is required to create AI Search, AI Services, and storage resources.

### Record Your Values

| Item | Your Value |
|---|---|
| Subscription name | ________________ |
| Subscription ID | ________________ |
| Your role | ________________ |

- [ ] **Checkpoint:** I have Contributor or Owner on my Azure subscription.

---

## Part 2: Azure AI Foundry Portal Access

### 2.1 Verify Foundry Portal Access

1. Open **https://ai.azure.com**
2. Sign in with the same work account
3. You should see the Foundry home page with **+ Create project** visible

> **If you get "Access Denied" or a blank page:**
> - Your organization may have conditional access policies blocking ai.azure.com
> - Your account may not be licensed for Azure AI services
> - Contact your IT admin to whitelist ai.azure.com

### 2.2 Verify You Can Create Resources

1. On the Foundry home page, click **+ Create project** (do NOT complete it yet — just verify the form loads)
2. You should see fields for Project Name, Hub, Region, Subscription, and Resource Group
3. Click **Cancel** — we will create the project in Module 01

- [ ] **Checkpoint:** I can access ai.azure.com and see the Create Project form.

---

## Part 3: Region Selection and Model Quota

### 3.1 Choose Your Region

Not all regions support all Foundry features. Your region must support:
- **GPT-4o** model deployment
- **text-embedding-3-large** model deployment
- **Semantic Ranker** (AI Search)
- **Agentic Retrieval** (Foundry IQ)

**Recommended regions:**

| Region | GPT-4o | Embeddings | Semantic Ranker | Agentic Retrieval |
|---|---|---|---|---|
| **East US 2** | ✅ | ✅ | ✅ | ✅ |
| **Sweden Central** | ✅ | ✅ | ✅ | ✅ |
| **West US 3** | ✅ | ✅ | ✅ | ✅ |
| **South Central US** | ✅ | ✅ | ✅ | ✅ |

### 3.2 Check Model Quota

1. Go to **https://ai.azure.com**
2. Click **Models + endpoints** (or **Quotas** if visible)
3. Check that your subscription has available tokens-per-minute for:
   - **gpt-4o**: at least 30K TPM
   - **text-embedding-3-large**: at least 30K TPM

> **If quota is 0 or fully consumed:**
> - Delete unused model deployments to free quota
> - Try a different region
> - Request a quota increase via Azure Portal → **Help + support** → **New support request** → "Service and subscription limits (quotas)"

| Item | Your Value |
|---|---|
| Chosen region | ________________ |
| gpt-4o available TPM | ________________ |
| text-embedding-3-large available TPM | ________________ |

- [ ] **Checkpoint:** I have confirmed model quota is available in my chosen region.

---

## Part 4: RBAC Role Assignments (Critical)

There are **two RBAC flows** needed in this workshop. Both use managed identities. If you skip either one, downstream modules fail silently or with cryptic 403 errors.

### Understanding the Two Flows

```
RBAC Flow 1 — Indexing Time (Module 02)
┌─────────────────────┐         ┌──────────────────────────┐
│  AI Search Service   │ ──────▶│  Azure OpenAI / AI Svcs  │
│  (System MI)         │  needs  │                          │
│                      │  role:  │  "Cognitive Services     │
│  Resource type in    │         │   OpenAI User"           │
│  picker: Search      │         │                          │
│  services            │         │                          │
└─────────────────────┘         └──────────────────────────┘

RBAC Flow 2 — Agent Runtime (Module 03)
┌─────────────────────┐         ┌──────────────────────────┐
│  Foundry Project     │ ──────▶│  AI Search Service       │
│  (System MI)         │  needs  │                          │
│                      │  roles: │  • Search Index Data     │
│  Resource type in    │         │    Reader                │
│  picker: Foundry     │         │  • Search Index Data     │
│                      │         │    Contributor           │
│                      │         │  • Search Service        │
│                      │         │    Contributor           │
└─────────────────────┘         └──────────────────────────┘
```

> **You will perform these assignments in Modules 01–03 after resources exist.** This section is for awareness — understand the pattern now so you don't get confused later.

### 4.1 Pre-Assignment: Enable Managed Identity (After Resource Creation)

After creating AI Search (Module 01), you MUST enable its system-assigned managed identity:

1. Azure Portal → AI Search resource → **Settings** → **Identity**
2. **System assigned** tab → Status: **On** → **Save** → **Yes**
3. Wait for Object ID to appear

> 🔴 **This is the #1 most missed step.** If you skip this, the AI Search service will NOT appear in any RBAC role picker — the dropdown will be empty and you will think Azure is broken.

### 4.2 The Identity Picker Trap

When assigning roles, you must select the correct **resource type** in the managed identity picker:

| RBAC Flow | Resource Type to Select | Common Mistake |
|---|---|---|
| Flow 1 (Search → OpenAI) | **Search services** | Selecting "Foundry" instead |
| Flow 2 (Foundry → Search) | **Foundry** | Selecting "Search services" instead |

> The wrong selection completes without error but the wrong identity gets the role. The downstream operation fails silently.

- [ ] **Checkpoint:** I understand the two RBAC flows and the identity picker trap.

---

## Part 5: Entra ID Roles (SharePoint Path Only)

Skip this section if you will only use Blob Storage and Web knowledge sources.

### 5.1 Roles Needed for SharePoint Indexed Knowledge Source

To create an app registration and grant admin consent for SharePoint indexing:

| Entra Role | Why Needed |
|---|---|
| **Application Administrator** | Create app registrations and client secrets |
| **Cloud Application Administrator** | Grant admin consent for Graph API permissions |

### 5.2 Check Your Entra Roles

1. Open **https://entra.microsoft.com**
2. Navigate: **Identity** → **Users** → search for your name → click your profile
3. Click **Assigned roles**
4. Verify you have the roles above

> **If roles are missing:**
> - If your org uses **PIM (Privileged Identity Management):** Go to **Identity** → **Roles and administrators** → find the role → **Activate** → provide justification
> - If PIM is not available: Request the role from your Entra admin

### 5.3 Graph API Permissions You Will Need

The app registration (created in Module 02, SharePoint path) will require:

| Permission | Type | Requires Admin Consent |
|---|---|---|
| `Sites.Read.All` | Application | Yes |
| `Files.Read.All` | Application | Yes |

- [ ] **Checkpoint:** I have the Entra roles needed (or know who to contact).

---

## Part 6: Microsoft 365 Access (Work IQ Module Only)

Skip this section if you will not complete Module 05 (Work IQ).

### 6.1 Verify Microsoft 365 License

1. Go to **https://portal.office.com**
2. Sign in → click on your profile icon → **My account**
3. Under **Subscriptions**, verify you have:
   - Microsoft 365 E3/E5 or Business Premium (minimum for SharePoint + Teams)
   - **Microsoft 365 Copilot** license (required for Work IQ features)

### 6.2 Verify SharePoint Access

1. Go to **https://\<your-tenant\>.sharepoint.com**
2. Verify you can access at least one site with documents

### 6.3 Graph API Access

Work IQ uses Microsoft Graph to access M365 data. Verify:
- Your account can access **https://graph.microsoft.com**
- No conditional access policies block Graph API calls from Azure services

- [ ] **Checkpoint:** I have M365 access with Copilot license (for Work IQ module).

---

## Part 7: Pre-Workshop Summary Checklist

Complete this before Module 01. Confirm all items are checked before proceeding.

| # | Item | Status |
|---|---|---|
| 1 | Azure subscription with Contributor/Owner role | ☐ |
| 2 | Can access https://ai.azure.com and see Create Project form | ☐ |
| 3 | Region selected with model quota available | ☐ |
| 4 | Understand the two RBAC flows and identity picker trap | ☐ |
| 5 | (SharePoint) Entra roles: Application Admin + Cloud App Admin | ☐ |
| 6 | (Work IQ) M365 license with Copilot | ☐ |

| Value | Record Here |
|---|---|
| Subscription name | ________________ |
| Subscription ID | ________________ |
| Chosen region | ________________ |
| Your email/UPN | ________________ |

> **Important:** Do not start Module 01 until items 1–4 are checked. Items 5–6 are only needed for later modules.
