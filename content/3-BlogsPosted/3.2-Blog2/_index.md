---
title: "Blog 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Lambda and SQS: Why Successful Work Can Run Again

Published on 5 November 2026 (Vietnam time). [Original AWS Study Group VN post](https://www.facebook.com/groups/660548818043427/posts/2326372234794402/).

*The experiments in this article are proposed and have no measured results.*

An order-processing or sensor-data application can acknowledge incoming work quickly and process it later through SQS and Lambda. The difficult part is partial failure: some messages may complete before another fails. This research examines retry behaviour and correctness without claiming a deployed SQS pipeline in AUREL.

An SQS event source mapping polls messages and invokes Lambda in batches. Successfully processed messages are deleted, but delivery is at least once and duplicates remain possible. Applications must be designed for repeated processing. [Lambda with SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)

Consider a batch of ten messages where the sixth fails. Under default failure behaviour, the batch can reappear after visibility timeout, including messages whose business actions succeeded. An unconditional counter increment or transaction creation could therefore run twice.

Partial batch responses let the function identify failed messages. Both ReportBatchItemFailures on the event source mapping and the correct function response are necessary. Changing only the function code is insufficient. [Partial batch error handling](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)

![SQS event source mapping configuration](/images/aurel/blog-sqs.png)
*Image: AWS Documentation, event source mapping in the SQS tutorial. [Original tutorial](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs-example.html). The image is a reference, not evidence of a personally executed test.*

Reducing unnecessary retries does not remove the need for idempotency. Use a business request identifier and design for simultaneous workers and failures between the business effect and completion marker. A separate check-then-write sequence can race. The hardest boundary is an external effect that cannot participate in the database transaction; acknowledge and test that uncertainty rather than promising exactly-once behaviour.

A proposed experiment mixes valid messages, deliberately invalid messages and duplicate business IDs. Compare invocation/message counts with the final number of business outcomes. Verify that repeatedly failing messages reach the configured dead-letter queue. Function timeout, visibility timeout and redrive policy must be assessed together. [SQS configuration for Lambda](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html)

Persistent poison messages can increase compute and logging cost. Scope the execution role to the queue and result store and avoid logging sensitive payloads. Queueing separates reception from processing; application-level state and failure handling still determine whether results are correct.

#AWS #AWSStudyGroup #FCAJ #FirstCloudAIJourney #AWSLambda #AmazonSQS #Serverless #DistributedSystems
