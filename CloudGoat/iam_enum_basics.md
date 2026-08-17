# CloudGoat — iam_enum_basics

> **Platform:** CloudGoat (Rhino Security Labs)  
> **Scenario:** iam_enum_basics  
> **Category:** AWS IAM Enumeration  
> **Concepts:** IAM Users · IAM Roles · Inline Policies · Managed Policies · IAM Groups · Policy Documents · Tags as Flag Locations  
> **Flags:** 5

---

## Scenario Overview

CloudGoat is an intentionally vulnerable AWS environment by Rhino Security Labs, deployed via Terraform. The `iam_enum_basics` scenario provisions a low-privilege IAM user (`cg-bob`) and hides five flags across different IAM primitives — role tags, inline policy Sids, group paths, policy descriptions, and policy document resource ARNs. The objective is to enumerate everything the user can see and extract all five flags.

---

## Setup

```bash
cloudgoat create iam_enum_basics
# Terraform initializes and provisions the scenario
# Profile configured: iam_enum_basic
```

![CloudGoat scenario deploy and Terraform init](./Images/page-1.jpg)

---

## Step 1 — Establish Identity

```bash
aws sts get-caller-identity --profile iam_enum_basic
```

```json
{
  "UserId":  "AIDAR5E6LTLPA2GP7KFCU",
  "Account": "131328613086",
  "Arn":     "arn:aws:iam::131328613086:user/cg-bob-cgidgq7njp9v9w"
}
```

Starting user is `cg-bob`. Account ID is `131328613086` — needed for constructing ARNs throughout.

---

## Step 2 — List All IAM Users

```bash
aws iam list-users --profile iam_enum_basic
```

Two users found:

| Username | ARN |
|----------|-----|
| `cg-bob-cgidgq7njp9v9w` | `arn:aws:iam::131328613086:user/cg-bob-cgidgq7njp9v9w` |
| `cloudgoat-admin` | `arn:aws:iam::131328613086:user/cloudgoat-admin` |

![list-users output showing cg-bob and cloudgoat-admin](./Images/page-2.jpg)

---

## Step 3 — List All IAM Roles

```bash
aws iam list-roles --profile iam_enum_basic
```

Several roles found including AWS service roles and one suspicious custom role:

- `AWSServiceRoleForResourceExplorer`
- `AWSServiceRoleForSupport`
- **`cg-flag4-role-cgidgq7njp9v9w`** ← non-AWS name, investigate this

![list-roles output showing the flag role](./Images/page-2.jpg)

---

## Step 4 — Inspect the Suspicious Role → Flag 1

The `cg-flag4` naming convention hints a flag is embedded in or around this role.

```bash
aws iam get-role \
  --role-name cg-flag4-role-cgidgq7njp9v9w \
  --profile iam_enum_basic
```

The full role object includes a `Tags` array. Buried inside:

```json
{
  "Key":   "Flag",
  "Value": "HSM-r0l3_trus1_f0und"
}
```

**Flag 1 found in role tags.**

> Always check resource tags during IAM enumeration. Tags are often used for metadata but are equally accessible to any principal with `iam:GetRole`.

![get-role output with flag in Tags](./Images/page-3.jpg)

```
Flag 1: HSM-r0l3_trus1_f0und
```

The trust policy on this role also reveals that `cg-bob` is the only principal allowed to assume it — confirming this role is part of the challenge design.

---

## Step 5 — Enumerate Inline Policies on cg-bob → Flag 2

Inline policies are attached directly to a user and are not visible in the general policy list. They require a separate API call per user.

```bash
aws iam list-user-policies \
  --user-name cg-bob-cgidgq7njp9v9w \
  --profile iam_enum_basic
```

```json
{
  "PolicyNames": ["cg-flag2-inline-policy-cgidgq7njp9v9w"]
}
```

An inline policy exists. Fetch its document:

```bash
aws iam get-user-policy \
  --policy-name cg-flag2-inline-policy-cgidgq7njp9v9w \
  --user-name cg-bob-cgidgq7njp9v9w \
  --profile iam_enum_basic
```

```json
{
  "Statement": [{
    "Action":   "ec2:DescribeInstances",
    "Effect":   "Allow",
    "Resource": "*",
    "Sid":      "HSM1nl1n3p0l1cyd1sc0v3r3d"
  }]
}
```

**Flag 2 is the policy statement Sid.**

> The `Sid` (Statement ID) field in a policy statement is a free-text identifier. It is visible to anyone who can read the policy — another creative hiding spot for a flag.

![get-user-policy showing Sid as flag](./Images/page-4.jpg)

```
Flag 2: HSM1nl1n3p0l1cyd1sc0v3r3d
```

---

## Step 6 — Enumerate IAM Groups → Flag 3

```bash
aws iam list-groups --profile iam_enum_basic
```

```json
{
  "Groups": [{
    "Path":      "/HSM_gr0up_m3mb3rsh1p_f0und/",
    "GroupName": "cg-flag3-group-cgidgq7njp9v9w",
    "Arn":       "arn:aws:iam::131328613086:group/HSM_gr0up_m3mb3rsh1p_f0und/cg-flag3-group-cgidgq7njp9v9w"
  }]
}
```

