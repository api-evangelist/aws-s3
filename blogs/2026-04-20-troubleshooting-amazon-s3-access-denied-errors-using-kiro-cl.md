---
title: "Troubleshooting Amazon S3 access denied errors using Kiro CLI"
url: "https://aws.amazon.com/blogs/storage/troubleshooting-amazon-s3-access-denied-errors-using-kiro-cli/"
date: "Mon, 20 Apr 2026 18:33:33 +0000"
author: "Gopinath Jeganathan"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
<p>Managing data access across multiple layers of permissions is a common industry challenge. Changes to <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management</a> (AWS IAM) policies, <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket configurations, <a href="https://aws.amazon.com/kms/" rel="noopener noreferrer" target="_blank">AWS Key Management Service</a> (AWS KMS) key policies, or <a href="https://aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon Virtual Private Cloud</a> (Amazon VPC) endpoint policies can unintentionally cause access issues. When these access interruptions happen, they can halt critical business processes and affect multiple teams simultaneously, making rapid diagnosis and resolution essential for maintaining business continuity and reducing operational costs. Traditional troubleshooting approaches require sequential analysis of resource-based policies, identity-based policies, encryption settings, and VPC endpoint policies, which can create significant operational overhead as teams work to identify the root cause and implement appropriate fixes.</p> 
<p>S3 serves as the backbone of AWS infrastructure, with services like <a href="https://aws.amazon.com/lambda/" rel="noopener noreferrer" target="_blank">AWS Lambda</a>, <a href="https://aws.amazon.com/ec2/" rel="noopener noreferrer" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) and third-party applications relying on it for data storage, backup and content delivery. <a href="https://kiro.dev/cli/" rel="noopener noreferrer" target="_blank">Kiro CLI</a> is an AI-powered command-line tool that transforms complex AWS troubleshooting through intelligent analysis of service configurations, policies, and permissions across multiple AWS services. Traditional S3 access denied troubleshooting requires manually analyzing complex IAM policies, bucket configurations KMS permissions, and security controls across different layers, which can take hours to resolve and demands deep expertise in AWS permission models.</p> 
<p>In this post, we discuss how Kiro CLI simplifies troubleshooting S3 access denied issues by walking through three real world scenarios: explicit deny statements in bucket policies, implicit deny statements from missing KMS permissions, and VPC endpoint policy restrictions. You will learn how Kiro CLI systematically analyzes the complete permission chain, including IAM policies, bucket configurations, encryption settings, and network controls, to quickly identify root causes and provide actionable remediation guidance.</p> 
<h2>Solution overview</h2> 
<p>This solution uses the enhanced error messages returned by <a href="https://aws.amazon.com/about-aws/whats-new/2025/06/amazon-s3-context-http-403-access-denied-error-message-aws-organizations/" rel="noopener noreferrer" target="_blank">Amazon S3 for access denied errors within the same organization</a>. Kiro CLI transforms the complex process of troubleshooting S3 access denied errors into an intelligent, step-by-step diagnostic process through systematic analysis of the complete permission chain.</p> 
<p>This post demonstrates Kiro CLI’s capabilities through three comprehensive scenarios:</p> 
<ul> 
 <li><strong>Explicit deny statements</strong> – Where bucket policies override IAM permissions</li> 
 <li><strong>Implicit deny scenarios</strong> – Involving missing AWS KMS permissions for encrypted objects</li> 
 <li><strong>Security control conflicts</strong> – VPC endpoint policies restrict S3 access despite proper IAM and bucket permissions</li> 
</ul> 
<p>Kiro CLI automatically detects permission issues across multiple layers, from explicit policy denials to encryption permissions and account-level security controls, providing clear explanations of why access is being blocked along with guided remediation steps. This approach saves significant troubleshooting time compared to manual analysis, while offering structured remediation guidance for both common and complex S3 access scenarios.After we walk you through launching Kiro CLI, we explore how to troubleshooting the following S3 access denied issues:</p> 
<ul> 
 <li><strong>Scenario 1</strong> – Troubleshoot explicit deny statements</li> 
 <li><strong>Scenario 2</strong> – Troubleshoot implicit deny statements</li> 
 <li><strong>Scenario 3</strong> – Troubleshooting restricting using an S3 VPC Endpoint policy</li> 
