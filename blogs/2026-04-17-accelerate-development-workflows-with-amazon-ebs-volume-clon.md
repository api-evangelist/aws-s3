---
title: "Accelerate development workflows with Amazon EBS Volume Clones"
url: "https://aws.amazon.com/blogs/storage/accelerate-development-workflows-with-amazon-ebs-volume-clones/"
date: "Fri, 17 Apr 2026 19:25:24 +0000"
author: "Shibin Michaelraj"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
<p>Providing a copy of production data to developers, testers, or disaster recovery (DR) teams quickly is a common operational challenge our customers face. Whether it’s a daily database refresh for a support environment, a one-time troubleshooting session that requires real-world data, or a disaster recovery drill that needs to run against a current replica, the underlying requirement is the same: create a usable copy of an existing volume and do it fast. Until now, the standard approach has been to create an <a href="http://aws.amazon.com/ebs">Amazon Elastic Block Store</a> (Amazon EBS) snapshot of the source volume and then restore it into a new volume. This workflow is reliable, but it introduces latency, operational overhead, and wait times that can stretch into hours for large volumes. For teams that need copies frequently or on short notice, this friction adds up. It delays development cycles, limits the frequency of DR testing, and often results in teams working with stale data or skipping validation steps entirely because the cost of getting a fresh copy is too high.</p> 
<p><a href="https://docs.aws.amazon.com/ebs/latest/userguide/ebs-copying-volume.html">Amazon EBS Volume Clones</a> address this by making it possible to create an instant copy of an EBS volume. The resulting clone is a point-in-time copy that is immediately available and can be attached to an instance and used the moment it is created.</p> 
<p>In this post, we discuss how EBS Volume Clones can transform your workflows.</p> 
<h2>Overview of EBS Volume Clones</h2> 
<p>EBS Volume Clones provide a fast way to create instant point-in-time copies of your EBS volumes within the same Availability Zone. When you create a Volume Clone, the new volume is available instantly, so you can attach it to an instance and start using it. Volume Clones are crash-consistent, capturing data blocks written to the source volume the moment the clone operation begins. You don’t need to wait for data transfer to complete before accessing your cloned volume; you get immediate access with single-digit millisecond latency.</p> 
<p>Although the cloned volume is immediately usable, EBS copies data blocks from the source volume in the background through an initialization process. During this phase, the clone delivers baseline performance that is sufficient for most workloads. When initialization is complete, the cloned volume delivers its full provisioned performance, whether that’s the IOPS and throughput of gp3, io2, or other volume types. Importantly, the clone operation doesn’t affect the performance of your source volume. You can continue using it normally throughout the process.</p> 
<h2>Refreshing development environments with production data</h2> 
<p>Teams running large production databases such as Oracle, SQL Server, PostgreSQL, MySQL, or InterSystems IRIS often need to regularly refresh copies of that data into development, quality assurance (QA), and support environments. Before Volume Clones, this meant snapshot-and-restore cycles that could take hours for multi-terabyte databases—during which time teams were either waiting or working with stale data.</p> 
<p>For example, a healthcare organization serving over 1 million members uses Volume Clones to refresh their Electronic Health Record (EHR) support environments daily. They manage approximately 118 TiB of data across multiple volumes, and their previous snapshot-based workflow took approximately 12 hours. After they built automation using Volume Clones, that daily process now completes in around 54 minutes, a 93% reduction with no impact to their production systems, freeing their technical teams to focus on improving healthcare delivery rather than managing infrastructure.</p> 
<p>The following diagram compares the organization’s traditional workflow with a clone-based workflow.</p> 
<p><img alt="" class="alignnone wp-image-28810 size-large" height="432" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/16/BlogPicture1-1-1024x432.png" width="1024" /></p> 
<h2>Beyond dev/test: How customers are using Volume Clones</h2> 
<p>Refreshing development environments is the most common starting point for Volume Clones, but the feature applies wherever you need a fast copy of an existing volume. In this section, we discuss additional workflows where customers are using Volume Clones.</p> 
<h3>Disaster recovery testing</h3> 
<p>Most organizations know they should test their DR procedures more frequently than they do. The barrier is operational: standing up a test environment against real data, running the test, and then rolling back to resume normal operations is complex enough that teams often test only when compliance requires it.</p> 
<p>EPS Strategies, a company that delivers mainframe performance optimization through their Pivotor reporting software, previously had to interrupt their ongoing replication process every time they wanted to run a DR drill. With Volume Clones, they now create a clone of their secondary volume and test against it—without disrupting their replication or recovery posture.</p> 
<p>“Volume Clones reduce our operational overhead significantly,” says the CTO of EPS Strategies. “Previously, our DR testing required stopping replication, moving volumes, testing, and complex rollback procedures. Now we simply create a clone of our secondary volume and test without disrupting our ongoing DR readiness. The feature also provides an additional safety net during actual DR scenarios by maintaining an original copy from the start of DR activation, avoiding time-consuming recoveries from S3 backups if needed.”</p> 
<p>The following diagram compares the company’s traditional workflow with a clone-based workflow.</p> 
<p><img alt="Diagram of clone workflow" class="alignnone wp-image-28809 size-large" height="430" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/16/BlogPicture2-1-1024x430.png" width="1024" /></p> 
<h3>Blue/green deployments</h3> 
<p>In a blue/green deployment, the speed at which you can stand up the green environment directly determines how quickly and safely you can release. For stateful components such as databases, caches, and persistent application volumes, creating the green environment has traditionally required snapshot-based volume duplication with associated wait times and warm-up latency. Volume Clones help teams create instant, data-consistent copies of production volumes to seed the green environment, validate the deployment, and cut over with confidence. For organizations operating large fleets where a single deployment may involve cloning dozens or hundreds of volumes in parallel, this translates directly into shorter release cycles and reduced deployment risk.</p> 
<p>The following diagram compares the traditional workflow using blue/green deployment with a clone-based workflow.</p> 
<p><img alt="Diagram of clone workflow" class="alignnone wp-image-28807 size-large" height="431" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/16/BlogPicture4-1-1024x431.png" width="1024" /></p> 
<h3>CI/CD pipelines</h3> 
<p>Customers are using Volume Clones to seed continuous integration and delivery (CI/CD) build environments with prepopulated dependencies, package caches, or test datasets. The process involves maintaining a base volume with a regularly updated set of build artifacts, cloning it at the start of each pipeline run, building against the clone, and deleting it when the job is complete.</p> 
<p>A customer who builds mobile games adopted this approach after iterating through several alternatives, including remote repository downloads, Docker layer caching, and EBS snapshots. With Volume Clones, they reduced their end-to-end build time from over 12 minutes to under 4, a 60% reduction.</p> 
<p>The following diagram compares the company’s traditional workflow with a clone-based workflow.</p> 
<p><img alt="Diagram of clone workflow" class="alignnone wp-image-28808 size-large" height="431" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/16/BlogPicture3-1-1024x431.png" width="1024" /></p> 
<h3>Operational maintenance and patching</h3> 
<p>Customers are using Volume Clones as a safety mechanism for routine maintenance. One pattern involves cloning root volumes before applying system patches, validating the patch against the clone, and proceeding to production only after confirmation. If an issue occurs, the original volume is untouched. When automated through AWS Lambda, this turns monthly patching from a critical maintenance window into a routine operation with a built-in rollback path.</p> 
<h2>Getting started with EBS Volume Clones</h2> 
<p>Creating Volume Clones is straightforward and can be accomplished through the <a href="https://console.aws.amazon.com/">AWS Management Console</a>, the <a href="http://aws.amazon.com/cli">AWS Command Line Interface</a> (AWS CLI) <a href="https://docs.aws.amazon.com/cli/latest/reference/ec2/copy-volumes.html">copy-volumes command</a>, <a href="https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CopyVolumes.html">AWS SDKs</a>, or infrastructure as code (IaC) tools like <a href="http://aws.amazon.com/cloudformation">AWS CloudFormation</a>.</p> 
<h3>Using the AWS Management Console</h3> 
<p>On the <a href="http://aws.amazon.com/ec2">Amazon Elastic Compute Cloud</a> (Amazon EC2) console, complete the following steps:</p> 
<p>1. On the Amazon EC2 console, choose Volumes under Elastic Block Store in the navigation pane.</p> 
<p>2. Select your source volume, and on the Actions dropdown menu, choose Copy volume.</p> 
<p>3. Configure your desired volume parameters (type, size, IOPS, throughput), and choose Copy volume.</p> 
<p>The cloned volume becomes available within seconds.</p> 
<h3>Using the AWS CLI</h3> 
<p>For programmatic access or automation, use the copy-volumes command:</p> 
<p><code>aws ec2 copy-volumes \</code></p> 
<p><code>--source-volume-id vol-01234567890abcdef \</code></p> 
<p><code>--volume-type gp3 \</code></p> 
<p><code>--size 100 \</code></p> 
<p><code>--iops 3000 \</code></p> 
<p><code>--throughput 250</code></p> 
<p>This example creates a clone of volume vol-01234567890abcdef with gp3 volume type, 100 GiB size, 3,000 IOPS, and 250 MiBps throughput. The cloned volume becomes immediately available for attachment to EC2 instances in the same Availability Zone.</p> 
<h3>Integration with Amazon EKS</h3> 
<p>Volume Clones integrate with the <a href="https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html#managing-ebs-csi">Amazon EBS Container Storage Interface (CSI) driver</a>, so you can clone persistent volumes for containerized applications running on <a href="https://aws.amazon.com/eks/">Amazon Elastic Kubernetes Service</a> (Amazon EKS).</p> 
<h2>Conclusion</h2> 
<p>Amazon EBS Volume Clones change how organizations approach data copy workflows. By reducing wait times and delivering immediate access to production-identical data, teams can move faster—refreshing development databases, accelerating CI/CD pipelines, validating disaster recovery procedures, or troubleshooting production issues without disruption. We encourage you to explore how EBS Volume Clones can transform your workflows. Start with your most time-consuming data refresh process and experience the difference.</p> 
<p>For detailed documentation and advanced configuration options, refer to <a href="https://docs.aws.amazon.com/ebs/latest/userguide/ebs-copying-volume.html">Copy an Amazon EBS volume</a>.</p> 
<h3>FAQs</h3> 
<p><strong>1. Can I use a Volume Clone immediately after it is created? </strong></p> 
<p>Yes, a Volume Clone is instantly available and can be attached to an instance right away.</p> 
<p><strong>2. What performance can I expect from a clone while it is still initializing? </strong></p> 
<p>During initialization, a clone delivers baseline performance equal to the lowest of 3,000 IOPS / 125 MiB/s, the source volume’s provisioned performance, or the clone’s own provisioned performance; it can exceed that baseline if the source volume has unutilized capacity.</p> 
<p><strong>3. Can I use EBS Volume Clones with containerized workloads on Amazon EKS? </strong></p> 
<p>Yes, EBS Volume Clones integrate with the Amazon EBS Container Storage Interface (CSI) driver, allowing you to clone persistent volumes for applications running on Amazon Elastic Kubernetes Service (Amazon EKS).</p>
