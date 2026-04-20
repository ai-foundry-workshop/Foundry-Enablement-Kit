# Module 07 — Troubleshooting Playbook

## Purpose
Use this playbook during live labs to quickly diagnose and unblock learners. Every issue was encountered in real workshops with exact error messages, root causes, and portal navigation to fix.

## Fast Triage Flow
1. Ask: "What exactly do you see on screen?" (get the exact error message)
2. Find the matching issue below
3. Follow the fix steps
4. Run the verification step before moving on

---

## Issue 1: Cannot Create Foundry Project — Permission Denied

### Error
```
You don't have permission to create resource
```
or
```
The client does not have authorization to perform action
```

### Root Cause
Account does not have **Contributor** or **Owner** role on the subscription.

### Fix
1. Azure Portal → **Subscriptions** → click subscription → **Access control (IAM)**
2. **+ Add** → **Add role assignment**
3. Role: `Contributor` → **Next**
4. **+ Select members** → find learner's email → **Select** → **Review + assign**
5. Wait 30 seconds for propagation

### Verification
Learner refreshes ai.azure.com → **+ Create project** works without error.

---

## Issue 2: Region Out of Capacity

### Error
```
The region 'eastus2' is currently out of the resources required
```

### Fix
Change region to: `South Central US`, `West US 3`, or `Sweden Central`. Cross-region connections still work.

---

## Issue 3: Model Deployment — Quota Exceeded

### Error
```
InsufficientQuota: The specified capacity ... is not available
```

### Fix
1. Foundry → **Models + endpoints** → **Quotas**
2. Options:
   - Reduce TPM (e.g., 80K → 30K)
   - Delete unused deployments
   - Try `Standard` instead of `GlobalStandard`
   - Try a different region

### Verification
Deployment status: **Succeeded**.

---

## Issue 4: AI Search Managed Identity Not Appearing in RBAC Picker

### Symptom
AI Search service does not appear in the managed identity selection list.

### Root Cause
System Assigned Managed Identity was not enabled on AI Search.

### Fix
1. Azure Portal → AI Search → **Settings** → **Identity**
2. **System assigned** tab → Status: **On** → **Save** → **Yes**
3. Wait 15–20 seconds for Object ID to appear

### Verification
Object ID is displayed. AI Search now appears in RBAC pickers.

---

## Issue 5: Wrong Identity Selected in RBAC Dropdown

### Symptom
Role assignment completes, but downstream operation still fails (indexer 403 or agent 403).

### Root Cause
Wrong **resource type** selected in the managed identity picker:
- Selected "Foundry" when it should be "Search services" (or vice versa)

### How to Diagnose
1. Resource → **Access control (IAM)** → **Role assignments**
2. Check the **Principal** column — if the wrong service is listed, the assignment is on the wrong identity

### Fix
Do NOT delete the wrong assignment. Add a NEW one with the correct identity:
- **RBAC Flow 1** (Search → OpenAI): resource type = **Search services**
- **RBAC Flow 2** (Foundry → Search): resource type = **Foundry**

---

## Issue 6: Indexer Fails — Cannot Authenticate to OpenAI

### Error
```
Failed to authenticate using managed identity for endpoint
'https://....openai.azure.com/'
```

### Root Cause
RBAC Flow 1 is missing. AI Search does not have `Cognitive Services OpenAI User` on the OpenAI resource.

### Fix
1. Azure Portal → Azure AI Services resource → **Access control (IAM)**
2. **+ Add** → **Add role assignment**
3. Role: `Cognitive Services OpenAI User`
4. Members: Managed identity → Resource type: **Search services** → select your search service
5. **Review + assign**

### Verification
AI Search → **Indexers** → click indexer → **Run** → status: **Success**.

---

## Issue 7: Agent Returns 403 — MCP Access Denied

### Error
```
ErrorAccess denied when connecting to the MCP server at
https://<search>.search.windows.net:443/knowledgebases/.../mcp
while enumerating tools (HTTP 403 Forbidden)
```

### Root Cause
RBAC Flow 2 is missing. Foundry project does not have the required roles on AI Search.

### Fix
Assign **three roles** on the AI Search resource:
1. `Search Index Data Reader`
2. `Search Index Data Contributor`
3. `Search Service Contributor`

For each:
- Members: Managed identity → Resource type: **Foundry** → select your project
- **Review + assign**

Wait 30 seconds for propagation.

### Verification
Agent playground → retry query → response comes through without 403.

---

## Issue 8: Invalid Client Secret (SharePoint Indexed Path)

### Error
```
AADSTS7000215: Invalid client secret provided.
Ensure the secret being sent in the request is the client secret value,
not the client secret ID.
```

### Root Cause
Copied the **Secret ID** column instead of the **Value** column.

### Fix
1. Entra → App registrations → your app → **Certificates & secrets**
2. **+ New client secret** → Add
3. **IMMEDIATELY copy the Value column** (not Secret ID)
4. Update the configuration and retry

