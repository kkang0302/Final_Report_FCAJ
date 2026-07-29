---
title : "Asynchronous Processing"
date : 2024-01-01 
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

# Decoupling the System via Asynchronous Processing with Amazon SQS & Workers

In the CourShare platform, tasks such as video transcoding and delivering transactional email notifications are resource-intensive and time-consuming. We decouple these workloads from the synchronous client-facing API pipeline using **Amazon SQS** message queues alongside background daemon tasks: **VideoWorker** and **NotificationWorker**.

## Step 1: Initialize Amazon SQS Queues
1. Navigate to the **SQS Console** -> **Create queue**.
2. Create 2 **Standard** queues:
   - `courshare-video-transcode-queue` (for video transcoding tasks).
   - `courshare-notification-queue` (for transaction and enrollment notifications).
3. Set the **Default Visibility Timeout** to 10 minutes (600 seconds) for the video transcode queue (because transcoding large files requires substantial processing time; this prevents messages from showing back up in the queue while the worker is actively processing them).

## Step 2: Build and Deploy the VideoWorker (HLS Transcoding)
When an instructor successfully uploads a raw source video (.mp4) to the S3 Media bucket, the Course Service publishes a JSON message containing the `bucketName` and `objectKey` to the `courshare-video-transcode-queue`.

The **VideoWorker** runs continuously as an ECS Service in the Private Subnet, performing the following:
1. Polls (using Long Polling) messages from the SQS queue via the AWS SDK:
   ```javascript
   const { SQSClient, ReceiveMessageCommand, DeleteMessageCommand } = require("@aws-sdk/client-sqs");
   // Long polling is set by specifying WaitTimeSeconds = 20
   ```
2. Upon receiving a message, VideoWorker downloads the raw `.mp4` file from the S3 Media bucket to the task container's local ephemeral storage.
3. Spawns an **ffmpeg** subprocess to transcode the video into HTTP Live Streaming (HLS) format:
   ```bash
   ffmpeg -i input.mp4 -profile:v baseline -level 3.0 -s 640x360 -start_number 0 -hls_time 10 -hls_list_size 0 -f hls index.m3u8
   ```
   *This process generates a master index file `index.m3u8` and multiple small segmented media chunks `.ts` (e.g., `index0.ts`, `index1.ts`...).*
4. Uploads all `.m3u8` and `.ts` files to the appropriate course subfolder back in the S3 Media Bucket.
5. Deletes the message from the SQS queue to acknowledge successful execution.

## Step 3: Deploy the NotificationWorker (Email Alerts)
Similarly, when payment transactions or course enrollments complete, the respective microservices dispatch a JSON message containing contact details and email templates to the `courshare-notification-queue`.

The **NotificationWorker** polls this queue, parses the payloads, and calls the mailer integration to deliver emails directly to student and instructor mailboxes, reducing API response times for clients.

<!-- TODO: chèn screenshot - [AWS SQS Console displaying the list of SQS Queues, showing their URLs and Visibility Timeout configurations] -->

<!-- TODO: chèn screenshot - [Amazon S3 Media Bucket folder structure displaying the index.m3u8 file and ts segment segments uploaded by the VideoWorker] -->