</ul> 
<h2>Prerequisites</h2> 
<p>To get started, complete the following prerequisite steps:</p> 
<ol> 
 <li>If you’re new to Kiro CLI, refer to <a href="https://kiro.dev/docs/cli/" rel="noopener noreferrer" target="_blank">Get started</a> in the Kiro documentation for initial setup guidance.</li> 
 <li>Installation of Kiro CLI: 
  <ul> 
   <li>Follow the installation steps in the <a href="https://kiro.dev/docs/cli/installation/" rel="noopener noreferrer" target="_blank">documentation</a> based on your operating system</li> 
   <li>Verify the installation by running kiro –version in your terminal</li> 
  </ul> </li> 
 <li>AWS credentials configuration: 
  <ul> 
   <li>Configure your AWS credentials using aws configure</li> 
   <li>The configured AWS profile will be used by Kiro CLI to validate S3 access permissions, and if you want to use a profile other than the default, you can reference specific profiles within your queries.</li> 
   <li>If you’re troubleshooting permissions for a different role: 
    <ul> 
     <li>Ensure your credentials can assume the target IAM role</li> 
     <li>For validating permissions of another role, refer to the <a href="https://repost.aws/knowledge-center/iam-assume-role-cli" rel="noopener noreferrer" target="_blank">re:Post article</a> for detailed steps to configure AWS credentials with role assumption.</li> 
    </ul> </li> 
  </ul> </li> 
</ol> 
<p>Although this post uses IAM user credentials for simplicity, production environments should use temporary credentials, such as AWS Identity Center or AWS Security Token Service(STS) assume-role, with MFA enabled. If using long-term IAM user access keys, enable MFA and rotate keys every 90 days maximum. Kiro CLI can suggest policy changes and request permission to implement them; you can approve or deny each change. However, approving automated policy modifications can grant unintended permissions. Always review suggested changes carefully to understand the security impact, test in non-production environments, and follow your security review processes before approving changes to production. This post demonstrates Kiro CLI’s diagnostic capabilities only, not automated remediation.</p> 
<h2>Launch Kiro CLI</h2> 
<p>Open your terminal, verify your AWS identity using aws sts get-caller-identity, then launch Kiro CLI by executing the command kiro-cli chat.</p> 
<p><img alt="Amazon Q Kiro CLI chat interface showing ASCII art logo with tip about defining indexed resources for agent configuration auto-sync" class="aligncenter wp-image-28762 size-full" height="584" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16521.png" width="964" /></p> 
<p>When troubleshooting S3 access denied issues, Kiro CLI conducts a systematic analysis by examining multiple configuration layers including bucket policies, IAM policies, object ownership settings, KMS key policies, and VPC endpoint policies. During this diagnostic process, Kiro CLI will request your permission to run specific AWS commands to gather the necessary configuration data.</p> 
<p>When prompted, carefully review each permission request and confirm by entering y if you approve. We recommend using y (yes) instead of t (trust) in this scenario because y makes it possible to validate each change to your security policies before its made.</p> 
<p>The following is an example permission prompt:</p> 
<p><img alt="AWS CLI security prompt requesting user confirmation to execute s3control get-public-access-block command for account 123456789012 in us-east-1 region" class="aligncenter wp-image-28761 size-full" height="272" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16522.png" width="1024" /></p> 
<p>Kiro CLI systematically executes a series of AWS commands to analyze permissions across multiple layers, including bucket policies, IAM roles, encryption settings, and access controls. Based on your specific query, it provides comprehensive troubleshooting along with a detailed root cause analysis.</p> 
<p>Kiro CLI uses low-level S3 API calls (such as “GetObject”) rather than high-level CLI commands (such as “aws s3 cp”) during its diagnostic process. Low-level API calls return more detailed error messages that include specific policy evaluation context, enabling Kiro CLI to pinpoint the exact permission layer causing the denial.</p> 
<p>The following scenarios demonstrate how Kiro CLI diagnoses three common types of S3 access denied errors, each caused by a different layer in the AWS permission stack. We start with an explicit deny in a bucket policy, move to a missing AWS KMS decryption permission, and conclude with a restrictive VPC endpoint policy.</p> 
<h3><strong>Scenario 1: Troubleshoot Explicit Deny Statements</strong></h3> 
<p>In this scenario, a developer with permissions to download a file from a specific S3 bucket is unable to download the file. This represents a common situation where explicit deny statements override allow permissions.</p> 
<p>When attempting to download a file from the S3 bucket amzn-s3-demo-company-reports, the developer encounters an access denied error:</p> 
<p><img alt="AWS CLI command showing 403 Forbidden error when attempting to copy HalfyearReports.pdf from S3 bucket amzn-s3-demo-company-reports due to insufficient permissions" class="aligncenter wp-image-28760 size-full" height="80" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16523.png" width="985" /></p> 
<p>Instead of manually examining multiple IAM policies and bucket configurations, the developer uses Kiro CLI to diagnose the issue with a natural language query.</p> 
<p><img alt="Terminal prompt showing user query asking AI assistant to analyze S3 permission issues preventing download of HalfyearReports.pdf from amzn-s3-demo-company-reports bucket" class="aligncenter wp-image-28759 size-full" height="69" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16524.png" width="1238" /></p> 
<h4>Kiro CLI provides the following root cause analysis:</h4> 
<pre><code class="lang-bash">## Analysis Summary