### Important
> ⚠️ You will see two columns. Copy **VALUE**, not Secret ID. The Value disappears once you leave this page.

---

## Issue 9: Admin Consent Button Greyed Out

### Symptom
App registration → API permissions → "Grant admin consent" button is greyed out.

### Root Cause
Account lacks Entra admin role (Cloud Application Administrator or higher).

### Fix Option A — PIM
1. Entra → **Roles and administrators** → **Cloud Application Administrator** → **Activate**
2. Provide justification → wait for activation
3. Return and click Grant consent

### Fix Option B — Admin URL
Generate the consent URL and send to admin:
```
https://login.microsoftonline.com/<TENANT_ID>/adminconsent?client_id=<APP_ID>
```

---

## Issue 10: Agent Answers from General Knowledge (Not Grounded)

### Symptom
Agent answers correctly but without citations, or answers out-of-scope questions with general knowledge.

### Diagnosis Checklist

| Check | How to Verify | Fix |
|---|---|---|
| KB not attached | Agent config → Knowledge section | Attach the KB |
| Instructions too weak | Read the Instructions field | Add: "NEVER answer from training data" |
| KB not Active | Knowledge → KB status | Wait for provisioning or fix source connection |
| Source has no documents | AI Search → query `*` → 0 results | Re-run indexer or re-upload documents |

### Verification
1. In-scope question → grounded answer with citation
2. Out-of-scope → refusal message

---

## Issue 11: Knowledge Base Not Appearing in Agent Picker

### Symptom
KB picker is empty when trying to attach a KB to an agent.

### Root Cause
1. Created a **Knowledge Source** but not a **Knowledge Base** (they are different objects)
2. KB was created in a different project
3. KB is still provisioning

### Fix
1. Knowledge → Knowledge bases → verify KB exists and is **Active**
2. If missing: create one (Module 04, Step 4.1)
3. If in wrong project: switch projects using the breadcrumb

---

## Issue 12: Search Returns Zero Results After Indexing

### Symptom
Indexer shows success but search returns empty.

### Diagnosis
1. AI Search → **Indexers** → Documents processed = 0?
   - Fix: Check data source connection, verify files exist
2. Index fields → `content` not marked **Searchable**?
   - Fix: Recreate index with correct attributes
3. Wrong index selected in query?
   - Fix: Verify index name in search explorer

---

## Issue 13: Bing/Web Knowledge Source Not Working

### Symptom
Web knowledge source shows Active but agent doesn't return web results.

### Root Cause
1. Bing Search resource not created or API key expired
2. Agent instructions explicitly say "only use knowledge base" (blocking web source)

### Fix
1. Verify Bing Search resource exists in Azure Portal with valid key
2. Check agent instructions — allow web grounding for appropriate questions
3. Test Bing API directly: verify the key returns results

---

## Issue 14: Work IQ Returns No Results

### Symptom
Agent with Work IQ enabled returns "I don't have access to that information."

### Root Cause
1. M365 Copilot license not assigned
2. Graph API consent not granted
3. User has no data (empty inbox, no calendar events)

### Fix
1. Verify Copilot license: M365 admin center → Users → Licenses
2. Check consent: Entra → Enterprise apps → find the Foundry app → Permissions
3. Test with known data: send yourself a test email, then query

---

## Issue 15: SharePoint Remote — No Results

### Symptom
SharePoint Remote knowledge source is Active but returns no documents.

### Root Cause
1. User doesn't have access to the specified SharePoint site
2. Site URL is wrong
3. SharePoint search is not indexing content (crawl schedule)

### Fix
1. Verify user can access the SharePoint site directly in a browser
2. Verify site URL format: `https://<tenant>.sharepoint.com/sites/<sitename>`
3. In SharePoint admin center → check that the site's content is being crawled

---

## Quick Reference Card

### Two RBAC Flows (Print This)

```
Flow 1: INDEXING (Search → OpenAI)
  Where: Azure AI Services resource → IAM
  Role:  Cognitive Services OpenAI User
  Who:   AI Search system MI
  Picker: Resource type = "Search services"

Flow 2: RUNTIME (Foundry → Search)
  Where: AI Search resource → IAM
  Roles: Search Index Data Reader
         Search Index Data Contributor
         Search Service Contributor
  Who:   Foundry Project system MI
  Picker: Resource type = "Foundry"
```

### Knowledge Source vs. Knowledge Base

```
Source = raw connector → points to data
KB     = intelligent wrapper → adds model + reasoning + output mode

Agents attach to KBs, not Sources.
```

### Emergency Contacts

| Issue Type | Who to Contact |
|---|---|
| Azure subscription/access | Your Azure admin |
| Entra roles / admin consent | Your Entra admin |
| M365 / Copilot licensing | Your M365 admin |
| Workshop content issues | Repository maintainer |
