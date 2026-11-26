# Session 5 – Security of Cloud Computing Services

**Identity and Access Management (IAM) across AWS, Azure and Google Cloud**
SGH · Cloud Computing · Academic year 2026/2027 · Study guide

---

## What you should be able to do after this session

1. Tell **authentication** apart from **authorization**, and explain why having an identity in the cloud is not the same as having access to a resource.
2. Design **least-privilege access** in AWS IAM, Microsoft Entra ID and Google Cloud IAM, and name the right mechanism in each of the three clouds.
3. Recognize the **most common IAM misconfigurations** before they turn into an incident.

If you only remember one sentence from this session, make it this one:

> **Permissions should be minimal and explicit, never maximal and implicit.**

The rest of this guide comes back to that idea in different forms.

---

## 1. The story to remember: Capital One, 2019

In 2019 an attacker stole the personal and financial records of **106 million people** from Capital One, one of the largest US banks. Not a single password was cracked.

Here is how it worked:

- Capital One ran a **web application firewall (WAF)** on an ordinary **EC2 instance**. The WAF was misconfigured and had a **server-side request forgery (SSRF)** flaw.
- An **IAM role** was attached to that instance. A role is a machine identity with a set of permissions, and this one had far more than the firewall needed, including full access to the **S3 buckets** holding customer data.
- Through the SSRF flaw, the attacker queried the **AWS instance metadata service** and asked for the role's credentials. The service handed them over.
- With those credentials she read the buckets.

The outcome was 106 million exposed records, a criminal conviction for the attacker and an **$80 million fine** from the US banking regulator (OCC).

**Why this case matters:** no encryption was broken. What failed was a **permissions configuration**. Cloud security is, to a very large extent, about who is allowed to do what.

---

## 2. Authentication vs. authorization

These two words get mixed up all the time in everyday language. In IAM they are two separate questions, answered by two separate systems.

| | Authentication | Authorization |
|---|---|---|
| Question | *Who are you?* | *What are you allowed to do?* |
| How it's checked | Password, hardware key, certificate, MFA code | Permissions checked against a specific resource |
| Who answers | The sign-in system | The IAM policy engine |

At Capital One the attacker didn't need to authenticate as a privileged administrator. She hijacked a machine identity whose **authorization** was too broad.

> **Core idea, version 1:** holding an identity does not mean holding access. Access has to be granted explicitly, by a separate mechanism.

---

## 3. The shared responsibility model through a security lens

You met this model in Session 1. Here it is again, applied to security.

- **Security *of* the cloud is the provider's job.** That covers physical data centers, hardware, virtualization and the global backbone network. You never configure or audit this layer.
- **Security *in* the cloud is your job.** That covers IAM configuration, data classification and encryption, network rules, and patching your own instances.

So who configured the over-permissioned role at Capital One? Not AWS. An architect on the bank's side did. Your provider will never log into your account to fix a permissions mistake. Only your own team can.

> **Core idea, version 2:** IAM is the part of cloud security the provider never takes off your hands. It is always your responsibility.

---

## 4. The threat landscape in 2026

The Cloud Security Alliance's *Top Threats to Cloud Computing* report (August 2026) surveyed 507 practitioners who defend cloud environments for a living. The top five:

| # | Threat | Score | Change |
|---|---|---|---|
| 1 | Inadequate identity and access management | 7.95 | up from #2 |
| 2 | AI-enhanced attacks | 7.91 | new |
| 3 | Insecure third-party resources (supply chain) | 7.85 | up from #5 |
| 4 | Insecure interfaces and APIs | 7.83 | |
| 5 | Misconfiguration and inadequate change control | 7.77 | down from #1 |

Further down the list: AI system compromise (new), advanced persistent threats, no cloud security strategy, insecure software development, accidental data disclosure, and system vulnerabilities.

Two things to notice. First, the scores are tightly packed (7.95 down to 7.45 across all eleven), so treat this as a list of problems that all matter, with identity just edging ahead. Second, for almost every item, the answer to "whose problem is this, the provider's or the customer's?" is the customer. Identity sits at the top, and only you can fix it.

---

## 5. The guiding principles

**Principle of least privilege.** Every identity gets exactly the permissions its task requires. Nothing extra "just in case", nothing "because it might come in handy".

