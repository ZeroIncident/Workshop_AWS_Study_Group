---
title: "COST — Estimation and Cost Optimization"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

#### Workload Scenarios and Methodology

Cost estimate prepared on 03/11/2026 for Singapore in USD/month, excluding VAT, credits, Free Tier, and discounts. This is an independent planning model based on AWS published pricing, not an invoice or an AWS Pricing Calculator export.

Assumptions: 10,000 visits/month, 100,000 API requests, 100,000 SSR requests; average SSR duration 200 ms at 0.5 GB, average Lambda duration 200 ms at 0.25 GB; 20 GB frontend bandwidth; 5 administrators; 1,000 emails; 0.1 GB DynamoDB table; 2 GB S3 storage; 0.25 GB log ingestion; 4 CloudWatch alarms and 1 dashboard. SSR latency and memory figures are planning assumptions, not benchmarked measurements.

The 3.2 million RRU allowance provisions for strong-consistent GET /menu requests returning 100 items. Do not assume one HTTP request consumes only one RRU. The 20,000 WRU estimate accounts for rate limiters and transaction overhead; replace with observed ConsumedCapacity when available.

#### Detailed Cost Breakdown

| Service | Quantity | Unit rate (USD) | Monthly (USD) |
|---|---|---|---|
| Amplify build | 30 minute | 0.01 | 0.3000 |
| Amplify storage | 1 GB-month | 0.023 | 0.0230 |
| Amplify transfer | 20 GB | 0.15 | 3.0000 |
| Amplify SSR requests | 1e+05 request | 0.0000003 | 0.0300 |
| Amplify SSR duration | 2.778 GB-hour | 0.20 | 0.5556 |
| Lambda arm64 compute | 5,000 GB-second | 0.0000133334 | 0.0667 |
| Lambda requests | 1e+05 request | 0.0000002 | 0.0200 |
| HTTP API requests | 1e+05 request | 0.00000125 | 0.1250 |
| DynamoDB strong reads | 3.2e+06 RRU | 0.0000001425 | 0.4560 |
| DynamoDB writes | 2e+04 WRU | 0.00000071 | 0.0142 |
| DynamoDB storage | 0.1 GB-month | 0.285 | 0.0285 |
| DynamoDB PITR | 0.1 GB-month | 0.228 | 0.0228 |
| S3 Standard storage | 2 GB-month | 0.025 | 0.0500 |
| S3 PUT | 1,000 request | 0.000005 | 0.0050 |
| S3 GET | 1e+04 request | 0.0000004 | 0.0040 |
| Cognito Essentials | 5 MAU | 0.015 | 0.0750 |
| SES Essentials outbound | 1,000 email | 0.00016 | 0.1600 |
| CloudWatch logs ingestion | 0.25 GB | 0.70 | 0.1750 |
| CloudWatch log storage | 0.1 GB-month | 0.03 | 0.0030 |
| CloudWatch standard alarms | 4 alarm-month | 0.10 | 0.4000 |
| CloudWatch dashboard budget | 1 dashboard-month | 3 | 3.0000 |
| Report Amplify transfer | 1 GB | 0.15 | 0.1500 |
| Report Amplify storage | 0.1 GB-month | 0.023 | 0.0023 |
| API/S3 internet transfer allowance | 1 budget allowance | 2 | 2.0000 |
| CodeCommit allowance | 1 budget allowance | 1 | 1.0000 |
| Report build and SSR allowance | 1 budget allowance | 0.5 | 0.5000 |
| **Total** | | | **12.17** |
| **Budget +25%** | | | **15.21** |

Attached resources: [CSV with formulas and input sources](/downloads/cost-model.csv) · [JSON Scenario](/downloads/cost-scenario.json)

Lambda: 100,000 × 0.2 seconds × 0.25 GB = 5,000 GB-seconds. SSR: 100,000 × 0.2 × 0.5 / 3,600 = 2.7778 GB-hours. Allowance lines represent contingency provisions rather than quoted AWS unit rates. Regional rates were matched against the AWS Price List; Amplify rates follow the official pricing documentation. A 3 USD dashboard buffer is retained regardless of potential free-tier allocations. SES uses the recorded Essentials tier at 0.16 USD / 1,000 emails. Cognito uses marginal Essentials pricing and should be confirmed against the actual deployed tier. The CodeCommit allowance should be adjusted based on account users, storage, and quotas.

Unquantified items: email attachments/payload data, DynamoDB restore/export, additional builds and artifacts, large log queries, paid support plans, custom domain/DNS, WAF, NAT, SQS, and proposed architectural evolutions. API/S3 transfer includes a 2 USD buffer that may not suffice under alternative workloads. Multi-region replication and SMS MFA are excluded from this estimate.

#### Actual AWS Pricing Calculator Estimate

Workload estimate `AUREL-FCAJ-20261103` was created using the official in-console AWS Pricing Calculator API. ID: `5c91cb04-0f41-436d-ae5f-72733aa95e31`. AWS returned a status of VALID; all 20 usage lines are VALID with no input errors. Rate type is `BEFORE_DISCOUNTS` with a price timestamp of 03/11/2026. This models costs without provisioning resources and is not an invoice.


#### AWS Pricing Calculator Results Table

| Service | AWS Monthly Cost (USD) |
|---|---|
| AWSAmplify | 4.419412 |
| AWSLambda | 0.086667 |
| AmazonApiGateway | 0.125000 |
| AmazonCloudWatch | 0.578000 |
| AmazonCognito | 0.075000 |
| AmazonDynamoDB | 0.493000 |
| AmazonS3 | 0.059000 |
| AmazonSES | 0.160000 |
| **Total (20 lines)** | **5.996079** |

The approximately 6.00 USD/month total covers only the 20 modeled usage lines across 8 services, not the complete operational scope. Amplify includes report hosting: 60 build minutes, 1.1 GB storage, 21 GB delivery, 110,000 SSR requests, and 11,000 GB-seconds. Workload usage types use APS1 for Singapore; the Pricing Calculator API was invoked via the us-east-1 endpoint.

The variance from the independent budget of 12.17 USD is explained by: 3 USD dashboard buffer, 2 USD API/S3 transfer buffer, and 1 USD CodeCommit allowance outside the AWS Calculator model, alongside report buffers, storage tiering, and rounding. AWS priced 0.1 GB DynamoDB storage at 0 USD under current free tiers; the independent estimate conservatively applies a paid marginal rate. BEFORE_DISCOUNTS does not eliminate 0 USD service tiers.
