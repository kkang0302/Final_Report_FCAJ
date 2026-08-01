---
title: "Blog 3"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Large-scale Asynchronous Video Upload and Processing Solution with S3 Presigned URLs, SQS, and ECS Fargate

When building CourShare, one of the core yet challenging features was managing and streaming lecture videos. Educational videos are typically large. If users upload them directly to microservices (such as the Course Service), the system would quickly experience bandwidth congestion, buffer overflow, or disrupt other functionalities.

To solve this problem, I designed and implemented an Asynchronous Video Processing pipeline that leverages the power of AWS services. In this post, I will share the details of this architecture.

![Video processing pipeline architecture](/images/3-BlogPosted/7c712349-a07d-437e-b63f-976cb4acbd0d.jpg)

### Challenges of Video Processing in a Microservices Model

For typical web applications, the file upload flow is usually:
1. Client sends the file (POST multipart/form-data) to the Backend.
2. Backend temporarily stores the file in disk or memory.
3. Backend uploads that file to S3.

This flow works well for avatar images or small documents. But for lecture videos that are tens of minutes long (hundreds of MBs to gigabytes), the backend will get choked while receiving and forwarding the file. Furthermore, playing raw videos directly (such as .mp4 files) consumes excessive bandwidth and yields a poor user experience for users on slow networks since they must download the entire file to watch it.

---

### Solution Architecture: Separating Upload and Processing

The architecture of CourShare's video processing flow consists of three main phases:

#### 1. Direct Upload via S3 Presigned URLs
To protect the backend, CourShare allows clients to upload files directly to Amazon S3 without going through the API Gateway or the Course Service.
* When an instructor uploads a video, the client requests an *S3 Presigned URL* from the Course Service.
* The Course Service generates a short-lived temporary URL (e.g., 15 minutes) with file size and format constraints, and returns it to the client.
* The client performs a PUT request directly from the browser to S3 via that Presigned URL. S3's infrastructure automatically handles large uploads optimally.

#### 2. Triggering Asynchronous Processing via Amazon S3 Event Notifications & SQS
Once the raw video is fully uploaded into the S3 Media Bucket (under the `/uploads` directory):
* Amazon S3 triggers an `ObjectCreated:Put` event notification.
* This event is sent directly to an *Amazon SQS (Simple Queue Service)* queue.
* Utilizing SQS acts as a load buffer, preventing system overload when dozens of instructors upload videos simultaneously.

#### 3. Automated Transcoding with ECS Fargate Workers
A pool of Worker tasks running on *Amazon ECS Fargate* continuously listens (long-polling) to the SQS queue:
* Upon receiving a message containing details about a new video file, the Worker downloads the raw video from S3.
* The Worker uses the FFmpeg library to transcode the video into the *HLS (HTTP Live Streaming)* format. The video is sliced into small segment files (.ts, a few seconds each) along with an index playlist file (.m3u8).
* The transcoded output is uploaded back to the S3 Media Bucket under the `/processed` directory.
* Once completed, the Worker updates the lecture status in the Course Service database to `Ready` and deletes the message from SQS.

---

### Optimizing User Experience with Amazon CloudFront & OAC

To ensure students can stream lecture videos smoothly with minimal latency and no interruptions:
* All videos in the `/processed` folder are distributed via *Amazon CloudFront CDN*. CloudFront caches video segments at Edge Locations nearest to the users.
* To protect the copyright of lecture videos (preventing users from sharing direct S3 links), the Media Bucket is configured to be completely private. CloudFront accesses S3 via *Origin Access Control (OAC)*—AWS's latest security mechanism, ensuring that nobody can access the videos without going through the designated CDN domain.