**Zero trust.** "Never trust, always verify." Being on the corporate network, or simply having an account, is no longer proof that you should be trusted. Every request is checked.

**Just-in-time (JIT) access.** Elevated permissions are granted temporarily, for the duration of a specific task, and then removed. They are not handed out permanently.

> **Core idea, version 3:** access is denied by default. Every permission must be granted explicitly. Nothing is implicit.

---

## 6. The anatomy of IAM: three building blocks

AWS, Azure and Google Cloud use different names, but every IAM system is built from the same three pieces:

| Block | Question | Examples |
|---|---|---|
| **Identity** | WHO? | A user, a group, or a machine identity (an application, a CI/CD pipeline, a service) |
| **Policy / permission** | WHAT? | A set of allowed (or denied) actions: read, write, delete… |
| **Resource** | ON WHAT? | A specific bucket, virtual machine or database |

Every IAM policy statement in every cloud answers these three questions. The syntax differs between clouds. The logic doesn't.

---

## 7. AWS Identity and Access Management (IAM)

**Users vs. roles.** An **IAM user** is a persistent identity (a person or an application) with standing credentials. An **IAM role** is a temporary identity that a service or a person "borrows" for the duration of a task.

**JSON policies.** Permissions are written as JSON documents made of `Allow` / `Deny` statements. Each statement names the actions, the resources and, optionally, conditions.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": "arn:aws:s3:::team-dev-bucket/*"
  }]
}
```

This policy is least privilege in practice. It doesn't grant "access to S3". It grants two specific actions on one specific bucket. Had the Capital One firewall role been written like this, the attacker would have reached one irrelevant bucket and nothing else.

**The root account** has unrestricted permissions. Don't use it for daily work. Lock it away behind MFA and keep it for the few tasks that require it by name.

**MFA everywhere it can go**, especially on privileged accounts. It's a single, cheap barrier that would have stopped many well-known incidents.

---

## 8. Microsoft Entra ID (formerly Azure Active Directory)

If older materials say "Azure AD", they mean the same service. It was renamed to Microsoft Entra ID.

- **Tenant** – a dedicated, isolated instance of your organization's identity directory. Every account and group starts here.
- **Users and groups** – managed centrally in the tenant, often synchronized with the organization's on-premises Active Directory.
- **RBAC roles** (role-based access control) – built-in roles such as **Owner**, **Contributor** and **Reader**, assigned at a **scope**: a subscription, a resource group or a single resource.
- **Conditional Access** – an extra layer in which context decides access: location, device, or the sign-in risk score.

**Key difference from AWS:** in Azure you assign a role to a *place in the resource hierarchy*, not just to an identity, and permissions are inherited downward from there.

---

## 9. Google Cloud IAM

- **Members (principals)** – a Google Account, a Google group, a service account, or an entire domain.
- **Roles come in three tiers:**
  - *basic* – very broad (Owner, Editor, Viewer)
  - *predefined* – fine-grained roles for individual services, maintained by Google
  - *custom* – roles you build yourself for an exact set of permissions
- **Resource hierarchy:** Organization → Folder → Project → Resource. Permissions granted higher up flow automatically down the whole structure.

That hierarchy is powerful, because one decision at the organization level can cover hundreds of projects. It's also the most common source of over-permissioning in GCP. Someone grants a role "at the top" to save time, and suddenly half the company has access it doesn't need.

---

## 10. Cheat sheet: same concept, different names

| Concept | AWS | Microsoft Entra ID / Azure | Google Cloud |
|---|---|---|---|
| Personal identity | IAM user | User (Entra ID) | Google Account |
| Group | IAM group | Group (Entra ID) | Google Group |
| Bundle of permissions | IAM policy (JSON) | RBAC role | IAM role |
| **Machine identity** | IAM role / instance profile | Managed Identity | Service account |
| Multi-factor authentication | MFA (virtual/hardware) | MFA / Conditional Access | 2-Step Verification |
| Resource hierarchy | Account → AWS Organizations | Tenant → management group → subscription | Organization → folder → project |

Pay special attention to the **machine identity** row. That's the kind of identity that failed at Capital One, and the names are easy to mix up under exam pressure.

---

## 11. Machine identities: most of your identities aren't people

In a typical cloud account, most identities are applications, CI/CD pipelines, scheduled jobs and containers. They have to prove who they are just like people do. There are three ways to do it:

| | Static keys | Attached identity | Federated identity |
|---|---|---|---|
| Verdict | **Avoid** | **Default choice** | **For workloads outside the cloud** |
| How it works | A long-lived secret stored somewhere | The platform gives the resource short-lived credentials and rotates them itself | An external token (e.g. from GitHub Actions or Kubernetes) is exchanged for credentials that expire within the hour |
| Risk | Valid until someone rotates it, which in practice nobody does. Leaks via Git repos, laptop backups, Slack messages | Nobody ever sees a secret | No stored secret on the outside system |
| AWS | Access key | IAM role / instance profile | IAM OIDC provider |
| Azure | Client secret | Managed identity | Entra workload identity federation |
| GCP | Service account key (JSON) | Attached service account | Workload Identity Federation |

**The uncomfortable twist:** Capital One's firewall *already* used an attached role, which is the good option, and the leaked credentials were short-lived. It didn't help, because the role was allowed to read everything.

- Short-lived credentials limit **how long** an attacker has.
- Least privilege limits **what** they can do.

You need both.

---

## 12. Federation: one identity, three clouds

How do people get into three clouds without three passwords? Through **federation**.

Your company already has one **identity provider** (Entra ID, Okta or Google Workspace). Password, MFA and device checks happen there, once. The clouds trust that provider over **SAML** or **OpenID Connect (OIDC)** and never keep their own copy of your password.

| Cloud | How federation is set up | Where permissions land |
|---|---|---|
| AWS | IAM Identity Center | Permission sets mapped to AWS accounts |
| Azure | Built in (Entra ID *is* the directory) | RBAC role assignments on subscriptions and resource groups |
| Google Cloud | Cloud Identity, or Workforce Identity Federation | IAM role bindings on projects and folders |

**Why a business student should care:** when someone leaves the company, IT switches off one account and access disappears from all three clouds at once. Without federation, someone has to remember three consoles, and sooner or later they forget one.

---

## 13. Live demo recap: the same read-only access, built three times

**Goal:** one user may read files in one storage bucket. Nothing else. Then prove it.

| AWS | Azure | Google Cloud |
|---|---|---|
| Create a role with `s3:GetObject` on one bucket | Assign **Storage Blob Data Reader** | Grant `roles/storage.objectViewer` on one bucket |
| Test with the **IAM policy simulator**: `GetObject` allowed, `DeleteObject` implicitly denied | Scope it to **one container**, not the whole subscription | Ask **Policy Troubleshooter** whether the user can delete: *denied* |
| Ask **IAM Access Analyzer** what's unused | Verify with **Check access** in the portal | Review the **IAM recommendations** |

**The takeaway:** in AWS the permission sits on the **identity**. In Azure and Google Cloud it sits on a **place in the resource hierarchy**.

---

## 14. Five common IAM misconfigurations

1. **Using the root / owner account for daily work.** Keep root for the handful of tasks that require it by name. Everything else should run from a scoped-down account.
2. **Wildcard permissions** (`"Action": "*"`, `"Resource": "*"`, or Owner for everyone "to keep things simple"). Convenient while testing, dangerous in production. This is exactly the pattern behind Capital One.
3. **Long-lived, static access keys.** A key that never expires and never rotates is a standing invitation. Prefer short-lived credentials or federation.
4. **Privileged accounts without MFA.** One phished password should never be enough to reach an admin role. Require a second factor, ideally a phishing-resistant one such as a hardware key or a passkey.
5. **Identities nobody owns any more.** These are accounts of people who left, test service accounts, and keys from finished projects. Nobody notices them, which is what makes them useful to attackers. Every cloud can flag unused access: **IAM Access Analyzer** (AWS), **access reviews** (Entra ID), **IAM recommendations** (Google Cloud).

---

## 15. Case study: anatomy of an exposed VM

The session ended with a demo on a real virtual machine (`insecure-demo-vm`, zone `europe-west1-b`). It was deliberately configured for "convenience" and exposed to the internet with a firewall rule open to `0.0.0.0/0`. The numbers below are illustrative but typical for an open SSH port.

### What the internet did in about 30 minutes

| Metric | Value |
|---|---|
| First scan after exposure | **4 minutes** |
| Failed login attempts | **1,352** in a 32-minute window |
| Unique source IP addresses | **73**, from about 18 countries |
| Root takeover | **11 minutes**, via weak password + passwordless sudo |
| Peak traffic | 131 failed attempts per minute |

None of it was a targeted attack. This is the background noise of the internet, and it reaches every open port. The most-guessed usernames were `root`, `admin`, `test`, `ubuntu` and `user`. `admin` was a real account on this VM, so the bots hit it blind simply because it's on every dictionary list.

### The attack surface

Two ports were open to the world: **22/tcp** (SSH, where passwords get guessed) and **80/tcp** (an HTTP panel without TLS, which adds attack surface). On top of that, four settings let attackers in:

| Setting | Why it's dangerous |
|---|---|
| `PasswordAuthentication yes` | SSH accepts passwords, so they can be guessed |
| `Password123` | A dictionary password, cracked in seconds |
| `sudo NOPASSWD:ALL` | The `admin` account gets root without being asked for a password |
| `PermitRootLogin yes` | You can log in directly as root |

None of these is a bug in the code. They are options switched on for convenience. The problem is the combination.

### The takeover chain: five steps, no exploit

1. **`0.0.0.0/0`** – every address on the internet can reach ports 22 and 80.
2. **Password authentication** – SSH lets anyone keep guessing passwords.
3. **`Password123`** – a dictionary attack finds it in seconds.
4. **`NOPASSWD` sudo** – the `admin` account becomes root without a password.
5. **Root login** – full, persistent control of the machine.

The result was a full takeover in about 11 minutes, achieved purely by guessing a password.

A side note on what an attacker does next with root: they can read **`/etc/shadow`**, the Linux file that stores password hashes for local accounts and is readable only by root. With those hashes they can crack passwords **offline**, with no attempt limits and no traces in the SSH logs. If the same password is reused anywhere else, the breach spreads. Accounts that use SSH keys only have `!` or `*` in place of a hash, so there's nothing to crack.

### How to close it

| Setting | Wrong (demo) | Right |
|---|---|---|
| Network access | `sourceRanges: 0.0.0.0/0` | Your own IP only, or IAP TCP forwarding |
| SSH sign-in | `PasswordAuthentication yes` | SSH keys only |
| Root over SSH | `PermitRootLogin yes` | `PermitRootLogin no` |
| sudo rights | `NOPASSWD:ALL` | sudo with a password, least privilege |
| Account password | `Password123` | No passwords; MFA / password manager |
| Panel on :80 | Open to `0.0.0.0/0` | Behind a VPN / internal only |

Each change in the right-hand column breaks one link of the chain. The two that remove most of the risk on their own are **restricting network access** and **using keys instead of passwords**. The demo finished by deleting the VM and its firewall rule, so nothing was left exposed.

---

## 16. One idea, said three times

- **In words:** permissions should be minimal and explicit, never maximal and implicit.
- **As a diagram:** Identity → Policy → Resource. Every arrow must be defined explicitly. If an arrow is missing, there is no access at all.
- **As an analogy:** IAM works like hotel key cards. Each card opens only the rooms it should (the guest's room, maybe the gym, never other guests' rooms or the back office). Any card can be revoked instantly without changing the locks.

---

## Practice exercises

### Exercise A – Design access for a three-person startup

A small team is launching a cloud project:

- The **developer** needs full access, but only to the DEV environment.
- The **ops person** needs read-only access to billing data.
- The **CI/CD pipeline** needs deploy-only permissions in production: no read, no delete.

For each need, name the right IAM mechanism in AWS, Azure and GCP.

<details>
<summary>Model answer</summary>

| Need | AWS | Azure | Google Cloud |
|---|---|---|---|
| Developer, DEV only | Policy with a condition on a resource tag (e.g. `env=dev`) | RBAC role scoped to the *dev* resource group | Role granted at the *dev* folder level |
| Ops, billing read-only | AWS Billing ReadOnlyAccess | Billing Reader role | Billing Account Viewer role |
| CI/CD, deploy only | IAM role assumed by the pipeline, with deploy actions only (ideally via OIDC federation) | Managed Identity with a custom role | Service account with the Cloud Deploy role |

</details>

### Exercise B – Spot the flaws: this policy passed code review

A web shop's product page runs on EC2. All it needs is to read product photos from one bucket, `sgh-shop-photos`. This policy is attached to its role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": "s3:*",          "Resource": "*" },
    { "Effect": "Allow", "Action": "iam:PassRole",  "Resource": "*" }
  ]
}
```

1. Find three things wrong with it.
2. Rewrite the first statement so it allows only what the shop needs.

*Hint: one line is worse than it looks.*

<details>
<summary>Model answer</summary>

1. **`"s3:*"`** allows every S3 action, including deleting objects, deleting buckets and changing bucket policies. The shop only needs `s3:GetObject`.
2. **`"Resource": "*"`** covers every bucket in the account, not just `sgh-shop-photos`. That's the Capital One pattern again.
3. **`iam:PassRole` on `"*"`** is the hidden one. It lets the app hand *any* role in the account to a new EC2 instance or Lambda function. If an admin role exists, whoever controls the web app can effectively become admin. This is **privilege escalation**.
4. *Bonus:* there are no conditions at all, for example no restriction to the company network.

Corrected policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::sgh-shop-photos/*"
  }]
}
```

The second statement is simply deleted.

</details>

---

## Self-check questions

1. What is the difference between authentication and authorization? Which one failed at Capital One?
2. Under the shared responsibility model, who is responsible for IAM configuration, and why?
3. What does "access is denied by default" mean in practice?
4. Name the three building blocks of every IAM system.
5. What's the difference between an IAM user and an IAM role in AWS?
6. Where does a permission "land" in AWS, and where does it land in Azure and Google Cloud?
7. Why is granting a role at the organization level in GCP risky?
8. Give the name of a machine identity in each of the three clouds.
9. Capital One's role used short-lived credentials. Why didn't that prevent the breach?
10. Why does federation make it safer when an employee leaves?
11. Which two changes would have removed most of the risk from the exposed VM?
12. Why is `iam:PassRole` on `"*"` dangerous?

<details>
<summary>Short answers</summary>

1. Authentication = *who are you*; authorization = *what are you allowed to do*. Authorization failed: the role's permissions were far too broad.
2. The customer. It's security *in* the cloud, and the provider never configures your permissions for you.
3. No identity can do anything until a permission is explicitly granted; anything not granted is refused.
4. Identity, policy/permission, resource.
5. A user is a persistent identity with standing credentials; a role is a temporary identity borrowed for a task.
6. AWS: on the identity. Azure and GCP: on a place (scope) in the resource hierarchy.
7. Permissions inherit downward, so one grant reaches every folder and project beneath it.
8. AWS: IAM role / instance profile; Azure: Managed Identity; GCP: service account.
9. Short-lived credentials limit *how long* an attacker has access, not *what* they can do. The role could read everything.
10. Access is switched off in one place (the identity provider) and disappears from all clouds at once, so no forgotten accounts are left behind.
11. Restricting network access (own IP or IAP instead of `0.0.0.0/0`) and using SSH keys instead of passwords.
12. It lets the holder attach any role, including an admin role, to a new resource, which is a direct path to privilege escalation.

</details>

---

## Glossary

| Term | Meaning |
|---|---|
| **IAM** | Identity and Access Management: the system that decides who can do what on which resource |
| **Authentication (AuthN)** | Proving who you are |
| **Authorization (AuthZ)** | Checking what you're allowed to do |
| **MFA** | Multi-factor authentication: a second proof of identity on top of a password |
| **Least privilege** | Granting only the permissions a task requires |
| **Zero trust** | A security model in which nothing is trusted by default and every request is verified |
| **JIT access** | Elevated permissions granted temporarily, only for the duration of a task |
| **Principal** | The identity a policy applies to (user, group, role, service account) |
| **Machine identity** | An identity used by software rather than a person |
| **Federation** | Trusting an external identity provider instead of keeping separate accounts in each system |
| **SAML / OIDC** | Standard protocols used for federated sign-in |
| **SSRF** | Server-side request forgery: tricking a server into making requests on the attacker's behalf |
| **Instance metadata service** | An internal endpoint a VM uses to learn about itself, including its role credentials |
| **Privilege escalation** | Gaining more permissions than you were originally given |
| **`/etc/shadow`** | The Linux file that stores password hashes for local accounts, readable only by root |

---

**Next session (Session 6):** we know *who* can get in and *what* they can do. But where does the data actually sit once a virtual machine is stopped or deleted? We'll look at virtual disks: Amazon EBS, Azure Disk Storage and Google Persistent Disk.
