---
title: "Overview and Completion Criteria"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Product and Users

AUREL is an original demonstration brand with a custom dark/gold interface. Guests can browse 100 Vietnamese dishes and supplementary items, view credited venue photographs, submit reservation requests, and send messages. Staff accounts are provisioned via Cognito and assigned to the admin group to manage data.

Reservation requests are set to pending upon receipt. Staff then decide whether to confirm or cancel. This is an important business distinction: receiving a successful form submission notification does not mean the restaurant has confirmed the reservation.

#### Technology

Next.js 15.5.28, React 19, TypeScript, Tailwind CSS, Zod, AWS SDK v3, Node.js 22, pnpm 11.25.0. Lambda runs on arm64 with 256 MB memory and a 20-second timeout in Singapore. Data is stored on DynamoDB on-demand, source code is privately hosted on CodeCommit, and SSR interface runs on Amplify WEB_COMPUTE.

#### Workshop Deliverables

- A 7-page website with functional forms integrated with the backend.
- A 100-dish menu with self-hosted WebP images; clearly documented sources and licenses.
- Explicit access control boundaries between public and admin APIs.
- Capacity protection via transactions within a single slot; UUID deduplication preventing duplicate requests.
- Infrastructure templates, build/deploy instructions, testing, observability, and cleanup procedures.

#### Limitations to Understand Before Starting

SES remains in the sandbox; admin MFA lacks a handling UI; no outbox or queue has been deployed. The capacity model only applies to the same time slot and does not account for dining duration. Venue photographs are curated from multiple locations rather than an actual physical AUREL restaurant.

Performance targets and expanded architectures are labelled as proposals rather than implemented features. The workshop does not include commercial SLA commitments, formal security audits, or cross-device testing.
