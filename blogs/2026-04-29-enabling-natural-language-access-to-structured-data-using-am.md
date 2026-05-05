---
title: "Enabling natural language access to structured data using Amazon S3 Tables and Amazon Bedrock Knowledge Bases"
url: "https://aws.amazon.com/blogs/storage/enabling-natural-language-access-to-structured-data-using-amazon-s3-tables-and-amazon-bedrock-knowledge-bases/"
date: "Wed, 29 Apr 2026 19:10:21 +0000"
author: "Jane Ridge"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
<p>Organizations generate massive volumes of structured data from customer transactions, operational metrics, product catalogs, and compliance records. This data contains insights that can help businesses make better and timely decisions. Financial advisors need to review client transaction histories, retail analysts track inventory trends, and healthcare administrators monitor patient outcomes. Yet accessing these insights creates a bottleneck. Business users must either learn SQL, wait for technical teams to run queries, or rely on pre-built dashboards that may not answer their specific questions. These barriers slow decision-making as data scales.</p> 
<p><a href="https://aws.amazon.com/s3/features/tables/" rel="noopener" target="_blank">Amazon S3 Tables</a> provide managed storage for structured data using <a href="https://iceberg.apache.org/" rel="noopener" target="_blank">Apache Iceberg</a>, delivering scalable analytics without operational overhead. When integrated with <a href="https://aws.amazon.com/bedrock/knowledge-bases/" rel="noopener" target="_blank">Amazon Bedrock Knowledge Bases</a>, this architecture enables natural language querying. Business users can ask questions in plain English and receive accurate answers from governed data sources. No SQL expertise required, no waiting for technical teams, just conversational access to real-time insights.</p> 
<p>In this post we walk through implementing this solution using a financial services example. You’ll set up S3 Tables, configure <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/serverless-etl-aws-glue/aws-glue-data-catalog.html" rel="noopener" target="_blank">AWS Glue Data Catalog</a> integration, connect <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-serverless.html" rel="noopener" target="_blank">Amazon Redshift Serverless</a>, and create a Bedrock Knowledge Base that translates questions like “Show transactions for client 12345” into optimized queries against your structured data.</p> 
<h2>Solution benefits</h2> 
<p>This architecture enables secure, natural language querying of structured data. It integrates storage, analytics, and AI to deliver faster insights, lower costs, and clearer operations</p> 
<ul> 
 <li>Faster decisions: Real-time analytics and natural language queries shorten the path from question to insight</li> 
 <li>Lower costs: Pay-as-you-go pricing, efficient storage, and optimized queries reduce both storage and compute spend</li> 
 <li>Broader access: Non-technical users can analyze data through conversational AI with no SQL required</li> 
 <li>Stronger governance: Fine-grained permissions and auditing through <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> keep data secure and compliant</li> 
 <li>Less operational effort: Automated compaction and metadata management reduce ongoing maintenance</li> 
 <li>High reliability: Built on the durability and availability of <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> for enterprise-grade workloads</li> 
</ul> 
<h2>Solution overview</h2> 
<p>S3 Tables serve as the foundation for storage and analytics, integrating with <a href="https://aws.amazon.com/redshift/" rel="noopener" target="_blank">Amazon Redshift</a> and the Glue Data Catalog to enable natural language queries through Bedrock Knowledge Bases. User questions are translated into structured queries against S3 Tables, returning governed, real-time insights from financial data.</p> 
<p>The following architecture diagram shows this solution.</p> 
<p><img alt="Architecture overview for natural language querying of data using Amazon Bedrock, Amazon Redshift, and Amazon S3 Tables" class="aligncenter wp-image-28955 size-full" height="922" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/ArchitectureDiagram.png" width="1430" /></p> 
<h2>How it works</h2> 
<p>The following steps describe how the solution processes a natural language query end to end:</p> 
<ol> 
 <li>Natural language query: A financial user asks a question in natural language (for example, “Show last 5 transactions for client 12345”). The identity is validated through <a href="https://aws.amazon.com/iam/identity-center/" rel="noopener" target="_blank">AWS IAM Identity Center</a> for secure, role-based access.</li> 
 <li>Bedrock Knowledge Bases: <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> uses <a href="https://aws.amazon.com/what-is/retrieval-augmented-generation/" rel="noopener" target="_blank">Retrieval Augmented Generation (RAG)</a> to access schema metadata and generates a SQL query specifically formatted for Redshift, referencing Glue Data Catalog.</li> 
 <li><a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a> metadata lookup: Redshift queries the S3 Table Data Catalog for schema and table definitions, supporting table versioning and column-level visibility for Iceberg tables.</li> 
 <li>Querying S3 Tables: Redshift runs the generated query against S3 Tables in the Iceberg format.</li> 
 <li>Returning results: Query results are securely returned to Bedrock in a tabular format.</li> 
 <li>Natural language answer: Bedrock synthesizes results into a clear, natural language response for the user (for example, “Client 12345 spent $620 at ABC Store on June 10”).</li> 
