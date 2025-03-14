---
id: cloudfront
title: Amazon CloudFront
description: The Sumo Logic app for Amazon CloudFront provides analytics on visitor information, rates and statistics, content being served, and other metrics.
---

import useBaseUrl from '@docusaurus/useBaseUrl';

<img src={useBaseUrl('img/integrations/amazon-aws/cloudfront.png')} alt="Thumbnail icon" width="50"/>

Amazon CloudFront is a content delivery network (CDN) that provides an easy way for companies to distribute content to end-users with low latency and high data transfer speeds. The Sumo Logic app for Amazon CloudFront provides analytics on visitor information, rates and statistics, content being served, and other metrics. The app uses predefined searches and Dashboards that provide visibility into your environment for real time analysis of overall usage.

## Log types

The Sumo Logic app for Amazon CloudFront uses logs from an Amazon S3 bucket. These logs will be generated by the Amazon CloudFront service. For setup details, refer to Collecting Logs for Amazon CloudFront.

### Sample log messages

```json
2017-09-27 00:21:12 ORD51-M1 335 65.30.1.138 GET domain.cloudfront.net /content/FDW/HLS/HLS360p/FDW_s01e002_tv_hv_or_en_xx_HLS360p_16x9_00_v00154.ts 403 https://www.company.com/stream/food-wars/s01e002 Mozilla/5.0%2520(Macintosh;%2520Intel%2520Mac%2520OS%2520X%252010.12;%2520rv:54.0)%2520Gecko/20100101%2520Firefox/54.0 Policy=eyJTdGF0ZW1lbnQiOiBbeyJSZXNvdXJjZSI6Imh0dHBzOi8vZDM5dG5yZzRlbnBrNncuY2xvdWRmcm9udC5uZXQvY29udGVudC9GRFcvSExTL0hMUzM2MHAvRkRXX3MwMWUwMDJfdHZfaHZfb3JfZW5feHhfSExTMzYwcF8xNng5XzAwX3YwMDE1NC50cyIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTUwMzk4MTc1OX0sIklwQWRkcmVzcyI6eyJBV1M6U291cmNlSXAiOiI2NS4zMC4xLjEzOC8zMiJ9LCJEYXRlR3JlYXRlclRoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTUwMzk3NDI1OX19fV19&Signature=NCDlKRp6Nv9triqA1r-RBulrMXlvQCRxH16c3dP4RGdmwx8yQO0d75%7EdN94-wwaQ2x7NDlzNUrn7IXkUyHJN3S9kdx7RfVt-gQw9E3hMc4rYYe5NVR0wAeye%7E3gMKuFY%7EhshJqMrbE96HmzzhgQ5qS9gW797PDiwddCmtjYxqgndfF7jO2JJ9QwSpHfqcn5Ceo89Ra0mxwjo4JYu8JfiDhbOAkTU7mpy1ql%7EmYOuwc4zntjMK%7ERKOtcrV3sP9uIunpdh6Ur0-pOmPYTJt13VgUfoYmFB0CJnc8TMosN8ouqMIlSnLXfeKiIdDiP%7EGQKtYeZ54NVx6PqrmOQBSVhikg__&Key-Pair-Id=APKAIWFUV66JZQCBHYXA - Error h5BKcPRKo5oIEz0KZ06V6bRCTJttiW_WUJQmT71jjTnYGE8pA1kfQA== domain.cloudfront.net https 722 0.001 - TLSv1.2 ECDHE-RSA-AES128-GCM-SHA256 Error HTTP/2.0
```

### Sample queries

```sql title="Count of HTTP Status Codes"
_sourceCategory= aws/cf | parse "*\t*\t*\t*\t*\t*\t*\t*\t*\t*\t*\t*\t*\t*\t*" as _filedate,_ftime,edgeloc, scbytes, c_ip,method,cs_host,uri_stem,status,referer,user_agent,uri_query,cookie,edgeresult,requestid
| count as count by status
| sort by count
```

