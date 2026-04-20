# Module 05 — Extend to Work IQ

## What You Are Building
Extending your Foundry agent to access **Microsoft 365 organizational data** — emails, calendar events, Teams messages, OneDrive files, and People profiles — through **Work IQ** (formerly Microsoft 365 Copilot knowledge). This gives your agent enterprise context beyond static documents.

## Scenario Context
> Contoso HR wants the onboarding assistant to go beyond static policies. New hires should be able to ask:
> - "Who is my manager and what's their email?"
> - "What meetings do I have scheduled this week?"
> - "Show me the onboarding deck that was shared in the HR Teams channel"
> - "Find the benefits enrollment form from my inbox"

---

## Understanding Work IQ

### What Work IQ Provides

Work IQ connects your Foundry agent to the **Microsoft Graph** — the unified API for M365 data.

| Data Domain | What the Agent Can Access | Graph API |
|---|---|---|
| **People** | Org chart, profiles, contact info, reporting structure | `/users`, `/me/people` |
| **Email** | Inbox messages, sent items, search across mail | `/me/messages` |
| **Calendar** | Events, meetings, availability | `/me/events` |
| **Teams** | Channel messages, chat history, shared files | `/teams/.../messages` |
| **OneDrive** | Files, shared documents, recent files | `/me/drive` |
| **SharePoint** | Site content, lists, document libraries | `/sites` |

### Work IQ vs. Foundry IQ

