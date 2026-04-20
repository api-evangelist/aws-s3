# Amazon S3 API

Amazon Simple Storage Service (S3) is an object storage service offering industry-leading scalability, data availability, security, and performance for storing and retrieving any amount of data.

## APIs

### Amazon S3 REST API
RESTful API for bucket management, object CRUD, access control, versioning, lifecycle policies, and multipart uploads.
- **Documentation**: https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html
- **OpenAPI**: [openapi/aws-s3-openapi.yaml](openapi/aws-s3-openapi.yaml) (97 operations)
- **Authentication**: https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html

## Artifacts

| Directory | Contents |
|---|---|
| [openapi/](openapi/) | 1 OpenAPI specification (97 operations) |
| [json-schema/](json-schema/) | 574 JSON Schema files |
| [json-structure/](json-structure/) | 574 JSON Structure files |
| [json-ld/](json-ld/) | 1 JSON-LD context file |
| [examples/](examples/) | 574 example files |
| [rules/](rules/) | Spectral ruleset |
| [capabilities/](capabilities/) | Naftiko capability definitions |
| [vocabulary/](vocabulary/) | Domain vocabulary |

## Features

- **Scalable Object Storage** — Store and retrieve any amount of data at any time from anywhere.
- **Versioning** — Keep multiple variants of an object in the same bucket.
- **Lifecycle Policies** — Automatically transition or expire objects.
- **Cross-Region Replication** — Automatically replicate objects across AWS Regions.
- **Server-Side Encryption** — Encrypt objects at rest using SSE-S3, SSE-KMS, or SSE-C.
- **Access Control** — Fine-grained access via bucket policies, ACLs, and IAM policies.
- **Event Notifications** — Trigger Lambda, SQS, or SNS on bucket events.
- **S3 Select** — Retrieve a subset of data using SQL expressions.
- **Transfer Acceleration** — Speed up uploads using CloudFront edge locations.
- **Intelligent-Tiering** — Automatically optimize storage costs based on access patterns.

## Use Cases

- **Data Lake Storage** — Store raw data for analytics with Glue, Athena, and Redshift Spectrum.
- **Static Website Hosting** — Host static websites and single-page applications.
- **Backup and Archive** — Store backups with lifecycle policies to Glacier for long-term retention.
- **Media Storage and Distribution** — Serve images, videos, and documents globally via CloudFront.
- **Application Data Storage** — Store user-generated content, logs, and configuration files.

## Links

- **Website**: https://aws.amazon.com/s3/
- **Getting Started**: https://aws.amazon.com/s3/getting-started/
- **Pricing**: https://aws.amazon.com/s3/pricing/
- **Console**: https://console.aws.amazon.com/s3/
- **Blog**: https://aws.amazon.com/blogs/storage/
- **Change Log**: https://aws.amazon.com/releasenotes/Amazon-S3/
- **Status**: https://health.aws.amazon.com/health/status

## Maintainers

- **Kin Lane** — kin@apievangelist.com