## Collecting Logs for the Amazon CloudFront app

### Before you begin

Before you install the Amazon CloudFront app, complete the following tasks:

1. Enable [CloudFront logging](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html).
2. Confirm that access logs are being delivered to the Amazon S3 bucket.
3. [Grant Sumo Logic Access to the Amazon S3 Bucket](/docs/send-data/hosted-collectors/amazon-aws/grant-access-aws-product).

Once you begin uploading data, your daily data usage will increase. It's a good idea to check the **Account** page to make sure that you have enough quota to accommodate additional data in your account. If you need additional quota, you can [upgrade your account](/docs/manage/manage-subscription/upgrade-cloud-flex-account.md) at any time.

### Add an AWS Source

import Aws3 from '../../reuse/apps/create-aws-s3-source.md';

<Aws3/>

### Multiline Processing Boundary Regex Example

If your CloudFront log message is of this format:
```bash
2017-06-13    22:02:13    SYD1 ..............
```

You could use this Boundary Regex:
```bash
^\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2}:\d{2}.*
```

## Installing the Amazon CloudFront app

Now that you have set up collection for CloudFront, install the Sumo Logic app for Amazon CloudFront to use the pre-configured searches and dashboards that provide visibility into your environment for real-time analysis of overall usage.

import AppInstall from '../../reuse/apps/app-install.md';

<AppInstall/>

## Viewing Amazon CloudFront dashboards

Amazon CloudFront is a content delivery network (CDN) that allows an easy way for companies to distribute content to end-users with low latency and high data transfer speeds. The Sumo Logic application for Amazon CloudFront provides analytics on visitor information, rates and statistics, content being served, and other metrics. The app uses predefined searches and Dashboards that provide visibility into your environment for real-time analysis of overall usage.

### Overview

<img src={useBaseUrl('img/integrations/amazon-aws/AWS-Cloudfront-Overview.png')} style={{border: '1px solid gray'}} alt="Amazon CloudFront" />

- **Client Geo Distribution.** Performs a geo lookup search and displays visitor's client distribution for the last 24 hours on a map of the world.
- **Cache Hit and Miss.** Displays the cache's hits and misses over time in timeslices of five minutes for the last three hours in a pie chart.
- **HTTP Status Codes Over Time.** Shows HTTP status codes over time in timeslices of five minutes for the last three hours in a bar chart.
- **Visitor Access Platforms.** Provides information on the platforms that visitors use to access the site for the last three hours in a pie chart.
- **Requests Served by Edge Location.** Displays visitor requests served by edge location sorted by count for the last three hours in a pie chart.
- **Number of Unique Visitors.** Shows unique visitors to the site based on IP address over the last three hours in a single value chart.

### Latency Monitoring

<img src={useBaseUrl('img/integrations/amazon-aws/AWS-CloudFront-Latency-Monitoring.png')} style={{border: '1px solid gray'}} alt="Amazon CloudFront" />

- **Longest Latency by GeoLocation**. See the locations with long latency in the last hour on a world map.
- **90th 95th 99th Pct Time_taken Trend**. See the trend of 90th, 95th, and 99th percentiles of time taken in the last 24 hours on a line chart.
- **Outlier - Average Latency Time**. See the outlier of the average latency time in the last 24 hours on a line chart.
- **Outlier - Average Latency Time by Edge Location**. See the details of the outlier of average latency time in the last 24 hours such as the time, edge location, average time taken displayed in a table.
- **Average Latency Time by CloudFront Edge**. See the average latency time by Cloud Front Edge in the last 24 hours on a line chart.
- **Average Latency Time in Seconds by Region**. See the average latency time in seconds by region across the world in the last 24 hours on a bar chart.
- **Global Latency Time in Seconds**. See the global latency times in the last 24 hours on a line chart.

### Visitor Statistics

<img src={useBaseUrl('img/integrations/amazon-aws/Amazon-CloudFront-VisitorStatistics.png')} style={{border: '1px solid gray'}} alt="Amazon CloudFront" />

