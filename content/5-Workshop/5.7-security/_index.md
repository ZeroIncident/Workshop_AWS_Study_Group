---
title: "Security and Well-Architected Review"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Identity and Access Control

Guests use public routes for the menu, published dish photos, reservations, and contact. Administrators must have a valid unexpired token with the expected issuer/client and admin group membership. HttpOnly cookies reduce direct JavaScript token access; Secure, SameSite=Strict and Origin checks protect BFF mutations. Tokens are not stored in localStorage. Logout attempts GlobalSignOut, but issued JWTs may still be accepted by Gateway until expiration; immediate revocation is not claimed.

Cognito MFA is currently set to OFF. Do not set MFA to REQUIRED before the UI handles challenges, as this will prevent admin logins. Subsequent priorities are TOTP/challenge support, account recovery, and thorough testing before enforcing MFA.

#### Runtime IAM

The Lambda role only accesses the project table, S3 uploads/*, menu-images/*, and the designated SES identity. However, the template currently uses DynamoDBCrudPolicy, which grants broader actions than the runtime actually needs. This should be narrowed upon review to Query, GetItem, PutItem, DeleteItem, UpdateItem, and relevant data operations within TransactWriteItems, scoped strictly to the specific table ARN; do not grant account-wide DynamoDB access.

A narrower policy example for review (not yet applied to the current stack):

```json
{
  "Effect": "Allow",
  "Action": ["dynamodb:Query", "dynamodb:GetItem", "dynamodb:PutItem", "dynamodb:DeleteItem", "dynamodb:UpdateItem"],
  "Resource": "arn:aws:dynamodb:REGION:ACCOUNT:table/YOUR_TABLE"
}
```

TransactWriteItems permissions are authorized based on its constituent component operations; do not invent non-existent IAM actions simply because the API name is TransactWriteItems. Deployment roles have different needs from runtime roles and require a separately reviewed deployment policy.

#### Data and Networking

S3 enables all four Block Public Access settings, SSE-S3, enforced Object Ownership, and non-TLS denial. DynamoDB enables SSE and PITR. No public database password exists. Presigned URLs are not public objects, but anyone with a valid URL can use it; do not expose these URLs in reports or logs.

Public endpoints are necessary for guest access. Lambda does not have a public IP directly accessed by guests; invocation is handled by Gateway. No custom VPC is deployed. Only add private subnets/endpoints when justified by networking requirements, considering NAT/endpoint costs.

API logs record only metadata, omitting forms, emails, phone numbers, Cookies, or Authorization headers. Guest records currently lack a complete retention/deletion workflow; formal data policies are required before production operations.

#### Pillar Review

| Pillar | Implementation | Limitations / Improvements |
|---|---|---|
| Operational Excellence | IaC, pipelines, tests, logs, and runbooks | Standardize rollbacks and verified small changes |
| Security | Cognito/JWT/group, cookies, private S3, no hardcoded keys | MFA, non-root deployment access, narrowed IAM actions |
| Reliability | DDB transactions/PITR, managed services | Outbox/queue, restore drills, no multi-region DR |
| Performance Efficiency | WebP, thumbnails, self-hosted fonts, arm64 | Load testing needed; dashboard requires API pagination |
| Cost Optimization | On-demand billing, 30-day lifecycle, 14-day log retention | Finalize Calculator estimate and Budget alerts |
| Sustainability | Demand-based compute, smaller image bytes and font payload | Measure traffic and remove unused resources |

#### References

- [Least privilege and temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM DynamoDB actions](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazondynamodb.html)
