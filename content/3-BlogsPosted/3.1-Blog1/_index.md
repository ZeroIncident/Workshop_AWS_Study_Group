---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Managed Spot Training: Cost Savings Need Reliable Recovery

Published on 3 November 2026 (Vietnam time). [Original AWS Study Group VN post](https://www.facebook.com/groups/660548818043427/posts/2324996344931991/).

*The experiments in this article are proposed and have no measured results.*

Training cost includes model experiments, tuning and repeated runs rather than only the final successful job. This article examines SageMaker AI Managed Spot Training through both cost and completion reliability. It is a research article with a proposed experiment, not a claim of measured savings.

Managed Spot Training uses spare capacity that can be interrupted. A job may take longer to start or finish, and shortage does not automatically switch it to On-Demand. The practical question is where execution resumes after an interruption. [Managed Spot Training](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training.html)

SageMaker synchronises checkpoint files between a container path and S3 and restores them when training restarts. A custom training script must still save and reload the correct state. Model weights alone may be insufficient: optimizer, learning-rate scheduler and step state can be needed for a faithful continuation. A checkpoint file’s presence does not prove successful recovery. [Checkpoint configuration](https://docs.aws.amazon.com/sagemaker/latest/dg/model-checkpoints.html)

![SageMaker checkpoint recovery](/images/aurel/blog-spot.png)
*Image: AWS Documentation, synchronising and restoring checkpoints. [Original context](https://docs.aws.amazon.com/sagemaker/latest/dg/model-checkpoints-resume.html). This is an AWS reference illustration, not a screenshot of a completed personal experiment.*

A proposed comparison uses identical model, data and settings for On-Demand and Managed Spot Training. First, independently test restoring a known checkpoint. Then record wall-clock completion time, billable time, interruption count and final model quality. SageMaker job phases such as Starting, Training and Interrupted help explain timing. [Job lifecycle](https://docs.aws.amazon.com/sagemaker/latest/dg/model-managed-spot-training-status.html)

Checkpoint frequency is another variable. Long intervals risk repeating more work; frequent saves introduce write and synchronisation overhead. Compare several intervals on the same workload and record total runtime alongside work repeated. Do not report a percentage saving without the relevant job data and a comparable baseline.

Place inputs, checkpoints and model artifacts in clearly identified locations. Limit the training role to required S3 paths and review retained artifacts after the experiment. Storage and ancillary resources are part of the cost model rather than disappearing when compute finishes.

Spot is worth considering for recoverable experiments that tolerate waiting. Deadline-sensitive work must weigh the discount against delayed completion. This research supports that decision framework; it does not establish the savings of an unexecuted experiment.

#AWS #AWSStudyGroup #FCAJ #FirstCloudAIJourney #AmazonSageMaker #ManagedSpotTraining #MachineLearning #CostOptimization
