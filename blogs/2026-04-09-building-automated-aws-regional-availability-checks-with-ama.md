---
title: "Building automated AWS Regional availability checks with Amazon S3"
url: "https://aws.amazon.com/blogs/storage/building-automated-aws-regional-availability-checks-with-amazon-s3/"
date: "Thu, 09 Apr 2026 18:42:19 +0000"
author: "Ajay Malalikar"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
<p>Every day, organizations expand into new markets, migrate critical workloads across geographies, and build systems that need to operate reliably in multiple locations. At the root of these efforts is a simple question: “What can I deploy, and where?” The answer shapes important architecture decisions, from which AWS Regions to expand into, to how you design for resilience. For data storage professionals, this question becomes even more critical. Data regulations and data residency requirements continue to change and grow more complex across geographies. As organizations expand, every team involved needs fast, reliable answers.</p> 
<p>At AWS, we publish comprehensive regional availability data covering services, features, APIs, and infrastructure resource types across all AWS Regions to help answer availability questions. You can explore this data interactively on the <a href="https://builder.aws.com/build/capabilities/explore?tab=service-feature" rel="noopener" target="_blank">AWS Capabilities by Region page</a>, or query it using the <a href="https://awslabs.github.io/mcp/servers/aws-knowledge-mcp-server/" rel="noopener" target="_blank">AWS Knowledge MCP Server</a>. These tools work well for exploration and planning. However, teams need this data in their own environments, accessible on their terms, shaped for specific use cases with the capability to integrate into the tools and workflows they already run. That means direct access to the underlying data, delivered through a storage solution they already use, like <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a>.</p> 
<p>In this post, we introduce a new data access option for AWS Regional availability hosted on S3 to help you copy the data directly into your account using <a href="https://docs.aws.amazon.com/AmazonS3/latest/API/Type_API_Reference.html" rel="noopener" target="_blank">AWS API</a> or <a href="https://docs.aws.amazon.com/cli/latest/reference/s3/" rel="noopener" target="_blank">AWS CLI</a>. This data is published through an <a href="https://aws.amazon.com/s3/features/access-points/" rel="noopener" target="_blank">S3 Access Point</a> and available to all authenticated <a href="https://docs.aws.amazon.com/mediapackage/latest/userguide/policy-principal.html" rel="noopener" target="_blank">AWS Principals</a>. You can use the same <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> policies, SDK integrations, and access patterns as you do for your own S3 data, and no onboarding is required. Because the data lives in S3, you can also land it directly in a data lake or sync it to your own bucket and query it with <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a>. We walk through how to configure IAM policy for principals, download structured datasets in JSON, CSV, and Apache Parquet formats, and integrate them into real workflows like pre-deployment template validation and regional expansion gap analysis. Whether you’re designing resilience across regions, navigating data residency requirements, or validating your infrastructure before deployment, this data is now available to download, store, and integrate using the S3 workflows you already run.</p> 
<h2>Available data</h2> 
<p>The data covers three main categories, each mapping to a different stage of your planning and deployment workflow. Use these to verify regional support before you commit to an architecture, write application code, or deploy infrastructure.</p> 
<table cellpadding="2" class=" aligncenter" style="height: 212px;" width="799"> 
 <tbody> 
  <tr> 
   <td><strong>Entity type</strong></td> 
   <td><strong>Description</strong></td> 
   <td><strong>Use case</strong></td> 
  </tr> 
  <tr> 
   <td>Services &amp; features (products)</td> 
   <td>Service availability and feature support by region</td> 
   <td>Verify S3 features before architecting storage</td> 
  </tr> 
  <tr> 
   <td>APIs (apis)</td> 
   <td>API operation availability by region</td> 
   <td>Check DynamoDB API support before writing application code</td> 
  </tr> 
  <tr> 
   <td>CloudFormation resources (cfn_resources)</td> 
   <td>Resource type support by region</td> 
   <td>Validate templates before deployment</td> 
  </tr> 
 </tbody> 
