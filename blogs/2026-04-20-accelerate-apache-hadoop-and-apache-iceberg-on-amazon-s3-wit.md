---
title: "Accelerate Apache Hadoop and Apache Iceberg on Amazon S3 with the Analytics Accelerator Library"
url: "https://aws.amazon.com/blogs/storage/accelerate-apache-hadoop-and-apache-iceberg-on-amazon-s3-with-the-analytics-accelerator-library/"
date: "Mon, 20 Apr 2026 15:26:41 +0000"
author: "Ran Pergamin"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
<p>Organizations processing large-scale data for analytics, machine learning (ML), and business intelligence face a persistent challenge: how to access and read massive datasets quickly and cost-effectively. As data volumes grow exponentially, the performance of data access patterns becomes more critical. Inefficient read operations can lead to longer processing times, higher compute costs, and delayed insights, all of which ultimately impact an organization’s ability to make timely, data-driven decisions. While <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html" rel="noopener" target="_blank">best practices for optimizing Amazon S3 data access</a>&nbsp;exist, implementing them consistently across diverse workloads can be challenging.</p> 
<p><a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> has become the foundation for more than 1 million data lakes, providing the durability, availability, scalability, and security that modern analytics workloads require. Through years of collaboration with customers running analytics pipelines on S3, <a href="https://aws.amazon.com/" rel="noopener" target="_blank">Amazon Web Services (AWS)</a> has gained deep insights into more optimal access patterns across a variety of use cases. This knowledge has been codified into the <a href="https://github.com/awslabs/analytics-accelerator-s3" rel="noopener" target="_blank">Analytics Accelerator Library for S3 (AAL)</a>, an open source GitHub library that implements S3 client-side best practices and intelligent prefetching strategies to maximize performance for data-intensive analytics applications by enhancing existing connectors.</p> 
<p>In this post, we introduce the AAL and demonstrate how it accelerates data access for analytics workloads. AAL is now included by default in <a href="https://hadoop.apache.org/" rel="noopener" target="_blank">Apache Hadoop</a> 3.4.3 and available in <a href="https://iceberg.apache.org/" rel="noopener" target="_blank">Apache Iceberg</a> 1.9.0 and later, with special optimizations for <a href="https://parquet.apache.org/" rel="noopener" target="_blank">Apache Parquet</a> files. You can use this library to achieve better throughput, lower latency, and reduced compute costs. This can be achieved without requiring changes to your application code, and while helping you process data faster and deliver insights more quickly.</p> 
<p><strong>Note:</strong> In our <a href="https://www.tpc.org/information/benchmarks5.asp" rel="noopener" target="_blank">Transaction Processing Performance Council Decision Support (TPC-DS)</a> derived benchmark testing, Analytics Accelerator Library for S3 improved query execution time by up to 1.2X for IO-intensive data lake workloads on <a href="https://spark.apache.org/" rel="noopener" target="_blank">Apache Spark</a>.</p> 
<h2>Why we built the AAL</h2> 
<p>We observed a common challenge while working with customers running large-scale analytics workloads on S3. Our analysis of workloads revealed opportunities to improve how applications interact with S3. For example, we found that many analytics jobs weren’t achieving optimal performance due to their access patterns. Applications frequently issued many small, random read requests when sequential reads would be more efficient or didn’t effectively parallelize their S3 requests.</p> 
<p>Another finding revealed that Apache Parquet was the most widely used file format at the time of our analysis. We identified several inefficiencies in how existing connectors read Parquet files, particularly in their handling of Parquet file metadata and column access patterns. This led us to develop specific optimizations for Parquet files in AAL, focusing on where we could make the most impact to improve performance.</p> 
<p>These suboptimal patterns resulted in higher latency, lower throughput, and consequently longer job completion times and increased compute costs. Instead of creating a new connector, we chose to invest in enhancing the connectors already embedded in popular open source frameworks. This decision aligns with our core principle of enabling you to focus on deriving value from your data rather than managing infrastructure or optimizing performance.</p> 
<p>The AAL for S3 is an addition to existing S3 connectors, including <a href="https://hadoop.apache.org/docs/r3.4.1/hadoop-aws/tools/hadoop-aws/connecting.html" rel="noopener" target="_blank">S3A</a> and <a href="https://iceberg.apache.org/javadoc/1.1.0/org/apache/iceberg/aws/s3/S3FileIO.html" rel="noopener" target="_blank">S3FileIO</a>, integrating with them seamlessly through their existing API. Prior to developing the AAL, we analyzed the architecture of existing data lake connectors (such as S3A/Hadoop, and S3FileIO/Iceberg), and many standalone libraries to identify similar components and opportunities for code reuse. We integrated performance improvements directly into these frameworks so that you get these benefits automatically through their existing stack.</p> 
<p>This approach means you don’t need to modify their applications, learn new APIs, or change their workflows. We are actively contributing these improvements to the open source community, working closely with project maintainers and contributors to make sure the optimizations benefit the broader analytics community.</p> 
<h2>Key optimizations that accelerate your analytics workloads</h2> 
<p>We group these optimizations into two categories: Format-agnostic improvements, and Parquet format specific improvements.</p> 
<h3>1. Format-agnostic improvements</h3> 
<p>AAL provides multiple file format-agnostic improvements that impact all S3 access regardless of the logical organization of bytes in S3. The following are some of the major ones:</p> 
<p><strong>Request-reshaping:</strong> AAL optimizes range requests by avoiding open-ended requests, which improves efficiency when fetching specific portions of an object. It performs automatic request parallelization and provides support for timeouts and retries.</p> 
<p><strong>Sequential prefetching:</strong> AAL intelligently detects sequential read patterns and proactively prefetches data to reduce latency.</p> 
<p><strong>Small object prefetching:</strong> Objects smaller than a configurable size are automatically prefetched entirely, eliminating the overhead of multiple small requests.</p> 
<p><strong>Object metadata caching:</strong> Although data lake analytics engines can optimize queries to avoid multiple scans of the same data, they sometimes still need to query object metadata multiple times. For example, some engines make up to four S3 HEAD API requests to the same object for each Parquet read, which affects cost and stalling I/O. The AAL caches object metadata as allowed by consistency semantics (with configurable Time-To-Live), which reduces the number of S3 HEAD requests.</p> 
<p><strong>Memory management:</strong> We have seen some client libraries that may grow memory usage unbounded. The AAL implements a memory management system that prevents excessive memory consumption while maintaining high performance.</p> 
<h3>2. Apache Parquet format specific improvements</h3> 
<p>The second category is specific improvements for data in the Parquet file format. This Parquet format merits its own optimizations because, while its use is ubiquitous, connectors don’t always implement best practices for this format. In this category, we deliver optimizations that include the following:</p> 
<p><strong>Footer prefetching and caching:</strong> Parquet files store critical metadata in the footer of the object. AAL automatically reads and caches this metadata when a stream is opened, reducing first byte latency to parse the data and preventing multiple small GET requests later in the processing flow.</p> 
<p><strong>Vector reads:</strong> Open source Parquet readers are moving toward vectorized readers to accelerate data processing. Vectorized reads can read different parts of the same object in parallel. This approach is better aligned with Parquet’s columnar structure as compared to sequential reads, because it enables the reader to load multiple columns of the same row group or the same column across multiple row groups simultaneously. The AAL enables vectored reads, which allows AAL to group related requests together, thus reducing the total number of API calls to S3.</p> 
<p><strong>Predictive column prefetching:</strong> When vectored reads aren’t available, AAL tracks which columns are frequently accessed and proactively prefetches those columns when similar Parquet files are opened. For example, if your application reads columns <code>customer_id</code> and <code>purchase_amount</code> from one file, then AAL automatically prefetches those same columns when you open another file in the same dataset.</p> 
<h2>Getting started with the AAL</h2> 
<p>The AAL for S3 has been tested and integrated with the Apache Hadoop S3A client and with the Iceberg S3FileIO client. It also works seamlessly with datasets stored in <a href="https://aws.amazon.com/s3/features/tables/" rel="noopener" target="_blank">S3 Tables</a>.</p> 
<p>Go to the <a href="https://github.com/awslabs/analytics-accelerator-s3?tab=readme-ov-file#getting-started" rel="noopener" target="_blank">Getting Started page</a> on GitHub for installation instructions, configuration options, and integration examples.</p> 
<h2>Real-world performance impact</h2> 
<p>In our testing with customer workloads, the AAL has shown significant performance improvements across a variety of scenarios. We even got the following testimonial from <a href="https://voodoo.io/" rel="noopener" target="_blank">Voodoo.io</a>:</p> 
<blockquote> 
 <p style="text-align: left;">“<em>The AAL for S3 has been a game changer for our Apache Iceberg data processing pipeline at Voodoo.io. We’ve integrated it across our Bronze to Gold jobs, resulting in a 10% performance boost. The transparent integration into Iceberg release v1.9.0 made adoption effortless. This library has significantly enhanced our data processing efficiency with minimal effort on our part.</em>” – Data Engineering Team, Voodoo.io</p> 
