---
title: "Prerequisites"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Accounts and Permissions

Use an AWS account with permissions to create CloudFormation stacks and the services declared in the templates. IAM Identity Center and temporary roles are preferred; do not create root access keys. The initial project deployment used root via a temporary CLI login with account owner approval; this is an improvement area, not an operational best practice.

An owned email address is required for SES verification and admin emails. Lab participants must use their own account and email addresses; do not use the resources or administrative account of the public demo.

#### Tools

| Tool | Version / Purpose |
|---|---|
| Node.js | 22.13 or later; Node 22 recommended |
| pnpm | 11.25.0, matching packageManager |
| AWS CLI | v2; configured for SSO or temporary session login |
| Python | 3.9+ for source packaging |
| Git | Connecting to CodeCommit when updates are needed |
| SAM CLI | Optional; the primary guide uses CloudFormation package/deploy |
| Hugo | 0.134.3 to match the current report template |

Docker is not required for the esbuild packaging process used here.

#### Verification and Configuration Commands

```bash
node --version
pnpm --version
aws --version
python3 --version
git --version
aws configure sso --profile aurel-lab
aws sso login --profile aurel-lab
export AWS_PROFILE=aurel-lab
export AWS_REGION=ap-southeast-1
aws sts get-caller-identity
```

Keep identity outputs local on your machine; when including them in reports, show only the redacted role and region. Never capture access keys, session tokens, or passwords.

#### Quota and Budget Checks

The deployment account had a Lambda concurrency quota of 10. Do not set reserved concurrency equal to the entire quota because AWS requires an unreserved portion. The default API throttle of 10 requests/second with a burst of 20 does not inherently prevent Lambda throttling. Before the lab, inspect your account's actual quotas and set an appropriate AWS Budget; there is no evidence that an AWS Budget was created in this project.

#### References

- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Configuring CLI SSO Login](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