**Flag 3 is embedded in the group Path field.**

> IAM resource paths are hierarchical prefixes used to organize resources (e.g., `/engineering/`, `/prod/`). They are visible to any principal with list permissions and are often overlooked during enumeration.

![list-groups output with flag in path](./Images/page-4.jpg)

```
Flag 3: /HSM_gr0up_m3mb3rsh1p_f0und/
```

---

## Step 7 — Enumerate Managed Policies on cg-bob → Flag 4

Managed policies are standalone policy objects that can be attached to multiple principals. They appear separately from inline policies.

```bash
aws iam list-attached-user-policies \
  --user-name cg-bob-cgidgq7njp9v9w \
  --profile iam_enum_basic
```

```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "cg-flag1-managed-policy-cgidgq7njp9v9w",
      "PolicyArn":  "arn:aws:iam::131328613086:policy/cg-flag1-managed-policy-cgidgq7njp9v9w"
    },
    {
      "PolicyName": "IAMReadOnlyAccess",
      "PolicyArn":  "arn:aws:iam::aws:policy/IAMReadOnlyAccess"
    }
  ]
}
```

The `IAMReadOnlyAccess` managed policy is what gives `cg-bob` permission to run all these `list-*` and `get-*` IAM calls. The custom `cg-flag1-managed-policy` is the target.

```bash
aws iam get-policy \
  --policy-arn arn:aws:iam::131328613086:policy/cg-flag1-managed-policy-cgidgq7njp9v9w \
  --profile iam_enum_basic
```

```json
{
  "PolicyName":       "cg-flag1-managed-policy-cgidgq7njp9v9w",
  "DefaultVersionId": "v1",
  "Description":      "HSM{m4n4g3d_p0l1cy_m4st3r}"
}
```

**Flag 4 is in the policy Description field.**

![get-policy output with flag in description](./Images/page-5.jpg)

```
Flag 4: HSM{m4n4g3d_p0l1cy_m4st3r}
```

---

## Step 8 — Read the Policy Document → Flag 5

The policy metadata gives the `DefaultVersionId` as `v1`. Reading the actual policy document:

```bash
aws iam get-policy-version \
  --policy-arn arn:aws:iam::131328613086:policy/cg-flag1-managed-policy-cgidgq7njp9v9w \
  --version-id v1 \
  --profile iam_enum_basic
```

```json
{
  "Statement": [{
    "Action":   ["s3:ListBucket", "s3:GetObject"],
    "Effect":   "Allow",
    "Resource": "arn:aws:s3:::HSM{s3cr3t_js0n_str1ng}"
  }]
}
```

**Flag 5 is embedded inside the Resource ARN of the policy statement.**

> A policy Resource field is a string and can contain arbitrary text as long as it parses as a valid ARN. Here the bucket name in the ARN is the flag itself.

![get-policy-version with flag in Resource ARN](./Images/page-5.jpg)

```
Flag 5: HSM{s3cr3t_js0n_str1ng}
```

---

## All Flags

| # | Location | IAM Primitive | Flag |
|---|----------|---------------|------|
| 1 | Role Tags | `cg-flag4-role` | `HSM-r0l3_trus1_f0und` |
| 2 | Policy Statement Sid | Inline policy on cg-bob | `HSM1nl1n3p0l1cyd1sc0v3r3d` |
| 3 | Group Path | `cg-flag3-group` | `/HSM_gr0up_m3mb3rsh1p_f0und/` |
| 4 | Policy Description | `cg-flag1-managed-policy` | `HSM{m4n4g3d_p0l1cy_m4st3r}` |
| 5 | Policy Document Resource ARN | `cg-flag1-managed-policy` v1 | `HSM{s3cr3t_js0n_str1ng}` |

---

## IAM Enumeration Methodology — Checklist

```
aws sts get-caller-identity              → who am I
aws iam list-users                       → who else exists
aws iam list-roles                       → what roles exist, check trust policies
aws iam get-role --role-name <name>      → tags, trust policy details
aws iam list-groups                      → groups and their paths
aws iam list-group-policies              → inline policies on groups
aws iam list-attached-group-policies     → managed policies on groups
aws iam list-user-policies               → inline policies on users
aws iam get-user-policy                  → read inline policy document (check Sid)
aws iam list-attached-user-policies      → managed policies on users
aws iam get-policy                       → description, default version
aws iam get-policy-version               → actual permission statements
```

Every field in every IAM object is readable by any principal with `IAMReadOnlyAccess`. Tags, Sids, paths, descriptions, and resource ARNs are all data — and in a real environment, misconfigurations, secrets, or intelligence about the account structure can be found in any of them.

---

## Lesson Learned

`IAMReadOnlyAccess` feels harmless. It is not. With it, an attacker can:

- Map every user, role, group, and policy in the account
- Read every policy document to understand what permissions exist
- Identify over-privileged roles and their trust policies
- Find hardcoded strings, naming patterns, and account structure hints

Read-only IAM access is the first thing an attacker wants after initial credential access. Treat it as a critical permission, not a safe one.

---

*Writeup by Shrikrishna Hegde · CloudGoat — iam_enum_basics*
