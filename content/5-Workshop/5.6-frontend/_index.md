---
title: "Step 3 — Frontend and CI/CD"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Initialize Source and Amplify

Set API_URL, COGNITO_CLIENT_ID, and USER_POOL_ID from the backend outputs before running. The template provisions a dedicated CodeCommit repository, a read-only role for this repository, and an Amplify WEB_COMPUTE app. The source ZIP must contain package.json at its root.

```bash
python3 scripts/package-source.py --output ../aurel-codecommit-source.zip
aws s3 cp ../aurel-codecommit-source.zip \
  "s3://$ARTIFACT_BUCKET/source/aurel-lab.zip"
aws cloudformation deploy --stack-name aurel-frontend \
  --template-file infra/amplify-codecommit.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides SourceBucket="$ARTIFACT_BUCKET" \
    SourceKey=source/aurel-lab.zip ApiUrl="$API_URL" \
    CognitoClientId="$COGNITO_CLIENT_ID" \
    CognitoUserPoolId="$USER_POOL_ID"
aws cloudformation describe-stacks --stack-name aurel-frontend \
  --query 'Stacks[0].Outputs' --output table
```

Retrieve AppId and FrontendOrigin from Outputs, then set APP_ID/FRONTEND_ORIGIN. Update backend CORS and S3 CORS using the exact same origin without a trailing slash /.

```bash
aws cloudformation deploy --stack-name aurel-backend \
  --template-file packaged-backend.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND \
  --parameter-overrides FrontendOrigin="$FRONTEND_ORIGIN" \
    SenderEmail="$SENDER_EMAIL"
aws amplify start-job --app-id "$APP_ID" --branch-name main --job-type RELEASE
aws amplify list-jobs --app-id "$APP_ID" --branch-name main --max-results 5
```

Only consider deployment successful when the job status is SUCCEED. Verify BUILD/DEPLOY/VERIFY stages, open the URL, and cross-check API responses. In the actual deployment, job 5 succeeded with commit ffe7deaa88b7090ad0f10ca60ed1f86f3cc16d8b.

#### Environment Configuration

| Variable | Source / Rule |
|---|---|
| DATA_MODE | aws |
| API_BASE_URL, NEXT_PUBLIC_API_BASE_URL | ApiUrl, no trailing slash |
| COGNITO_CLIENT_ID, COGNITO_USER_POOL_ID | Backend outputs |
| APP_AWS_REGION | ap-southeast-1 |
| APP_ORIGIN | Exact frontend origin |

scripts/amplify-env.mjs only writes allowed configurations to .env.production. Never place admin passwords, SESSION_SECRET, or AWS keys into Amplify frontend variables. BFF uses the Cognito public client and API, requiring no direct database access roles.

#### Updating Source Code

Code.S3 only loads the source during initial repository creation. Replacing the ZIP in S3 and updating the stack will not update the Git history. Subsequent updates must use Git/CodeCommit with temporary IAM sessions, pushing to the main branch; Amplify will build automatically. Do not store tokens in remote URLs and avoid using Git helpers that cache credentials unintentionally.

If the account cannot connect to CodeCommit, use the GitHub alternative outlined in the application README and amplify.yaml template; never make the repository public with .env files. This alternative path has not been tested in a new account.

**References:**
- [Amplify Next SSR](https://docs.aws.amazon.com/amplify/latest/userguide/deploy-nextjs-app.html)
- [SSR environment](https://docs.aws.amazon.com/amplify/latest/userguide/environment-variables.html)

#### Dedicated Report Website on the Report Branch

Account quotas do not permit creating a new Amplify app; CloudFront requires account verification so an S3/CloudFront approach could not be deployed. The report is exported from Hugo to a minimal Next.js host on a separate CodeCommit report branch, attached to the Amplify report branch of the existing app. The restaurant's main branch and backend remain unchanged.

The host preserves the HTML, CSS, JavaScript, and fonts of the Hugo template; routes serve HTML from public, and static assets pass through Hosting. The configuration preserves trailing slashes and packages HTML within server trace. Large downloadable files are split when stored in Git. The download page fetches small chunks, validates SHA-256, and reconstructs the ZIP in the browser to avoid SSR response limits.

```bash
hugo --baseURL https://report.YOUR_APP_ID.amplifyapp.com/ --minify --cleanDestinationDir
python3 scripts/check-report.py
python3 scripts/export-host.py --destination ../aurel-report-host
# Push exported source to the separate report branch first.
aws cloudformation deploy --stack-name aurel-report-branch \
  --template-file report-branch.json --parameter-overrides ExistingAppId=YOUR_APP_ID
aws amplify start-job --app-id YOUR_APP_ID --branch-name report --job-type RELEASE
```

The README in the report source provides guidance on updating and cleaning up the branch separately without deleting the shared app.