</blockquote> 
<p>The improvements are particularly effective for workloads that:</p> 
<ul> 
 <li>Process many small Parquet files</li> 
 <li>Repeatedly access the same columns across multiple files</li> 
 <li>Read data sequentially through large objects</li> 
 <li>Need to scan a subset of columns from wide tables</li> 
</ul> 
<h2>Benchmarking on a TPC-DS derived test using Apache Spark</h2> 
<p>TPC-DS simulates a retail business environment with complex analytical queries across multiple dimensions including sales, returns, inventory, and customer behavior, using a star schema with fact and dimension tables.</p> 
<p>TPC-DS is widely used by organizations to compare database and analytic platforms, test query optimization capabilities, and validate system performance at various data scales from 1 GB to 100 TB. The results presented here aren’t directly comparable to the official TPC-DS results due to setup differences.</p> 
<h3>Testing methodology:</h3> 
<p>For the benchmark we used the following:</p> 
<ul> 
 <li>Spark 3.5.4</li> 
 <li>Hadoop 3.5</li> 
 <li>1+8 r6id.4xlarge instance cluster as the data lake engine.</li> 
</ul> 
<p>To streamline the setup, we used spark in standalone mode and used dynamic executor allocation. We configure spark to use the local NVMe for local temporary storage for caching and spills to disk. Although the Apache Hive/Parquet run used a local Hive catalog on the spark server, we used <a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a> as the data catalog for the Iceberg tables.</p> 
<p>We stored the TPC-DS benchmark data in parquet files. We carefully chose the distribution of the data in parquet files to already create a high performant data lake even before we enabled AAL. We stored each of the seven fact tables in large (approximately 100 MiB) Parquet files with a single row group. This ensured that the Spark read requests to S3 are mostly large and overcome any per-request overheads to improve performance.</p> 
<p>In benchmark testing, Analytics Accelerator Library for S3 improved query execution time by up to 1.2X on an IO-intensive 8 query subset derived from data lake workloads on Apache Spark.</p> 
<table cellpadding="3" style="height: 206px;" width="718"> 
 <tbody> 
  <tr> 
   <td style="text-align: center;"><strong>Provider</strong></td> 
   <td style="text-align: center;"><strong>Total time (seconds)</strong></td> 
   <td style="text-align: center;"><strong>Advantage</strong></td> 
   <td style="text-align: center;"><strong>Geomean (seconds)</strong></td> 
  </tr> 
  <tr> 
   <td colspan="4" style="text-align: center;"><strong>Hive/Parquet</strong></td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">Library enabled</td> 
   <td style="text-align: center;">467.24</td> 
   <td style="text-align: center;">1.2x faster with library enabled</td> 
   <td style="text-align: center;">54.97</td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">S3A</td> 
   <td style="text-align: center;">554.69</td> 
   <td style="text-align: center;">Baseline</td> 
   <td style="text-align: center;">65.21</td> 
  </tr> 
  <tr> 
   <td colspan="4" style="text-align: center;"><strong>Iceberg/Parquet</strong></td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">Library enabled</td> 
   <td style="text-align: center;">372.40</td> 
   <td style="text-align: center;">1.2x faster with library enabled</td> 
   <td style="text-align: center;">44.18</td> 
  </tr> 
  <tr> 
   <td style="text-align: center;">S3FileIO</td> 
   <td style="text-align: center;">442.28</td> 
   <td style="text-align: center;">Baseline</td> 
   <td style="text-align: center;">52.49</td> 
  </tr> 
 </tbody> 