</table> 
<h2>Data formats</h2> 
<p>You can consume the data in whichever format best fits your workflow. Data is refreshed daily and published to the S3 Access Point. Each dataset includes a manifest.json with metadata, including the last updated timestamp, so you always know how fresh the data is. The Parquet format is particularly useful if you’re landing this data into a data lake or querying it with Athena, <a href="https://docs.aws.amazon.com/redshift/latest/dg/c-getting-started-using-spectrum.html" rel="noopener" target="_blank">Amazon Redshift Spectrum</a>, or similar analytics tools.</p> 
<table border="0" cellpadding="2" class=" aligncenter" style="height: 173px;" width="888"> 
 <tbody> 
  <tr> 
   <td><strong>Format</strong></td> 
   <td><strong>Status</strong></td> 
   <td><strong>Use case</strong></td> 
  </tr> 
  <tr> 
   <td>JSON</td> 
   <td>Available</td> 
   <td>Programmatic access, scripting, application integration</td> 
  </tr> 
  <tr> 
   <td>CSV</td> 
   <td>Available</td> 
   <td>Spreadsheet analysis, reporting, data import</td> 
  </tr> 
  <tr> 
   <td>Parquet</td> 
   <td>Available</td> 
   <td>Big data analytics, data lake integration</td> 
  </tr> 
  <tr> 
   <td>JSON-LD</td> 
   <td>Coming soon</td> 
   <td>Linked data applications, AI agent ingestion, structured context for large language models (LLMs)</td> 
  </tr> 
 </tbody> 
</table> 
<h2>Additional use cases</h2> 
<p>The examples covered in this post cover pre-deployment validation and regional expansion planning, but the data supports a wide range of use cases. Compliance teams can generate audit-ready reports on service availability across their operating Regions. Migration teams can identify alternative services when preferred options aren’t available in a target Region. Furthermore, because the data is structured and refreshed daily, you can build dashboards that track regional parity over time, turning a one-time analysis into continuous visibility.</p> 
<table border="0" cellpadding="2" class=" aligncenter" style="height: 181px;" width="921"> 
 <tbody> 
  <tr> 
   <td><strong>Use case</strong></td> 
   <td><strong>Description</strong></td> 
  </tr> 
  <tr> 
   <td>Pre-deployment validation</td> 
   <td>Integrate availability checks into CI/CD pipelines to catch compatibility issues before deployment</td> 
  </tr> 
  <tr> 
   <td>Regional expansion planning</td> 
   <td>Analyze capability gaps between regions before migrating workloads</td> 
  </tr> 
  <tr> 
   <td>Compliance and audit</td> 
   <td>Generate reports on service availability for compliance documentation</td> 
  </tr> 
  <tr> 
   <td>Migration planning</td> 
   <td>Identify alternative services when preferred options aren’t available in the target Region</td> 
  </tr> 
 </tbody> 
</table> 
<h2></h2> 
<h2>Solution overview</h2> 
<p>This solution provides programmatic access to AWS Regional availability data through a public S3 Access Point, requiring only standard AWS authentication and three straightforward steps.</p> 
<p><img alt="Data Layer" class="aligncenter wp-image-28684" height="296" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/08/Data-Layer.png" width="742" /></p> 
<h3>Prerequisites</h3> 
<p>To follow along, you need:</p> 
<ul> 
 <li>An AWS account with IAM permissions to call S3 <code>GetObject</code></li> 
 <li>AWS CLI installed and configured</li> 
 <li>Python 3.10+ and PyYAML (`pip install pyyaml`) for the integration examples</li> 
 <li>An <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template (YAML) if you want to run Example 1 against your own infrastructure</li> 
</ul> 
<h3>Step 1: Configure IAM permissions</h3> 
<p>Add the following policy to your IAM role or user to allow access to the data:</p> 
<pre><code class="lang-json">{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "s3:DataAccessPointArn": "arn:aws:s3:us-east-1:686591367145:accesspoint/aws-capabilities-public"
    }
  }
}</code></pre> 
<p><strong>Note:</strong> This step is optional if your IAM role or user already has <code>s3:GetObject</code> permissions on all S3 resources.</p> 
<h3>Step 2: Access the data</h3> 
<p>Start by downloading the index to see all available versions and the latest version, then download the manifest to discover available files, check the latest updated timestamp, and understand the folder structure.</p> 
<pre># Download index
aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/index.json -

# Download manifest
aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/manifest.json -</pre> 
<p><strong>Note:</strong> Ensure your CLI is configured with credentials that have the permissions described in Step 1 (or equivalent to S3 read access).</p> 
<p>Next, download the data files you need.</p> 
<pre># Download service availability data
aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/json/products.json -
# Download CloudFormation Resources availability data
aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/json/cfn_resources.json -
# Download API availability data
aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/json/apis.json -
# Download as CSV for spreadsheet analysis
aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/csv/products.csv -</pre> 
<h3>Step 3: Integrate with your workflows</h3> 
<p>With the data downloaded, you can integrate it into existing tools and processes. The following examples show two common use cases.</p> 
<h3>Integration examples</h3> 
<p>First, download the data files using the following CLI commands.</p> 
<pre>aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/json/cfn_resources.json -

