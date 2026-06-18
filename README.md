# Azure IAM & RBAC Lab — Scope Hierarchy and Least Privilege

> A hands-on Azure identity lab demonstrating role-based access control,
> scope hierarchy, and the Principle of Least Privilege using built-in roles,
> security groups, and three real-world user situations.

---
## What is this project?

I built a working identity environment in Azure to show how role-based access
control (RBAC) actually behaves in practice. I created three users, each in their
own security group, each given a different role at a different scope. The purpose
was to prove that the level at which a role is assigned (the scope) controls how
far that access reaches, and that choosing the right built-in role grants someone
exactly what they need (least privilege).

---
## The Architecture

Three personas, three groups, three scopes —

```
 ──────────────────────────────────────────────────────────────────────

  👤 Lab User 1   ──▶   👁️ Reader @ Subscription      ──▶   sees EVERYTHING (read-only)

  👤 Eddie        ──▶   🔧 Contributor @ rg-app-dev   ──▶   builds in APP only

  💼 Sheldon      ──▶   💷 Cost Mgmt @ finance-rg     ──▶   manages COSTS in finance only
```
---
## What I Used

**Subscription**
- Azure subscription 1

**Resource Groups**
- rg-app-dev — the application tier
- finance-rg — the finance tier

**Security Groups**
- Read-Only — Reader role at subscription scope
- App-Dev — Contributor role on rg-app-dev
- Finance-Cost-Management-Group — Cost Management Contributor on finance-rg

**Users**
- Lab User 1 — member of Read-Only, inherits read-only access across the whole subscription
- Eddie Murphy — member of App-Dev, can build/manage resources in rg-app-dev only
- Sheldon Kamakazi — member of Finance-Cost-Management-Group, can manage costs/budgets in finance-rg only

**Roles used** (all built-in)
- Reader
- Contributor
- Cost Management Contributor

Every role is assigned to a **group**, never directly to a user. Each user inherits access through group membership — the group-based access principle.

---

## What I Tested

I logged in as each user in a separate private browser window to prove the
boundaries from the user's own perspective, not just the admin view.

- ☑ **Test 1 —** Logged in as Sheldon. His Resource groups list showed only finance-rg ("Showing 1–1 of 1"). Every other resource group in the subscription was invisible to him.
- ☑ **Test 2 —** As Sheldon, opened finance-rg → Cost Management → Budgets and viewed the budget scoped to that resource group. His role works exactly where it's assigned.
- ☑ **Test 3 —** Logged in as Eddie. His Resource groups list showed only rg-app-dev — the mirror image of Sheldon. finance-rg was completely absent.
- ☑ **Test 4 —** As Eddie, created a storage account into rg-app-dev. Deployment succeeded — his Contributor role lets him provision real infrastructure in his own resource group.
- ❌ **Test 5 —** As Eddie, tried to create a resource into finance-rg. The resource group never even appeared in the dropdown — he can't select a resource group he has no role on. Access control hides it entirely rather than just blocking it.

---

## Problems I Hit and How I Fixed Them

During this lab there was only really one hurdle, which I'll go into detail about:

### Problem 1 — Storage account blocked by resource provider

**What happened:** As Eddie, creating a storage account failed with: "Microsoft.Storage is not registered for the subscription, and you don't have permissions to register a resource provider."

**Why it happened:** Two separate things. First, resource providers must be registered on a subscription before that resource type can be used, and on a new subscription many aren't. Second, registering a provider is a *subscription-level* action — and Eddie's Contributor role is scoped only to rg-app-dev, so he had no subscription-level rights to do it.

**How I fixed it:** I switched to my Owner account and registered Microsoft.Storage at the subscription level (Subscription → Resource providers → Register). Back as Eddie, the storage account then created successfully into rg-app-dev.

**What I learned:** Some resource types need to be registered at the subscription level before they can be used. Eddie couldn't do this himself — registering is a subscription-level action, and his role only reached the resource group. So this also showed his scope limit in action.

---

## What I Learned

**Scope determines radius** — The level a role is assigned at matters as much as the role itself. Lab User 1's Reader at subscription scope sees far more than Eddie's Contributor on a single resource group, even though Contributor is the "stronger" role.