</ol> 
<h2>Solution walkthrough</h2> 
<p>The following steps guide you through setting up natural language access to your structured data. You create a table bucket, configure Glue Data Catalog integration, set up Redshift, create a Bedrock knowledge base, and configure the necessary permissions to enable secure querying.</p> 
<h3>Step 1. Set up your table bucket</h3> 
<p>This first step establishes your S3 Tables infrastructure foundation.</p> 
<p>1.1. Create your <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-buckets-create.html" rel="noopener" target="_blank">S3 Tables resource</a>.</p> 
<p>1.2. Create a new table bucket, <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-namespace.html" rel="noopener" target="_blank">setting up a namespace</a> within the bucket.</p> 
<p><img alt="Creating a table in Amazon Athena using an existing namespace in a table bucket." class="aligncenter size-full wp-image-28958" height="408" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/SpecifyNamespace-s3Tables.png" width="738" /></p> 
<p>1.3. When you have your table bucket set up, create the table within your S3 table bucket using <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a>, as shown in the preceding figure. You must set up the <a href="https://docs.aws.amazon.com/athena/latest/ug/querying.html" rel="noopener" target="_blank">Athena Query Result Location</a> before you can perform query functions.</p> 
<p>1.4. Open Athena Query Editor, select <strong>AwsDataCatalog</strong> as data source.</p> 
<pre><code class="lang-sql">-- create setup of table structure
CREATE TABLE my_sample_fin_data.fin_table (
	transaction_date		      date,
	account_id			      string,
	customer_name		      string,
	amount			            	double,
    	transaction_type		    	string,
    	balance_after_transaction	double,
    	credit_card_eligibility		Boolean
)
PARTITIONED BY (transaction_date)
TBLPROPERTIES ('table_type'='iceberg')</code></pre> 
<p>Before running, replace placeholder variables <code>my_sample_fin_data</code> and <code>fin_table</code> with your actual namespace and table names. Your setup should look like the following output.</p> 
<p><img alt="Creating an Iceberg table in Amazon Athena using SQL" class="aligncenter size-full wp-image-28953" height="564" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/CreateTable-Athena.png" width="936" /></p> 
<p>1.5. Next, insert the sample financial transaction data below (or create your own) using the example below. This includes account IDs, customer names, transaction amounts, types, and balances representing typical retail banking data.</p> 
<pre><code class="lang-sql">-- populate table with sample financial data
INSERT INTO &lt;s3_bucket_table_namespace&gt;.&lt;s3_bucket_table_name&gt;
(transaction_date, account_id, customer_name, amount, transaction_type, balance_after_transaction, credit_card_eligibility)
VALUES
(DATE '2023-04-10', 'ACC018', 'John Smith', 100.00, 'debit', 900.00, false),
(DATE '2023-04-13', 'ACC018', 'John Smith', 250.00, 'credit', 1150.00, false),
(DATE '2023-04-20', 'ACC018', 'John Smith', 150.00, 'debit', 1000.00, false),
(DATE '2023-04-20', 'ACC024', 'Sarah Clarks', 200.00, 'credit', 700.00, false),
-- add more rows</code></pre> 
<p>1.6. Verify the data is populated correctly, run the <code>SELECT</code> query shown in the following figure.</p> 
<pre><code class="lang-sql">SELECT * FROM &lt;s3_bucket_table_namespace&gt;.&lt;s3_bucket_table_name&gt;</code></pre> 
<h3>Step 2. Create an AWS Glue Data Catalog integration</h3> 
<p>In this section, you set up resource links to enable Bedrock Knowledge Bases to access S3 Tables through Redshift. S3 Tables maintains its own catalog (s3tablescatalog), separate from the standard Glue Data Catalog (AwsDataCatalog). Resource links federate these catalogs, presenting S3 Tables as traditional Glue databases and tables. This lets services like Bedrock Knowledge Bases and Athena query them using standard database.table syntax, with Lake Formation managing the federation.</p> 
<p>2.1. Create your <a href="https://docs.aws.amazon.com/glue/latest/dg/define-database.html" rel="noopener" target="_blank">database setup with AWS Glue</a>. The following figure shows how to create your database setup with Glue.</p> 
<p><img alt="Create a database in AWS Glue Data Catalog" class="aligncenter size-full wp-image-28956" height="522" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/CreateDB-GlueCatalog.png" width="1430" /></p> 
<p>2.2. Set up the resource link in Lake Formation. You create the resource link to the database by navigating to Lake Formation (you don’t want to do within Glue), choosing Data Catalog and Tables from the menu. Then, select the catalog from the table that corresponds to the S3 table in the table bucket, as shown in the following figure.</p> 
<p><img alt="Create a resource link in AWS Lake Formation" class="aligncenter size-full wp-image-28954" height="254" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/Action-SelectResourceLink.png" width="936" /></p> 
<p>2.3. Choose Actions and create a resource link at the table level for fine-grained access (you can also create it at the database level). Give the resource link a name, set the destination catalog to your default account catalog, and choose the Glue database from the previous step. The shared table and database auto-populate from the selected table, as shown in the following figure.</p> 
<p><img alt="Configure a resource link in AWS Lake Formation" class="aligncenter size-full wp-image-28964" height="774" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/CreateResourceLink.png" width="748" /></p> 
<h3>Step 3. Set up Amazon Redshift Serverless</h3> 
<p>Bedrock Knowledge Bases supports both Redshift provisioned clusters and Redshift Serverless workgroups as query engines. This walkthrough uses Redshift Serverless; the same steps apply to a provisioned cluster when selected in step 4.2.</p> 
<p>3.1. Create your Redshift Serverless infrastructure by setting up a namespace, workgroup, and IAM role. You can reference the following post for more information: <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-serverless.html" rel="noopener" target="_blank">Getting started with Amazon Redshift Serverless</a>.</p> 
<h3>Step 4. Set up Amazon Bedrock Knowledge Bases</h3> 
<p>Bedrock Knowledge Bases acts as the intelligent layer that translates natural language questions into structured queries, connecting to Redshift Serverless to return accurate, contextual answers from S3 Tables.</p> 
<p>4.1. Create Bedrock Knowledge Bases by navigating to Bedrock and selecting Knowledge Bases under Build. Configure a structured knowledge base, choose Redshift as the query engine, and create a new IAM service role.</p> 
<p>4.2. Connect your Redshift Serverless workgroup by selecting it as the query engine and using the Glue Data Catalog for storage. Specify database and table names in the format <code>database.table</code>, or use multiple entries or wildcards to include more tables, as shown in the following figure.</p> 
<p>As a best practice, you should give the table and columns descriptions while defining the knowledge base to improve the accuracy.</p> 
<p><img alt="Configure Amazon Redshift Serverless as the query engine and connect it to AWS Glue Data Catalog for querying S3 Tables" class="aligncenter size-full wp-image-28957" height="596" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/ConfigureBedrockKB.png" width="936" /></p> 
<p>4.3. Set up a customer managed IAM policy. To adhere to fine grained permissions, we create a policy that only gives access to our Glue resources (database and resource link) and the S3 table bucket resources (bucket, namespace, and table). Create a <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html" rel="noopener" target="_blank">new customer managed IAM policy</a> in your account and copy and paste the following JSON, replacing the placeholder variables with your actual environment values. Once created you’ll attach this policy to the Bedrock service role that is attached to your knowledge base.</p> 
<pre><code class="lang-json">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BedrockGlueAccess",
            "Effect": "Allow",
            "Action": [
                "glue:GetDatabases",
                "glue:GetDatabase",
                "glue:GetTables",
                "glue:GetTable",
                "glue:GetPartitions",
                "glue:GetPartition",
                "glue:SearchTables"
            ],
            "Resource": [
               "arn:aws:glue:&lt;your_region&gt;:&lt;your_accountID&gt;:catalog",
               "arn:aws:glue:&lt;aws_region&gt;:&lt;accountID&gt;:database/&lt;gluedatabase_name&gt;",
"arn:aws:glue:&lt;aws_region&gt;:&lt;accountID&gt;:table/&lt;gluedatabase_name&gt;/&lt;resource_link_name&gt;",
 "arn:aws:glue:&lt;aws_region&gt;:&lt;accountID&gt;:catalog/s3tablescatalog",
             "arn:aws:glue:&lt;aws_region&gt;:&lt;accountID&gt;:catalog/s3tablescatalog/&lt;s3_table_bucket_name&gt;",
"arn:aws:glue:&lt;aws_region&gt;:&lt;accountID&gt;:database/s3tablescatalog/&lt;s3_table_bucket_name&gt;/&lt;s3_table_namespace&gt;",
"arn:aws:glue:&lt;your_region&gt;:&lt;accountID&gt;:table/s3tablescatalog/&lt;s3_table_bucket_name&gt;/&lt;s3_table_namespace&gt;/&lt;s3_table_name&gt;"
            ]
        },
        {
            "Sid": "S3TablesAccess",
            "Effect": "Allow",
            "Action": [
                "s3tables:GetTable",
                "s3tables:GetTableMetadataLocation",
                "s3tables:GetTablePolicy",
                "s3tables:ListTables",
                "s3tables:ListTableBuckets",
                "s3tables:ListNamespaces",
                "s3tables:GetNamespace",
                "s3tables:GetTableBucket"
            ],
            "Resource": [
                "arn:aws:s3tables:&lt;your_region&gt;:&lt;accountID&gt;:bucket/&lt;s3_table_bucket_name&gt;",
                "arn:aws:s3tables:&lt;your_region&gt;:&lt;accountID&gt;:bucket/&lt;s3_table_bucket_name&gt;/*"
            ]
        },
        {
            "Sid": "LakeFormationDataAccess",
            "Effect": "Allow",
            "Action": [
                "lakeformation:GetDataAccess"
            ],
            "Resource": "*"
        }
    ]
}</code></pre> 
<p>4.4. Set up the Lake Formation permission. Grant the Bedrock Knowledge Bases service role access to the S3 table bucket and Glue database to enable data querying, as shown in the following figure. IAM permissions were configured in the previous step.</p> 
<p><img alt="Grant permissions in AWS Lake Formation for IAM principals to access Glue Data Catalog and S3 Tables" class="aligncenter size-full wp-image-28963" height="982" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/AddPermissions-LakeFormation.png" width="558" /></p> 
<p>Both grants should provide select and describe permissions, as shown in the following figure.</p> 
<p><img alt="Configure SELECT and DESCRIBE table permissions in AWS Lake Formation" class="aligncenter size-full wp-image-28962" height="501" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/TablePermissions-lakeFormation.png" width="1289" /></p> 
<p>For more details on granting permission to Data Catalog resources, go to the <a href="https://docs.aws.amazon.com/lake-formation/latest/dg/granting-catalog-permissions.html" rel="noopener" target="_blank">Lake Formation Developer Guide</a>.</p> 
<h3>Step 5. Set up Amazon Redshift Access</h3> 
<p>This section establishes secure authentication between Bedrock Knowledge Bases and Redshift. You configure IAM authentication to enable password-less access and grant the necessary catalog permissions for the knowledge base to execute queries.</p> 
<p>5.1. Create the Bedrock service role as a database user. This enables the Bedrock Knowledge Bases role to connect to Redshift using IAM authentication without a password. In the Redshift query editor, run the following query, replacing <code>&lt;BedrockRoleName&gt;</code> with your Bedrock service role name:</p> 
<pre><code class="lang-sql">CREATE USER "IAMR:&lt;BedrockRoleName&gt;" WITH PASSWORD DISABLE;</code></pre> 
<p>5.2. Grant catalog access in Redshift. This grants the Bedrock execution role permission to use the catalog containing the S3 Tables schema metadata. Replace &lt;BedrockRoleName&gt; with your Bedrock service role name:</p> 
<pre><code class="lang-sql">GRANT USAGE ON DATABASE awsdatacatalog TO "IAMR:&lt;BedrockRoleName&gt;";</code></pre> 
<p>Go to the <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-prereq-structured.html#knowledge-base-prereq-structured-db-access" rel="noopener" target="_blank">Bedrock User Guide</a> for more details on knowledge base service role access.</p> 
<h3>Step 6. Validate/test knowledge base</h3> 
<p>With infrastructure and permissions in place, sync the knowledge base to ensure it has the latest schema information, then test with natural language queries to confirm SQL generation and data access.</p> 
<p>Then, you sync the knowledge base. When the sync is successful, you can proceed.</p> 
<p><img alt="Test the knowledge base in Amazon Bedrock using the Test Knowledge Base button" class="aligncenter size-full wp-image-28960" height="550" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/TestKB.png" width="1430" /></p> 
<p>At this stage you can test the knowledge base. Select a model such as <a href="https://aws.amazon.com/nova/" rel="noopener" target="_blank">Amazon Nova Pro</a>, and in the preview pane start asking question to gain insights into your data. Try these sample queries:</p> 
<ul> 
 <li>“Show me the last 5 transactions for client 12345”</li> 
 <li>“What is the total amount spent by client 12345?”</li> 
 <li>“List all dining transactions for client 12345”</li> 
