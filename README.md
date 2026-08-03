# Amazon S3 API

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