- **Client Geo Distribution.** Performs a geo lookup search and displays visitor's client distribution for the last 24 hours on a map of the world.
- **Requests Served by Edge Location.** Displays visitor requests served by edge location sorted by count for the last three hours in a pie chart.
- **Visitor Access Platforms.** Provides information on the platforms that visitors use to access the site for the last three hours in a pie chart.
- **Visitor Session Duration Distribution Histogram.** Displays the duration of visitor sessions distributed by count and bucket size in a histogram.
- **Unique Visitors Over Time.** Shows unique visitors to the site by based on IP address in timeslices of five minutes over the last three hours in a column chart.
- **Visitor Browsers and Devices.** Displays the devices and browsers, counted by platform, used by visitors to access the site over the last three hours in a stacked column chart.

### Web Operations

<img src={useBaseUrl('img/integrations/amazon-aws/Amazon-CloudFront-WebOperations.png')} style={{border: '1px solid gray'}} alt="Amazon CloudFront" />

- **Edge Result.** Displays edge results by count and sorted by type for the last three hours in a pie chart.
- **Client and Server Errors Over Time.** Shows client and server errors over time in timeslices of five minutes for the last three hours in a column chart.
- **HTTP Response Classes.** Provides HTTP response classes by count in timeslices of five minutes for the last three hours in a timeline.
- **Cache Hit and Miss Over Time.** Displays the cache's hits and misses over time in timeslices of five minutes for the last three hours in a stacked column chart.
- **HTTP Status Codes Over Time.** Shows HTTP status codes over time in timeslices of five minutes for the last three hours in a timeline.
- **Traffic and Megabytes Served.** Provides information on site traffic hits and Megabytes served in timeslices of one hour over the last 24 hours in a combination column and line chart.

## Additional logs and metrics collection (Optional)

### Log and Metric types

* [CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/monitoring-using-cloudwatch.html)
* [Real-time logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/real-time-logs.html)
* CloudWatch Logs
  * [Edge Function Logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-logs.html)
* [CloudTrail Logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/logging_using_cloudtrail.html)

### Configure metrics collection

* Collect **CloudWatch Metrics** with namespace [AWS/CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/monitoring-using-cloudwatch.html) using [AWS Kinesis Firehose for Metrics](/docs/send-data/hosted-collectors/amazon-aws/aws-kinesis-firehose-metrics-source/) source. For `AWS/CloudFront` metrics and dimensions, refer to [Amazon CloudFront CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/programming-cloudwatch-metrics.html).

### Configure logs collection

* Collect **real-time Logs** using the [Amazon S3](/docs/send-data/hosted-collectors/amazon-aws/aws-s3-source/) source if you have real-time logs destination set to S3 bucket. With CloudFront [real-time logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/real-time-logs.html), you can get information about requests made to distribution in real-time (logs are delivered within seconds of receiving the requests). You can use real-time logs to monitor, analyze, and take action based on content delivery performance.
* Collect **Amazon CloudWatch Logs** using the [AWS Kinesis Firehose for Logs](/docs/send-data/hosted-collectors/amazon-aws/aws-kinesis-firehose-logs-source/) source. You can use Amazon CloudWatch Logs to get logs for your [edge functions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/edge-functions-logs.html), both Lambda@Edge and CloudFront Functions.
* Collect [AWS CloudTrail Logs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/logging_using_cloudtrail.html) using the [AWS CloudTrail](/docs/send-data/hosted-collectors/amazon-aws/aws-cloudtrail-source/) source. Amazon CloudFront is integrated with CloudTrail, an AWS service that captures information about every request sent to the CloudFront API by your AWS account, including your IAM users. CloudTrail periodically saves log files of these requests to an Amazon S3 bucket that you specify. CloudTrail captures information about all requests, whether they were made using the CloudFront console, the CloudFront API, the AWS SDKs, the CloudFront CLI, or another service, for example, AWS CloudFormation.