</ul> 
<p><img alt="Example natural language query results using Amazon Bedrock knowledge base" class="aligncenter size-full wp-image-28959" height="384" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/KB-Results.png" width="936" /></p> 
<h2>Extending this pattern to AWS managed S3 table buckets</h2> 
<p>Although the architecture described in this post uses customer-created S3 table buckets, the same pattern extends naturally to <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-aws-managed-buckets.html" rel="noopener" target="_blank">AWS managed S3 table buckets</a>, as shown in the following figure. Each AWS account has one AWS managed table bucket per <a href="https://aws.amazon.com/about-aws/global-infrastructure/regions_az/" rel="noopener" target="_blank">AWS Region</a>, named <code>aws-s3</code>, which serves as a centralized location for all managed tables automatically created by AWS services in that AWS Region. <a href="https://aws.amazon.com/s3/features/metadata/" rel="noopener" target="_blank">Amazon S3 Metadata</a> is an example: when you enable it on a general purpose S3 bucket, AWS automatically provisions Apache Iceberg-compatible tables under a dedicated namespace within this <code>aws-s3</code> bucket, including a journal table that captures near real-time object change events and an optional live inventory table that provides a queryable snapshot of all objects and their current state.</p> 
<p>These are Apache Iceberg-compatible tables; thus you can follow the same steps outlined in this post: registering the table bucket with Glue Data Catalog, creating resource links in Lake Formation, configuring Redshift, and connecting a Bedrock Knowledge Base. When its configured, business users can ask questions such as “Which objects were deleted from the production bucket in the last 7 days?” or “Show me all objects larger than 1 GB uploaded this month” without writing SQL.</p> 
<p>For AWS managed table buckets, use <code>aws-s3</code> as the bucket name in your S3 Tables and Glue catalog Amazon Resource Names (ARNs) (for example, <code>s3tablescatalog/aws-s3</code>), with the namespace following the format <code>b_&lt;your-general-purpose-bucket-name&gt;</code> and table names fixed as inventory or journal. The Redshift <code>GRANT USAGE ON DATABASE awsdatacatalog</code> step remains unchanged, because catalog federation is handled at the Glue and Lake Formation layer.</p> 
<p>The key distinction is that tables in the <code>aws-S3</code> bucket are read-only and managed exclusively by AWS, which aligns perfectly with the RAG query pattern used throughout this architecture, ensuring accuracy and removing operational overhead.</p> 
<p><img alt="Amazon Bedrock test interface displaying a natural language query and returned results showing S3 object data using the Nova Pro model" class="aligncenter wp-image-28961 size-full" height="436" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/28/S3ManagedTables-KB.png" width="936" /></p> 
<h2>Cleaning up</h2> 
<p>To avoid incurring future charges, delete the Redshift Serverless instance, Bedrock Knowledge Base, Glue Data Catalog, and S3 Tables.</p> 
<h2>Conclusion</h2> 
<p>You’ve built an architecture that transforms structured data access. By combining S3 Tables with Amazon Bedrock Knowledge Bases, you’ve enabled business users across your organization to query data conversationally without SQL expertise. While this walkthrough used financial services as an example, the same pattern applies to retail inventory analysis, healthcare patient monitoring, manufacturing quality metrics, or any domain managing structured data at scale. This pattern also extends to AWS managed S3 table buckets. To apply this pattern to AWS managed S3 table buckets, follow the steps in the <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html" rel="noopener" target="_blank">S3 tables documentation</a>.</p>
