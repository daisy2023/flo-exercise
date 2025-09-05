Automated Reporting Feature – System Design
Overview

We are extending our product with an automated reporting feature. Users participating in clinical trials need to receive weekly PDF summaries that aggregate database records and related files (text, images). These reports are generated via a long-running background process and delivered via email.

The system is designed for a large user base and heavy usage, with reliability, scalability, and fault tolerance as primary goals.

🎯 User Stories

As a user, I want to subscribe to a weekly report.

As a user, I want to receive the report via email.

🔑 Assumptions

File data is stored in AWS S3, metadata in NoSQL (MongoDB).

Reports require complex data aggregation and image/PDF processing.

The system must handle tens of thousands of concurrent report jobs.

Failures must not block processing → handled via retries + DLQs.

🏗️ High-Level Architecture


⚙️ Flow Description

Subscription

User clicks Subscribe.

API validates permissions and saves subscription metadata.

Job Scheduling

A scheduler runs periodically.

It checks subscriptions due for execution (createdAt <= now) and pushes jobs to the Job Queue.

Workers & Queues

Fetch Worker: retrieves documents & files → pushes results to Aggregation Queue.

Aggregation Worker: aggregates tabular + file data → pushes results to Processing Queue.

Image/PDF Worker: generates PDF reports, stores them in S3 → pushes results to Notification Queue.

Notification Worker: generates a secure S3 link → calls Email Service → delivers report.

Failure Handling

Each queue is backed by a Dead Letter Queue (DLQ).

Workers retry jobs automatically.

Persistent failures go to DLQ for investigation without blocking the pipeline.

File–Metadata Consistency

Files are uploaded to S3 first.

Metadata in NoSQL updated only after a successful upload.

Background reconciliation jobs compare S3 vs DB for consistency.

✅ Best Practices Applied

Scalability: Independent workers, horizontal scaling based on queue depth.

Reliability: DLQs, retries, idempotent job execution.

Performance: Queues buffer workload, workers process asynchronously.

Security: Reports delivered via pre-signed S3 links + encrypted at rest and in transit.

Observability: Monitor queue depth, worker health, failure rates.

Next Steps for Implementation

Define queue structure (separate SQS queues for fetch, aggregation, processing, notification).

Implement workers as stateless services (e.g., AWS Lambda, ECS, or Kubernetes Jobs).

Add monitoring/alerting (CloudWatch, Prometheus, or ELK).

Set up retention policies for S3 report files.

Define retry policies and DLQ handling.
