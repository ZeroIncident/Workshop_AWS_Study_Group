---
title: "Troubleshooting and Technical Lessons"
date: 2024-01-01
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

#### Project Troubleshooting Scenarios

| Situation | Diagnosis | Resolution and Evidence |
| :--- | :--- | :--- |
| Nested source ZIP folder | Build could not locate package.json at root | Created package-source script to produce a clean archive at the root |
| Reserved concurrency 10 conflicted with quota of 10 | Did not leave unreserved capacity | Removed reserved concurrency, retained API gateway throttling, and documented account limits |
| Origin configuration changes caused CORS/BFF errors | Public API and BFF have different request pathways | Aligned API/S3 CORS with the same origin, updated APP_ORIGIN, and rebuilt the public environment |
| SES sandbox restrictions | Sending capability does not cover arbitrary recipient addresses | Verified sender identity, tested with mailbox simulator, persisted requests prior to notification, and clearly documented sandbox mode |
| Documentation commits triggered app builds | Branch pipeline did not reliably recognize skip markers | Stopped docs-only build jobs; separated report repository from the application repository |
| Legacy seed and 100-item catalogue | Custom items added by users must not be erased | Implemented migration backup and recognized default sample dishes while preserving other data |
| Desiring real photos from different locations | Misrepresentation risk regarding branding/venue | Labeled images as documentary/inspiration assets with source attribution and licenses per photo |

#### Transferable Lessons

Tracing requests along data pathways helps identify whether errors originate in the browser, BFF, API Gateway, Lambda, or database. A successful build does not prove durable form submission; an HTTP 200 does not prove responsiveness or accessibility. An SES delivery failure does not mean the reservation transaction failed.

Least privilege requires examining individual actions and resources, not just policy names. Managed services do not automatically satisfy all security requirements. Retain protects data but introduces operational cleanup and billing oversight responsibilities.

#### Prioritized Future Roadmap

- Migrate deployment operations away from root, narrow IAM policies, implement and test MFA.
- Complete external notification delivery and set an agreed AWS Budget threshold.
- Introduce Outbox/SQS/DLQ, handle duplicate messages, and reconcile pending outbox records.
- Implement API pagination, rule-based pending reservation expiry, and modeling for tables/dining durations.
- Run controlled load testing, restore drills, and observe empirical p95 latency and cost measurements.
- Perform testing across Windows/mobile/accessibility, and add walkthrough video demonstrations and personally revised architecture diagrams.

This represents a technical roadmap rather than a list of currently implemented features. Directly attaching AWS WAF to an HTTP API is not supported; a compatible protective architecture must be designed first.
