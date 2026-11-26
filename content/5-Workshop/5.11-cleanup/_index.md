---
title: "Resource Cleanup"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

#### Principles and Inventory

These instructions target the reader's lab account. Do not execute cleanup on an active demo without having decided to retire the service. Deleting a CloudFormation stack does not mean all resources are removed: the DynamoDB table, media/artifact buckets, and CodeCommit repository have a `Retain` deletion policy.

Prior to deletion, export the physical resource inventory and identify ownership. Back up necessary data to an authorized private location and confirm backups can be read. A real-world restore drill has not yet been conducted to empirically validate RTO/RPO.

```bash
aws cloudformation list-stack-resources --stack-name aurel-backend
aws cloudformation list-stack-resources --stack-name aurel-frontend
aws cloudformation list-stack-resources --stack-name aurel-artifacts
aws cloudformation list-stack-resources --stack-name aurel-observability
```

#### Lab Cleanup Sequence

- Stop new pipelines/builds; record AppId and resource IDs prior to deletion.
- Archive report source/builds, then delete the aurel-report-branch stack and the source report branch if no longer needed. Do not delete the shared Amplify app or the restaurant's main branch. Any empty buckets remaining from CloudFront trials must be inventoried separately by stack.
- Delete the frontend stack; wait for DELETE_COMPLETE. CodeCommit has a Retain policy and requires separate cleanup.
- Delete the backend stack; wait for completion. Stack-owned Cognito users, SES identities, and log groups will be removed; the DynamoDB table and media bucket will remain.
- Delete the observability stack to clean up alarms, dashboards, and API access logs. Only delete if logs are no longer required for analysis.
- For retained S3 buckets, check versioning, all versions/delete markers, and multipart uploads. After backing up, empty the bucket, then delete it. The command rm --recursive does not delete all versions in a versioned bucket.
- Delete only the specific retained DynamoDB tables and repositories identified in your inventory; review any manual backups/exports and their associated fees.
- Clean up the artifact bucket, then delete the artifacts stack; handle the retained bucket if detached from stack management.
- Check out-of-stack Amplify logs, SNS subscriptions/Budgets added later, and resources in other regions. Do not delete unrelated project resources.

Sample stack deletion commands, to be executed only after deciding to conclude the lab and completing backups:

```bash
aws cloudformation delete-stack --stack-name aurel-frontend
aws cloudformation wait stack-delete-complete --stack-name aurel-frontend
aws cloudformation delete-stack --stack-name aurel-backend
aws cloudformation wait stack-delete-complete --stack-name aurel-backend
aws cloudformation delete-stack --stack-name aurel-observability
aws cloudformation wait stack-delete-complete --stack-name aurel-observability
```

Retained resource deletion is intentionally not automated via wildcards due to the risk of deleting out-of-scope data. Verify actual resource states and check AWS Billing after data updates; closing the browser does not stop charges. This report does not claim that the infrastructure serving the live demo has been deleted.

**References:**
- [Retain in CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html)
- [Emptying an S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html)
