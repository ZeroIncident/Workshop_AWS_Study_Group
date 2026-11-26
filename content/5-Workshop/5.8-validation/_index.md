---
title: "Testing and Validation"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Evidenced Results

The review verified that all 84 English and Vietnamese report pages returned HTTP 200. The AUREL homepage returned 200, the public API returned 100 dishes with unique IDs, and the admin API returned 401 when token was missing. HTTP results confirmed.

#### Test Results Table

| Group | Recorded Results | Scope |
|---|---|---|
| Local | 30 tests; typecheck/build; 28 HTTP smoke assertions | Prior to venue gallery release; DynamoDB transactions mocked |
| AWS Backend | 14 successful checks | Cognito/access token, ID token rejection, persistence/status/idempotency, live S3 PUT/GET, contact |
| Menu Release | Public/admin APIs return 100 dishes, 100 byte-matched images, 7 HTTP 200 pages | HTTP/assets/auth validation, without browser automation |
| Venue Gallery Release | 12 cards, 8 venue images, 20 byte-matched assets, menu remains at 100 | HTTP and server-rendered HTML |
| Monitoring | See updated evidence in section 5.9 | Simulated alarm transitions do not imply real outages |

#### Evidence Links


Browser checks were previously conducted on the initial deployment: BFF login, dashboard, logout, and reservations. Those screenshots predate the 100-dish menu update; they should not be used to claim that all interactions of the new gallery have been verified.

#### Reproducible Negative Requests

```bash
export API_URL=YOUR_API_URL
curl -i "$API_URL/admin/reservations"
curl -i -X POST "$API_URL/reservations" \
  -H 'Content-Type: application/json' --data '{}'
curl -i -X POST "$API_URL/reservations" \
  -H 'Content-Type: application/json' --data '{bad-json'
curl -i -X OPTIONS "$API_URL/menu" \
  -H 'Origin: https://untrusted.example' \
  -H 'Access-Control-Request-Method: GET'
```

Expected responses: 401, 400, and 400 respectively; untrusted origins do not receive an Access-Control-Allow-Origin header granting access. HTTP 200 for OPTIONS does not mean the origin is allowed. Do not run heavy load tests against the public demo.

#### State-Changing Tests — Lab Account Only

Use the form with a verified email or SES mailbox simulator, with a date 1–90 days in the future and at least 30 minutes in advance. Record the returned UUID, log in to admin, confirm, reload, and then cancel. Reconcile DynamoDB records and the slot counter. Resubmitting with the same UUID returns the same request; changing data requires a new UUID.

To test real concurrency, isolate a dedicated slot and send concurrent requests with total guests exceeding 24. After testing, verify total seats do not exceed 24 and no orphaned partial records exist. This concurrency load test has not been executed on live DynamoDB in the current evidence set.

Do not wipe the table to clean up tests. Clean up only the specific UUIDs created during testing; cancel first to release capacity, then delete test records if needed. Retain status logs without storing tokens or personal data in the report.

#### Remaining UI Checks

Direct testing is still needed on Windows/Chrome/Edge, iOS/Android devices, lightbox keyboard navigation, screen readers, color contrast, and forms on small screens. These cannot be assumed to pass solely from HTTP 200 responses or successful builds.