Root Cause: The primary issue preventing you from downloading the S3 object is an explicit DENY policy at the bucket level.

### Permission Structure Analysis:

1. Bucket Policy (BLOCKING ACCESS):
  • Contains an explicit DENY statement for s3:GetObject action
  • Applies to one principal ("Principal": "arn:aws:iam::123456789012:user/johndoe")
  • Covers all objects in the bucket ("Resource": "arn:aws:s3::: amzn-s3-demo-company-reports/*")

2. Your IAM Permissions (SUFFICIENT):
  • You have s3:GetObject permission in your inline policy
  • Policy correctly targets the amzn-s3-demo-company-reports bucket
  • IAM permissions would normally allow the download

3. Bucket Policy Override:
  • Explicit DENY in bucket policy overrides any ALLOW in IAM policies
  • This is AWS's fundamental security principle: DENY always wins

### Key Finding:
Even though you have the required IAM permissions, explicit DENY statements in resource policies always override ALLOW statements, 
regardless of IAM permissions.

### Resolution Options:
1. Remove or modify the bucket policy to allow your access
2. Contact the bucket owner to grant you access  
3. Add an exception to the DENY policy for your user ARN

The bucket policy is the definitive blocker preventing your download access.
</code></pre> 
<h3><strong>Scenario 2: Troubleshooting Implicit Deny Statements</strong></h3> 
<p>In this scenario, a developer has sufficient S3 permissions but encounters access issues when trying to download an encrypted object, illustrating an implicit deny scenario where the absence of required AWS KMS permissions blocks access despite no explicit deny statements. This demonstrates how S3 access can be affected by permissions in related services like AWS KMS, highlighting the importance of comprehensive cross-service permission analysis.</p> 
<p>When attempting to download a file from the S3 bucket amzn-s3-demo-company-reports, the developer encounters an access denied error:</p> 
<p><img alt="AWS CLI error showing AccessDenied when downloading encrypted S3 object due to missing kms:Decrypt permission for user UserA on KMS key" class="aligncenter wp-image-28758 size-full" height="134" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16525.png" width="1240" /></p> 
<h2><img alt="Terminal showing AI assistant processing user query at 10% to analyze S3 permission structure and identify root cause for HalfyearReports.pdf download failure" class="aligncenter wp-image-28757 size-full" height="78" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16526.png" width="1239" /></h2> 
<h4>Kiro CLI provides the following root cause analysis:</h4> 
<pre><code class="lang-bash">## Analysis Summary

Root Cause: KMS Key Permission Issue

The download failure is caused by insufficient KMS permissions, not S3 permissions. Here's the complete analysis:

### Permission Structure Analysis

S3 Permissions (Sufficient):
• Bucket ACL: You have FULL_CONTROL on the bucket
• Object ACL: You have FULL_CONTROL on the object
• No restrictive bucket policy exists
• Your AWS identity matches the bucket/object owner

KMS Permissions (Insufficient):
• Object is encrypted with KMS key: arn:aws:kms:us-east-1:123456789012:key/ab2c3d4e-e5f6-7890-abcd-ef1234567890. Your user arn:aws:iam::123456789012:user/UserA lacks kms:Decrypt permission
• Cannot even describe the key (lacks kms:DescribeKey permission)

### Root Cause
The object is encrypted with AWS KMS, but your IAM user doesn't have the required KMS permissions to decrypt it. Even though you own the S3 object, KMS uses separate permissions that must be explicitly granted.

### Required Actions
To fix this, you need one of the following:

1. Add KMS permissions to your IAM user/role:
  
json
   {
     "Effect": "Allow",
     "Action": [
       "kms:Decrypt",
       "kms:DescribeKey"
     ],
     "Resource": "arn:aws:kms:us-east-1:123456789012:key/ab2c3d4e-e5f6-7890-abcd-ef1234567890"
   }
   

