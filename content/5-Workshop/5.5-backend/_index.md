---
title: "Step 2 — Deploying AWS Backend"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Create Artifacts and Backend

The commands below use a macOS/Linux shell. On Windows, use WSL/Git Bash or adapt the variable syntax for PowerShell. Replace the email before running. Do not put AWS keys or passwords in the source. Stack names are intended for a dedicated lab account; verify they do not collide with other projects.

```bash
export AWS_PROFILE=aurel-lab
export AWS_REGION=ap-southeast-1
export SENDER_EMAIL='YOUR_OWN_VERIFIED_EMAIL'
pnpm build:lambda
aws cloudformation deploy --stack-name aurel-artifacts \
  --template-file infra/artifacts.yaml
export ARTIFACT_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name aurel-artifacts \
  --query 'Stacks[0].Outputs[?OutputKey==`BucketName`].OutputValue | [0]' \
  --output text)
aws cloudformation package --template-file infra/template.yaml \
  --s3-bucket "$ARTIFACT_BUCKET" --s3-prefix lambda \
  --output-template-file packaged-backend.yaml
aws cloudformation deploy --stack-name aurel-backend \
  --template-file packaged-backend.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND \
  --parameter-overrides FrontendOrigin=https://pending.invalid \
    SenderEmail="$SENDER_EMAIL"
aws cloudformation describe-stacks --stack-name aurel-backend \
  --query 'Stacks[0].Outputs' --output table
```

After creating the SES identity, check the verification email AWS sent to your inbox. The stack may wait for verification; if it does not complete, inspect CloudFormation Events rather than continuously creating new stacks. The pending.invalid origin is a bootstrap parameter that will be replaced with the Amplify URL in step 3.

#### Seed Data and Create Admin

Retrieve TableName, ApiUrl, MediaBucket, UserPoolId, and CognitoClientId from Outputs. Use your own lab account values, do not copy IDs from the demo.

```bash
export TABLE_NAME=YOUR_TABLE_OUTPUT
node dist/seed-aws.mjs
```

Seed only adds dishes that do not yet exist, preserves edited items, and can be rerun safely. A new table will contain 100 dishes, with no reservation data from the demo.

Use the script in bootstrap-admin.py: the script prompts for a password using getpass, creates the user without sending an invite email, adds the user to the admin group, and sets a permanent password to match the current BFF flow. Self-registration is disabled. The temporary password JSON file has restricted permissions and is deleted after the command executes; on Windows, run this in a personal directory with appropriate ACLs.

```bash
python3 bootstrap-admin.py --pool YOUR_USER_POOL_ID --email YOUR_ADMIN_EMAIL
```

#### Verification Immediately After Deploy

```bash
export API_URL=YOUR_API_URL
curl -i "$API_URL/menu"
curl -i "$API_URL/admin/reservations"
curl -i -X POST "$API_URL/reservations" \
  -H 'Content-Type: application/json' --data '{}'
```

Menu must return 200, admin routes without a token return 401, and an empty request body returns 400. Do not test browser forms until CORS is updated. CORS does not replace authorization and does not block non-browser clients.

Keep the stack/backend intact when setting up the frontend; do not create duplicate tables or SES identities manually. Update the origin parameter by redeploying the same packaged file without overwriting data.