| Aspect | Foundry IQ | Work IQ |
|---|---|---|
| **Data source** | Your custom data (AI Search, Blob, SharePoint indexed) | Microsoft 365 organizational data |
| **Access model** | Application-level or managed identity | Delegated (user's own M365 permissions) |
| **Security** | RBAC on Azure resources | M365 security trimming — users only see their own data |
| **Setup** | Knowledge sources + KB configuration | M365 Copilot license + Graph permissions |
| **Best for** | Expert knowledge from custom documents | Personal/organizational context |

> **Key insight:** You can use BOTH Foundry IQ and Work IQ together. The agent can answer "What is the dress code?" from Foundry IQ (company docs) and "Who sent me the latest onboarding email?" from Work IQ (user's inbox).

---

## Prerequisites

- [ ] Microsoft 365 E3/E5 or Business Premium license
- [ ] Microsoft 365 Copilot license assigned to the user
- [ ] Foundry agent created (Module 03)
- [ ] Admin consent for Graph API permissions (or self-service consent enabled)

---

## Step 5.1: Enable Work IQ in Your Foundry Project

### Option A: Through Foundry Agent Configuration

1. In **https://ai.azure.com** → your project → **Agents**
2. Click on your `contoso-hr-onboarding` agent
3. In the **Tools** or **Knowledge** section, look for **Work IQ** or **Microsoft 365**
4. Click **Add** → **Work IQ** (or **Microsoft 365 Knowledge**)
5. You will be prompted to:
   - Sign in with your M365 account (delegated authentication)
   - Grant consent for the agent to access your M365 data
6. Select the data domains to enable:
   - [ ] People & Org
   - [ ] Email
   - [ ] Calendar
   - [ ] Teams
   - [ ] OneDrive / Files
7. Click **Save**

### Option B: Through Declarative Agent in Copilot Studio

For organizations using Microsoft 365 Copilot extensibility:

1. Go to **https://copilotstudio.microsoft.com**
2. Create a new **Declarative Agent**
3. Under **Knowledge**, add:
   - Your Foundry agent as a backend (via API plugin or custom connector)
   - Microsoft 365 Graph data as a knowledge source
4. This creates a Copilot extension that surfaces in Microsoft 365 Copilot

> **When to use Option B:** When you want the agent to live inside Microsoft 365 Copilot chat (Teams, Outlook, etc.) rather than in the Foundry playground.

---

## Step 5.2: Configure Graph API Permissions

For Work IQ to access M365 data, the appropriate Graph API permissions must be consented.

### Delegated Permissions (User-Level Access)

| Permission | What It Allows | Data Domain |
|---|---|---|
| `User.Read` | Read user's own profile | People |
| `People.Read` | Read user's relevant people | People |
| `Mail.Read` | Read user's mailbox | Email |
| `Calendars.Read` | Read user's calendar | Calendar |
| `ChannelMessage.Read.All` | Read Teams channel messages | Teams |
| `Files.Read.All` | Read user's OneDrive and shared files | Files |

### How Consent Works

**If your org allows self-service consent:**
- The user is prompted at sign-in and can consent themselves

**If your org requires admin consent:**
1. Open **https://entra.microsoft.com**
2. **Identity** → **Applications** → **Enterprise applications**
3. Find the Foundry/Work IQ application
4. Click **Permissions** → **Grant admin consent**
5. Review and approve the requested permissions

---

## Step 5.3: Test Work IQ Queries

In the agent playground, test each data domain:

### Test 1: People & Org
```
Who is my manager?
```
**Expected:** Agent returns manager's name, title, email from the org chart.

### Test 2: Email
```
Show me the most recent email about onboarding.
```
**Expected:** Agent searches user's inbox, returns email subject, sender, summary.

### Test 3: Calendar
```
What meetings do I have tomorrow?
```
**Expected:** Agent returns meeting titles, times, and attendees from user's calendar.

### Test 4: Teams
```
Find the latest message in the HR Team general channel about benefits enrollment.
```
**Expected:** Agent searches Teams messages, returns relevant content.

### Test 5: Files
```
Find the employee handbook that was shared with me on OneDrive.
```
**Expected:** Agent searches OneDrive/shared files, returns file name and summary.

### Test Results

| Test | Domain | Expected | Actual | Pass? |
|---|---|---|---|---|
| Manager lookup | People | Name + email | ________________ | ☐ |
| Recent email | Email | Subject + sender + summary | ________________ | ☐ |
| Calendar | Calendar | Meetings + times | ________________ | ☐ |
| Teams message | Teams | Channel message content | ________________ | ☐ |
| File search | Files | File name + summary | ________________ | ☐ |

---

## Step 5.4: Combine Foundry IQ + Work IQ

The real power is combining both:

### Test: Cross-Source Query

```
What is the company PTO policy and do I have any upcoming meetings about time-off requests?
```

**Expected behavior:**
1. **Foundry IQ** retrieves the PTO policy from your knowledge base (company docs)
2. **Work IQ** checks the user's calendar for relevant meetings
3. Agent synthesizes both into a single response

### Test: Contextual Follow-Up

```
User: What is the benefits enrollment deadline?
Agent: [Answers from Foundry IQ - company docs]

User: Can you find if HR sent me an email about this?
Agent: [Answers from Work IQ - searches inbox]
```

---

## Step 5.5: Security and Data Boundaries

### What Users Can See

Work IQ respects Microsoft 365 security trimming:
- Users only see **their own** emails, calendar, and files
- Shared content only appears if the user already has permission
- Admins cannot use Work IQ to access other users' data through the agent

### What the Agent Can Access vs. Cannot

| Can Access | Cannot Access |
|---|---|
| User's own mailbox | Other users' mailboxes |
| Meetings user is invited to | Private meetings of others |
| Files shared with the user | Files with restricted access |
| Teams channels user is a member of | Private channels user isn't in |
| Public org chart info | Privileged HR/admin data (unless user has access) |

### Data Residency

Work IQ data stays within your Microsoft 365 tenant boundaries. The Foundry agent processes queries but does not store M365 data in the AI Search index or Blob storage.

---

## Step 5.6: Production Considerations

### When to Use Work IQ

| Use Case | Foundry IQ | Work IQ | Both |
|---|---|---|---|
| Policy Q&A from internal docs | ✅ | | |
| "Who is my manager?" | | ✅ | |
| "Find the latest HR announcement" | | ✅ | |
| "What is the dress code and when is my next HR meeting?" | | | ✅ |
| Customer-facing FAQ bot | ✅ | | |
| Internal employee assistant | | | ✅ |

### Licensing Requirements

| Feature | License Needed |
|---|---|
| Foundry IQ (knowledge bases) | Azure subscription + AI Foundry |
| Work IQ (M365 data) | Microsoft 365 E3/E5 + **Copilot license** |
| Copilot Studio extension | Microsoft 365 Copilot + Copilot Studio |

---

## Verification — You Are Done When

- [ ] Work IQ is enabled on your agent
- [ ] At least 2 data domains (People, Email, Calendar) return results
- [ ] Combined Foundry IQ + Work IQ query produces a unified answer
- [ ] Confirmed security trimming works (user only sees own data)

---

## What's Next?

You have built a complete enterprise AI agent that:
1. ✅ Connects to 5 types of knowledge sources (Module 02)
2. ✅ Answers from grounded enterprise documents with citations (Module 03)
3. ✅ Uses multi-source knowledge bases with intelligent retrieval (Module 04)
4. ✅ Accesses Microsoft 365 organizational data (Module 05)

### Recommended Next Steps

| Topic | Where to Learn More |
|---|---|
| **Code-first agents** | Use the Foundry SDK to build agents programmatically |
| **Agent evaluation** | Use Foundry evaluation tools to measure answer quality |
| **Deployment with azd** | Deploy your agent to production with Azure Developer CLI |
| **Observability** | Monitor agent performance with Application Insights |
| **Red teaming** | Test your agent against adversarial prompts |
| **CI/CD** | Automate agent deployment with GitHub Actions |

→ [06-Checkpoint-Questions.md](06-Checkpoint-Questions.md) — Verify your learning with module-by-module questions.