2. Update the KMS key policy to allow your user access

3. Contact the key administrator to grant you decrypt permissions

The error confirms this: User: arn:aws:iam::123456789012:user/UserA is not authorized to perform: kms:Decrypt on resource
</code></pre> 
<h3><strong>Scenario 3: Troubleshoot restrictions using a S3 VPC endpoint policy</strong></h3> 
<p>In this scenario, an EC2 instance with a privileged IAM role encounters unexpected access denied errors when attempting to download objects from an S3 bucket. The instance routes S3 traffic through a gateway VPC endpoint, whose restrictive policy overrides the IAM permissions. This illustrates how VPC endpoint policies act as an additional authorization layer that can block access regardless of IAM privileges. Understanding the interplay between IAM policies, network paths, and endpoint policies is essential when troubleshooting S3 connectivity issues.<strong><br /> </strong></p> 
<p>When attempting to download a file from the S3 bucket amzn-s3-demo-company-reports, the user encounters a forbidden error:</p> 
<p><img alt="AWS CLI error showing AccessDenied when calling GetObject operation to download HalfyearReports.pdf from S3 bucket amzn-s3-demo-company-reports" class="aligncenter wp-image-28756 size-full" height="94" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16527.png" width="1237" /></p> 
<p>VPC endpoint policies operate outside the typical IAM and bucket policy evaluation path, making them one of the hardest access denied root causes to identify. The developer turns to Kiro CLI to surface this network-level restriction:</p> 
<p><img alt="Two-part terminal screenshot showing AWS S3 AccessDenied error followed by AI assistant at 15% analyzing permission structure and security risks for HalfyearReports.pdf download failure" class="aligncenter wp-image-28755 size-full" height="83" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/15/image-aws-blog-storage-16528.png" width="1241" /></p> 
<h4>Kiro CLI provides the following root cause analysis:</h4> 
<pre><code class="lang-bash">## Analysis Summary

Based on my investigation, I've identified the root cause of why you can't download the S3 object s3://amzn-s3-demo-company-reports/HalfyearReports.pdf:

### Root Cause: VPC Endpoint Policy Restriction

The error message consistently shows: "no VPC endpoint policy allows the s3:GetObject action"

This indicates that:

1. Your EC2 instance is in a VPC that routes S3 traffic through a VPC endpoint
2. The VPC endpoint has a restrictive policy that explicitly denies S3 actions including:
   • s3:GetObject (downloading objects)
   • s3:GetBucketPolicy (viewing bucket policies)
   • s3:GetObjectAcl (viewing object ACLs)
   • s3:ListBucket (listing bucket contents)

### Permission Analysis

Your IAM role (EC2s3Admin) has sufficient permissions:
• AmazonS3ManagedAccess policy (S3 Download access)

However, VPC endpoint policies override IAM permissions when traffic routes through the endpoint.

### Required Actions to Fix

To resolve this issue, you need to:

1. Update the VPC endpoint policy to allow S3 actions:
  
json
   {
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
        	  "AWS": "arn:aws:iam::123456789012:user/johndoe"
         },
         "Action": [
           "s3:GetObject",
           "s3:GetBucketPolicy",
           "s3:ListBucket",
           "s3:GetObjectAcl"
         ],
         "Resource": [
           "arn:aws:s3:::amzn-s3-demo-company-reports",
           "arn:aws:s3:::amzn-s3-demo-company-reports/*"
         ]
       }
     ]
   }
   
2. Or modify the route table to bypass the VPC endpoint for S3 traffic (routes through internet gateway instead)

The VPC endpoint policy is the bottleneck preventing access, despite having sufficient IAM permissions for S3 operations. This is not a recommended option as the traffic is not traversed through secure VPC Endpoint.</code></pre> 
<h2>Conclusion</h2> 
<p>The troubleshooting methodology demonstrated in this blog post extends far beyond S3 access issues, offering a blueprint for diagnosing challenges across the entire AWS service ecosystem. Kiro CLI’s intelligent analysis can be similarly applied to troubleshoot issues in Amazon Relational Database Service (Amazon RDS), EC2 instances, Lambda, and other AWS services, following a similar systematic approach. By adopting Kiro CLI as a standard component of your AWS operational toolkit, organizations can significantly reduce resolution times and minimize service disruptions.</p> 
<p>Try out Kiro CLI for your own troubleshooting use cases, and share your questions and feedback in the comments.</p>