aws s3 cp s3://aws-capabilities-pub-ybkxdwgxrkfhwmq8b1neoq8ny6ua4use1b-s3alias/public/v1/json/products.json -</pre> 
<h4>Example 1: Validate CloudFormation template before deployment</h4> 
<pre><code class="lang-python">import json
import yaml
 
# Load availability data and CloudFormation template
 with open('cfn_resources.json') as f:
     cfn_availability = json.load(f)
 
 with open('my-infrastructure.yaml') as f:
     template = yaml.safe_load(f)
 
target_region = 'eu-west-1'
 
# Build set of available resource types for target Region
available_types = set()
for service in cfn_availability:
    for rt in service.get('resourceTypes', []):
        if rt.get('regionalAvailability', {}).get(target_region) == 'Available':
            available_types.add(f"AWS::{service['serviceName']}::{rt['resourceTypeName']}")
 
# Check each resource in template
 print(f"Validating template for {target_region}:\n")
 for resource_name, resource_def in template.get('Resources', {}).items():
     resource_type = resource_def.get('Type')
     if resource_type in available_types:
         print(f" ✅ {resource_name}: {resource_type}")
     else:
         print(f" ❌ {resource_name}: {resource_type} - NOT AVAILABLE")</code></pre> 
<p>This represents a typical example of the validation results generated by the automated system.</p> 
<p>The following is the validating template for eu-west-1:</p> 
<pre>&nbsp; ✅ MyLambdaFunction: AWS::Lambda::Function
&nbsp;&nbsp;✅ MyDynamoDBTable: AWS::DynamoDB::Table
&nbsp;&nbsp;❌ MySDCDeployment: AWS::SDC::Deployment - NOT AVAILABLE</pre> 
<h4>Example 2: Regional expansion gap analysis</h4> 
<p>Compare your source Region against a target Region to identify capability gaps before migration.</p> 
<pre><code class="lang-python">import json
 
with open('products.json') as f:
     products = json.load(f)
 
source_region = 'us-east-1'
 target_region = 'ap-southeast-1'
 
# Get available services in each Region
 def get_available_services(region):
     return {
         p['productName'] for p in products
         if p.get('regionalAvailability', {}).get(region) == 'Available'
     }
 
source_services = get_available_services(source_region)
 target_services = get_available_services(target_region)
 
# Identify gaps
 gaps = source_services - target_services
 common = source_services &amp; target_services
 
print(f"Source Region ({source_region}): {len(source_services)} services")
 print(f"Target Region ({target_region}): {len(target_services)} services")
 print(f"Common services: {len(common)}")
 print(f"\nServices NOT available in target Region:")
 for service in sorted(gaps):
     print(f"  - {service}")</code></pre> 
<h2>Considerations</h2> 
<p>Before implementing this solution, keep the following operational details in mind:</p> 
<ul> 
 <li><strong>Pricing:</strong> There is no additional charge for accessing the availability data. Standard AWS authentication is required, but no data transfer or request fees apply to the consumer.</li> 
 <li><strong>Data freshness:</strong> Availability data is refreshed daily. Check <code>manifest.json</code> for the <code>last_updated</code> timestamp.</li> 
 <li><strong>Security:</strong> Access is restricted to authenticated AWS principals only.</li> 
 <li><strong>Feedback:</strong> For data accuracy issues or feature requests, contact your account team or file a support case.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this post, we showed you how to access AWS Regional availability data programmatically through Amazon S3, configure IAM permissions for S3 Access Points, and integrate this data into CloudFormation validation and Regional expansion workflows. No cleanup is required, as this solution only reads publicly available data and does not create any resources in your account. We built this because we believe availability data should be as easy to consume programmatically as it is to browse interactively. Whether you’re validating templates in CI/CD, planning a multi-Region migration, or building governance tooling, the data is there for you—structured, fresh, and ready to integrate.</p> 
<p>To get started, configure your IAM permissions and download the manifest (see Step 2). We look forward to seeing what you build.</p>