**Scope limits visibility, not just actions** — A user assigned to one resource group doesn't just lack permission to change others — those resource groups don't even appear in their portal. Eddie couldn't see finance-rg and Sheldon couldn't see rg-app-dev. Least privilege working at the visibility level.

**Least Privilege through built-in roles** — I gave the finance persona Cost Management Contributor rather than Contributor. It grants exactly what a finance user needs (cost and budget management) and nothing more (no control over actual resources). Reaching for a narrow built-in role instead of a broad one is a core security habit.

**Group-based access** — Every role went on a group, never directly on a user. Users inherit access through membership. This scales: to onboard a new developer you add them to App-Dev, not re-assign roles one by one.

**Same level, different scope** — Eddie and Sheldon are both assigned at the resource-group level, but to *different* resource groups. A role scoped to rg-app-dev doesn't reach finance-rg and vice versa, so two users at the same tier stay completely isolated from each other.

---

## Screenshots

### Role assignments on rg-app-dev
![rg-app-dev-IAM](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/01%20-%20rg-app-dev%20Access%20control%20IAM%20.png?raw=true)

The App-Dev group has the Contributor role with Scope "This resource", while Read-Only's Reader shows Scope "Subscription (Inherited)". The contrast in the Scope column shows resource-group scope versus subscription inheritance at a glance.

### Role assignments on finance-rg
![finance-rg-IAM](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/02%20-%20rg-finance-Access%20control%20IAM%20.png?raw=true)

The Finance-Cost-Management-Group has Cost Management Contributor with Scope "This resource" — least privilege scoped to the finance tier only.

### Sheldon's access verified
![sheldon-assigments](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/03%20-%20Sheldon%20assignments%20on%20finance-rg%20.png?raw=true)

Check access for Sheldon Kamakazi shows Cost Management Contributor at Scope "This resource", and the Group assignment column confirms his access comes through the Finance-Cost-Management-Group rather than a direct assignment.

### Sheldon's view — sees only finance-rg
![sheldon-login](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/04%20-%20Sheldon%20view%20-%20Resource%20groups%20.png?raw=true)

Logged in as Sheldon, the Resource groups list shows "Showing 1–1 of 1" with only finance-rg visible. The rest of the subscription is completely invisible to him.

### Eddie's view — sees only rg-app-dev
![eddie-login](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/06%20-%20Eddie%20view%20-%20Resource%20groups%20.png?raw=true)

Logged in as Eddie, the Resource groups list shows only rg-app-dev — the mirror image of Sheldon. finance-rg is absent because he has no role there.

### The error Eddie hit
![storage-error](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/07%20-%20Eddie%20view%20-%20Storage%20Error.png?raw=true)

Creating a storage account as Eddie was blocked because Microsoft.Storage wasn't registered, and registering a provider is a subscription-level action his resource-group-scoped role can't perform.

### Resource provider registered (as Owner)
![resource-provider](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/08%20-%20Resource%20provider%20added.png?raw=true)

Switching to my Owner account, I registered Microsoft.Storage at the subscription level — the action Eddie couldn't perform, which unblocked storage creation across the subscription.

### Eddie creates a storage account in rg-app-dev
![storage-added](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/09-%20Eddie%20view%20-%20Storage%20account%20deployment%20succeeded%20in%20rg-app-dev.png?raw=true)

Back as Eddie, the storage account deployed successfully into rg-app-dev — proving his Contributor role lets him provision real resources in his own resource group.

### Lab User 1's view — sees everything (read-only)
![Labuser1-login](https://github.com/Awezkhan0/Azure-IAM-RBAC-Lab-Scope-Hierarchy-and-Least-Privilege/blob/main/Screenshots/10-%20Lab%20User%20view%20-%20Resource%20Groups.png?raw=true)

Logged in as Lab User 1, both rg-app-dev and finance-rg are visible. Reader at subscription scope reaches every resource group below it — the opposite of Eddie and Sheldon, who each see only one.

---

## Exam Relevance (AZ-104)

- **Manage identities and governance** — RBAC, built-in roles, role assignments at subscription vs resource group scope, security groups, and the Principle of Least Privilege.
- **Troubleshooting access** — distinguishing genuine RBAC denials from resource provider registration, a frequent real-world source of confusion.

---

*Part of my Azure cloud engineering learning journey alongside AZ-104 exam preparation.*