</table> 
<p>As previously mentioned, this library makes several improvements to the shape of the requests issued to avoid open-ended requests. This reduces the total data transferred by the connectors while simultaneously improving performance. In turn, we measured the total data accessed on the bucket. On Hive/Parquet, enabling AAL reduces the total data accessed from 17.4 TB to 9.7 TB—a 44% reduction. On Iceberg/Parquet, enabling AAL reduces the total data accessed from 13.2 TB to 8.4 TB, representing a 36% reduction. Furthermore, AAL also prefetches and caches parquet footer data, which means AAL doesn’t issue any small 16 byte requests, while 20% of all requests to S3 are less than16 bytes without AAL.</p> 
<h2>Conclusion</h2> 
<p>The Analytics Accelerator Library for Amazon S3 (AAL) represents our commitment to making S3 the best possible foundation for data analytics workloads. We enhance existing connectors with intelligent optimizations so that you achieve better performance and lower costs without changing their applications or workflows.</p> 
<p>AAL is available today on <a href="https://github.com/awslabs/analytics-accelerator-s3" rel="noopener" target="_blank">GitHub</a>, and we encourage you to try it with your own workloads. We’re excited to see how you use the AAL to accelerate your analytics, and we look forward to your feedback as we continue to evolve AAL.</p> 
<p>Have questions or feedback? Feel free to leave them in the comments section.</p>
