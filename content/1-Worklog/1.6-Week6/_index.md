---
title: "Week 6 Worklog"
date: 2026-07-20
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Set up static storage and lesson video assets on Amazon S3.
* Deploy an Event-driven model integrating S3 Event Notifications with the Amazon SQS message queue.
* Build a worker module to handle asynchronous media transcoding tasks (HLS) from the queue.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | - Initialize S3 Buckets: `courshare-frontend-web-bucket` and `courshare-media-hls-bucket` to manage assets and video files | 20/07/2026 | 20/07/2026 | Amazon S3 Storage Classes & Setup |
| 2 | - Configure S3 Event Notifications to send messages to the Amazon SQS queue (`courshare-video-processing-queue`) whenever a new `.mp4` file is created | 21/07/2026 | 21/07/2026 | S3 Event Notifications Guide |
| 3 | - Formulate an SQS Queue Policy permitting S3 to perform `sqs:SendMessage` restricted by source ARN conditions | 22/07/2026 | 22/07/2026 | Amazon SQS Security Policies |
| 4 | - Develop a Node.js worker to poll messages from the SQS queue containing newly uploaded media metadata | 23/07/2026 | 23/07/2026 | Node.js AWS SDK for SQS |
| 5 | - Perform integration tests by uploading video files and monitoring the event dispatch workflow to SQS | 24/07/2026 | 24/07/2026 | Event-driven integration testing |

### Week 6 Achievements:

* Amazon S3 storage buckets were created and secured with strict access controls.
* Successfully established an Event-driven automated notifications workflow between S3 and SQS for async video processing.
* Configured the SQS Queue Policy securely to block unauthorized message senders.
* A background Node.js worker successfully polls video upload events from the queue, preparing it for subsequent video transcoding operations.

![S3 Buckets](/images/1-WorkLog/Bucket.png)
*List of Amazon S3 Buckets created to store static web files and media assets.*

![CloudFront Distribution](/images/1-WorkLog/Distribution.png)
*Configuration details of CloudFront Distributions for global low-latency distribution.*

