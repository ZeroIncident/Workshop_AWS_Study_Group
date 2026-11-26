---
title: "Step 1 — Local Setup and Checks"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Prepare the Source Code

Use the current GitHub source repository to practice the dish photo upload feature. The archive ZIP supports the original lab with `package.json` located at the root. From the source directory:

```bash
unzip aurel-restaurant-source.zip -d aurel-lab
cd aurel-lab
corepack enable
corepack prepare pnpm@11.25.0 --activate
pnpm install --frozen-lockfile
pnpm setup
pnpm dev
```

Open http://127.0.0.1:3000. The setup script generates a local password and random session secret in .env.local if the file does not already exist. Default local admin is admin@aurel.local; inspect the password locally on your machine and never commit it to the report. Local development uses a single-process FileStore, not a multi-server database.

#### Pre-deployment Checks

Stop the dev server prior to running smoke tests if port release is needed. Smoke tests run on port 3100 with isolated temporary data.

```bash
pnpm check
pnpm build
pnpm test:smoke
```

check runs TypeScript, Vitest, and Lambda/seed bundling. Tests validate business logic, authorization, capacity limits, unaccented search, and pagination; DynamoDB transaction unit tests use mocks and do not simulate concurrent live service traffic.

Manual verification: search for "pho" and "phở", switch categories, paginate, submit invalid/valid forms, log in, and update reservation status. These manual steps serve as implementation guidelines; only results documented in section 5.8 constitute verified evidence.

#### Fonts and Images

Noto Sans and Noto Serif are embedded via next/font/local with Vietnamese glyphs; fallback fonts include Segoe UI, Arial, and Georgia. WebP images reside in public/images, with attribution and licensing referenced in the catalog. The interface does not require runtime calls to Google Fonts.
