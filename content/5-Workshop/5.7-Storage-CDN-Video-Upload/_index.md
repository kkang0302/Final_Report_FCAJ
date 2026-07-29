---
title : "Storage, CDN & Video Upload"
date : 2024-01-01 
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

# Integrating Amazon S3, CloudFront CDN & Secure Video Upload Workflows

The CourShare system stores two primary classes of content: static assets of the client application (Frontend SPA React) and large lecture video files. We use **Amazon S3** for storage and **Amazon CloudFront** as a content delivery network (CDN) to optimize page loading speeds and streaming delivery, combined with an **S3 Presigned URL** flow to secure the client video upload process.

## Step 1: Initialize and Configure S3 Buckets
We create 2 buckets (configured with Block all public access):
1. **Frontend Web Bucket (`courshare-frontend-web`):**
   - Stores the static React SPA build.
2. **Media Bucket (`courshare-media-bucket`):**
   - Stores raw uploaded source videos and transcoded HLS stream segments.
   - **CORS (Cross-Origin Resource Sharing) Configuration:** CORS is required on the Media bucket to allow client web browsers to execute HTTP PUT requests directly to S3:
     ```json
     [
       {
         "AllowedHeaders": ["*"],
         "AllowedMethods": ["PUT", "GET", "HEAD"],
         "AllowedOrigins": ["*"],
         "ExposeHeaders": []
       }
     ]
     ```

## Step 2: Global Distribution via Amazon CloudFront (OAC)
To secure the static website, we spin up a CloudFront Distribution and enforce **Origin Access Control (OAC)** to block direct public read access to the S3 bucket:

1. Navigate to the **CloudFront Console** -> **Create distribution**.
2. Origin domain: Select `courshare-frontend-web.s3.amazonaws.com`.
3. Origin access: Select **Origin access control settings (recommended)** -> Click **Create control setting** -> Select **Sign requests (recommended)**.
4. Default cache behavior: Choose **Redirect HTTP to HTTPS**.
5. Click **Create distribution**.
6. **Update S3 Bucket Policy:** Copy the policy JSON provided by the CloudFront Console and paste it into the S3 bucket policy editor for `courshare-frontend-web` to authorize CloudFront OAC read requests:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": {
       "Sid": "AllowCloudFrontServicePrincipalReadOnly",
       "Effect": "Allow",
       "Principal": {
         "Service": "cloudfront.amazonaws.com"
       },
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::courshare-frontend-web/*",
       "Condition": {
         "StringEquals": {
           "AWS:SourceArn": "arn:aws:cloudfront::<AWS_ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
         }
       }
     }
   }
   ```

## Step 3: Secure Video Uploads via S3 Presigned URLs
If we permit clients to upload large videos (often hundreds of megabytes) through backend services like the Course Service, the payload will choke server bandwidth and overtax container memory and CPU resources.

The optimal solution is leveraging **S3 Presigned URLs**:

```
+--------+     1. Request Presigned URL      +----------------+
| Client | --------------------------------> | Course Service |
|        | <-------------------------------- |                |
+--------+       2. Return Presigned URL     +----------------+
    |
    | 3. Direct HTTP PUT Upload
    v
+-----------+
| S3 Bucket |
+-----------+
```

### Implementing Presigned URL Generation (Course Service - Spring Boot):
1. Instantiate the Amazon S3 Client using the AWS SDK:
   ```java
   AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
           .withRegion(Regions.US_EAST_1)
           .build();
   ```
2. Generate the signed URL:
   ```java
   java.util.Date expiration = new java.util.Date();
   long expTimeMillis = expiration.getTime() + 1000 * 60 * 15; // URL expires in 15 minutes
   expiration.setTime(expTimeMillis);

   GeneratePresignedUrlRequest generatePresignedUrlRequest =
           new GeneratePresignedUrlRequest(bucketName, objectKey)
           .withMethod(HttpMethod.PUT)
           .withExpiration(expiration);
   
   URL url = s3Client.generatePresignedUrl(generatePresignedUrlRequest);
   return url.toString();
   ```
3. The backend returns this URL string to the client React application. The client then issues an HTTP PUT request containing the raw video payload to the presigned URL, uploading directly to the S3 Media bucket without taxing backend services.

<!-- TODO: chèn screenshot - [AWS CloudFront Distributions console showing the list of active distributions] -->

<!-- TODO: chèn screenshot - [S3 Permissions tab showing the bucket policy correctly configured for the CloudFront OAC Service Principal] -->

<!-- TODO: chèn screenshot - [Chrome developer tools network tab recording the HTTP PUT upload request directly to S3 returning 200 OK] -->
