---
title: "Enabling AI-powered analytics on enterprise file data: Configuring S3 Access Points for Amazon FSx for NetApp ONTAP with Active Directory"
url: "https://aws.amazon.com/blogs/storage/enabling-ai-powered-analytics-on-enterprise-file-data-configuring-s3-access-points-for-amazon-fsx-for-netapp-ontap-with-active-directory/"
date: "Fri, 24 Apr 2026 14:22:36 +0000"
author: "Jay Horne"
feed_url: "https://aws.amazon.com/blogs/storage/feed/"
---
<p>In the past, data stored in a file system was inaccessible to analytical tools like <a href="https://aws.amazon.com/quicksuite/" rel="noopener" target="_blank">Amazon Quick Suite</a> and <a href="https://aws.amazon.com/sagemaker" rel="noopener" target="_blank">Amazon SageMaker</a>. Now,&nbsp;Amazon FSx for NetApp ONTAP supports <a href="https://aws.amazon.com/s3/features/access-points/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3) Access Points</a>, so you can access your file data stored on FSx for NetApp ONTAP file systems as if it were in Amazon S3. With this capability, data scientists, analysts, and machine learning (ML) experts can use the Amazon S3 API and a broad set of integrations with generative AI, machine learning, and analytics services against enterprise file data that you store on AWS services.</p> 
<p>With this capability, you can use your enterprise file data to augment generative AI applications with <a href="https://aws.amazon.com/bedrock/knowledge-bases" rel="noopener" target="_blank">Amazon Bedrock Knowledge Bases</a> for Retrieval Augmented Generation (RAG), train ML models with SageMaker, generate insights with Amazon S3 integrated third-party services, use comprehensive research capabilities in AI-powered business intelligence (BI) tools such as Amazon Quick Suite, and run analyses using Amazon S3-based cloud-native applications, all while your file data continues to reside in your FSx for NetApp ONTAP file system.</p> 
<p>Amazon S3 Access Points for FSx for NetApp ONTAP provide a robust set of mechanisms for secure data access through S3. Access control operates at two distinct layers: <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> permissions and file system-level permissions. This dual-layer security model helps ensure that only authorized users and applications can access your data, respecting both cloud-native AWS security controls and traditional file system permissions. In this post, we walk you through how these layers work together and how to configure them for your environment.</p> 
<p>Additionally, we show you how to configure S3 access points for an FSx for NetApp ONTAP file system integrated with Windows Active Directory and how to control access for different users working with AI-powered applications such as Amazon Quick Suite. This example illustrates how organizations can use their existing Active Directory infrastructure while enabling modern cloud-native workflows.</p> 
<h2>Solution overview</h2> 
<p>Let’s look at how Amazon S3 access points can be configured to allow secure data access with Amazon Quick Suite, an AI-powered business intelligence and analytics platform.</p> 
<p>Consider a scenario involving Organization XYZ, a global manufacturing and retail company with finance and support business units, each with years of operational data.</p> 
<p>Organization XYZ has migrated their enterprise data from an on-premises environment to a single FSx for NetApp ONTAP file system with a Storage Virtual Machine (SVM) called SVM1 that hosts data for multiple departments. The file system is integrated with Windows Active Directory for centralized identity and access management.</p> 
<p>The finance department maintains critical financial data spanning more than 10 years of operational history. This data is stored in a dedicated volume named <em>Finance</em> on SVM1, organized into two primary folders:</p> 
<ul> 
 <li><code>/financial_records/</code>: Contains historical financial records, invoice archives, and audit documents.</li> 
 <li><code>/quarterly_reports/</code>:Stores quarterly financial reports and analysis documents.</li> 
</ul> 
<p>The finance department uses Active Directory groups to manage permissions:</p> 
<ul> 
 <li><strong>FinanceAdmin group</strong>: Full read/write permissions on all financial data.</li> 
 <li><strong>FinanceAnalysts group</strong>: Read-only access to /<code>financial_records</code>/ and read/write access to /<code>quarterly_reports</code>/</li> 
</ul> 
<p>In the first use case, Alice, a member of the Finance Analysts group, works on the analytics team and needs to perform deep research on spending patterns using the AI-powered research capabilities of Amazon Quick Suite. She wants to create comprehensive reports and interactive dashboards showing cost optimization opportunities by analyzing years of financial data.</p> 
<p>When configuring an Amazon S3 access point for FSx for NetApp ONTAP, the file system identity you associate with the access point should reflect who or what will be using it. For human-in-the-loop use cases—such as giving an individual user S3-based access to the files they already have permissions to through Active Directory—a user account identity is the natural fit, ensuring that the same AD-enforced permissions governing their file access are consistently applied through the S3 access point. Conversely, for automated workflows, scheduled jobs, or S3 applications that need to read or write data programmatically, a service account identity is more appropriate, providing a stable, non-interactive credential with scoped permissions tailored to the application’s specific needs. The right choice ultimately depends on your use case and access requirements: align the access point identity with the type of principal—human or system—that will be consuming it.</p> 
<p>The customer support department stores their operational data in a separate volume named <em>Support</em> on the same SVM1. This volume contains valuable customer interaction data organized into three folders:</p> 
<ul> 
 <li><code>/support_tickets/</code>: Historical support ticket records.</li> 
 <li><code>/transcripts/</code>: Customer service call transcripts and chat logs.</li> 
 <li><code>/feedback_docs/</code>: Product feedback and customer satisfaction surveys.</li> 
</ul> 
<p>Access to customer support data is managed through dedicated Active Directory groups:</p> 
<ul> 
 <li><strong>SupportAdmin group</strong>: Full read/write permissions on all support data.</li> 
 <li><strong>SupportAnalysts group</strong>: Read-only access to all folders for analysis purposes.</li> 
</ul> 
<p>In the second use case, Bob, a member of the support team, uses a service account, <em>SupportSvc</em>, to access support data to analyze customer sentiment and identify common product issues. The service account is used here because this is an existing support team workflow that’s also used to run extract, transform, and load (ETL) jobs with other software products in their environment. This demonstrates flexibility in credentialing between standard users and service accounts.</p> 
<p>Lastly, there’s an Admin user with broader permissions across both departments’ data for system management and cross-functional analysis purposes.</p> 
<p>To enable Alice, Bob, and the Admin user to access their respective FSx data through Amazon Quick Suite while maintaining the existing Active Directory security model, you will create four dedicated Amazon S3 access points:</p> 
<ul> 
 <li><strong>Finance S3 access point for Alice</strong>: This access point will be configured to map to Alice’s Active Directory identity, so she can access the Finance volume data through S3 APIs. The access point will respect her FinanceAnalysts group permissions, providing read-only access to financial records and read/write access to quarterly reports.</li> 
 <li><strong>Support S3 access point for Bob</strong>: This access point will map to the support service account, SupportSvc, so he can access the customer Support volume data. The configuration will honor the SupportAnalysts group permissions, granting read-only access to all support folders.</li> 
 <li><strong>Two S3 access points for Admin</strong>: These access points will be configured for the Admin user with broader access across, one for each the Finance and Support volumes, enabling cross-functional analysis and system management tasks.</li> 
</ul> 
<p>Each Amazon S3 access point will enforce security at two layers: IAM policies will control which AWS principals can use the access point, while FSx for NetApp ONTAP will enforce file system-level permissions based on the mapped Active Directory identities. This dual-layer approach helps ensure that users can only access data they’re authorized to see, even when accessing it through Amazon Quick Suite S3-based integrations.</p> 
<h2>Solution walkthrough</h2> 
<p>In the following sections, you will walk through the process of creating and configuring these Amazon S3 access points, setting up the appropriate IAM policies, and demonstrating how Alice and Bob can securely access their data through Amazon Quick Suite.</p> 
<p><em>Figure 1</em> illustrates the solution architecture and workflow.</p> 
<p><img alt="Diagram of a user accessing an S3 Access Point showing authentication through AD and IAM. Users or inputs connect to IAM, which routes traffic to Quick. These interact with Managed Active Directory, and data flows between them through FSx for NetApp ONTAP" class=" wp-image-28869 aligncenter" height="450" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-1-User-authentication-to-the-S3-access-point.png" width="714" /></p> 
<p style="text-align: center;"><em>Figure 1: </em><em>User authentication to the S3 access point</em></p> 
<p><strong>Key components:</strong></p> 
<ol> 
 <li><strong>Amazon FSx for NetApp ONTAP File System</strong>: Single-Availability Zone deployment with SSD storage, hosting multiple volumes containing departmental data.</li> 
 <li><strong>Windows Active Directory</strong>: Self-managed Active Directory providing user authentication and authorization.</li> 
 <li><strong>Storage Virtual Machines (SVMs)</strong>: Isolated file servers within the FSx file system, joined to Active Directory.</li> 
 <li><strong>FSx volumes</strong>: Separate volumes for finance and support data with NTFS security.</li> 
 <li><strong>S3 access points</strong>: Four access points, two per volume, each associated with a specific Active Directory user identity.</li> 
 <li><strong>IAM policies</strong>: Access point policies controlling which AWS principals can use each access point.</li> 
 <li><strong>Amazon Quick Suite knowledge base</strong>: AI-powered application consuming data through S3 access points.</li> 
 <li><strong>Users</strong>: Organization members accessing their authorized data through Quick Suite.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>For this walkthrough, you need the following prerequisites:</p> 
<ul> 
 <li>An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup" rel="noopener" target="_blank">AWS account.</a></li> 
 <li>An FSx for ONTAP filesystem deployed with Active Directory integration. If you don’t have an existing file system, you can deploy one by following the steps in <a href="https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/getting-started.html" rel="noopener" target="_blank">Getting started with Amazon FSx for NetApp ONTAP</a>.</li> 
 <li>A self-managed Active Directory environment with groups (&lt;group names&gt;) and users &lt;user names&gt; created as described previously</li> 
 <li>Client workstation instances</li> 
 <li>Amazon Quick <a href="https://docs.aws.amazon.com/quick/latest/userguide/admin-setting-up.html" rel="noopener" target="_blank">setup</a> with policies, roles, and users</li> 
</ul> 
<h2>Create and configure the access points and permissions</h2> 
<p>In this section we will cover the setup of our environment to show we have created all the prerequisite components including volumes, users, groups, and file shares. Additionally, we will show the permissions as they have been configured for the shares, groups, and users</p> 
<p><strong>Step</strong><strong> 1: Verify that the volumes have been pre-created and are online</strong></p> 
<ul> 
 <li>From the <a href="https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/managing-resources-ontap-apps.html#netapp-ontap-cli" rel="noopener" target="_blank">ONTAP CLI</a>, run the command <code>volume show</code> to verify the volumes are created and online as in <em>Figure 2.</em></li> 
</ul> 
<p><img alt="A screenshot of the ONTAP CLI showing the 'vol show –volume ”With the output verifying that the volumes needed for the exercise are created and online" class="size-full wp-image-28872 aligncenter" height="109" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure2-3.png" width="495" /></p> 
<p style="text-align: center;"><em>Figure 2: Command output showing a l</em><em>ist of volumes</em></p> 
<ul> 
 <li>Or from the AWS Management Console, navigate to<strong> Amazon</strong> <strong>FSx </strong>and choose<strong> Volumes</strong> in the navigation pane. Look for the volumes in the list as in <em>Figure 3</em>.</li> 
</ul> 
<p><img alt="A screenshot of the AWS Console web page showing the Amazon FSx volumes section. In the screenshot you can see that the prerequisite volumes are created and online" class="size-full wp-image-28875 aligncenter" height="232" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-3-List-of-volumes-in-the-Amazon-FSx-console.png" width="403" /></p> 
<p style="text-align: center;"><em>Figure 3: List of volumes in the Amazon FSx console</em></p> 
<p><strong>Step 2: Validate the CIFS shares are created and the permissions are set properly</strong></p> 
<p style="padding-left: 40px;"><em><strong>Note: </strong>This doesn’t apply to S3 access points, because only the file and folder NTFS (ACL) permissions are enforced for S3 access.</em></p> 
<ul> 
 <li>From the ONTAP CLI, execute the command <code>cifs share show</code> to view the details of the share on the file system as in <em>Figure 4</em>.</li> 
</ul> 
<p><img alt="cifs share show” command. Confirm that the share permissions are set so that the groups are correctly aligned to their respective shares.&gt;" class="alignnone size-full wp-image-28876 aligncenter" height="196" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-4-Details-of-the-share-obtained-by-using-the-ONTAP-CLI.png" width="454" /></p> 
<p style="text-align: center;"><em>Figure 4: Details of the share obtained by using the ONTAP CLI</em></p> 
<ul> 
 <li>Or from the Windows client, navigate to&nbsp;<strong>fsmgmt.msc</strong> and find the shares on <strong>SVM1</strong>. (<em>Figure 5</em>).</li> 
</ul> 
<p><img alt="A screenshot of the fmsgmt.msc snap-in connected to the FSx ONTAP file system. The shrares are shown as active with current connections" class=" wp-image-28877 aligncenter" height="136" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-5-Locate-the-shares-on-SVM2.png" width="432" /></p> 
<p style="text-align: center;"><em>Figure 5: Locate the shares on SVM2</em></p> 
<ul> 
 <li>Open the shares to view the properties of each as in <em>Figure 6</em>.</li> 
</ul> 
<p><strong>&nbsp;</strong></p> 
<p><em> <img alt="Screenshot of the Microsoft share level permissions as seen from the fsmgmt.msc console snap-in. The difference in permissions between the full-control and read-only access groups is highlighted.&gt;" class="size-full wp-image-28880 aligncenter" height="280" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-6-Permissions-page-of-the-share-properties.png" width="491" /></em></p> 
<p style="text-align: center;"><em>Figure 6: Permissions page of the share properties</em></p> 
<p><strong>&nbsp;</strong></p> 
<p><strong>Step 3: Verify the file and folder level NTFS (ACL) permissions</strong></p> 
<p>From Windows Explorer, right-click on the mounted file share and select <strong>Properties</strong>. Choose the <strong>Security </strong>tab to view the security settings of the file share. Make note of the difference in security settings between the group that has full-control compared to the read-only access level group as in <em>Figure 7</em>.</p> 
<p><strong> <img alt="A screenshot of the folder level NTFS permissions as seen through Windows Explorer. The difference between the full-control and read-only access level groups is highlighted.&gt;" class="size-full wp-image-28882 aligncenter" height="307" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-7-Security-page-of-the-share-properties.png" width="490" /></strong></p> 
<p style="text-align: center;"><em>Figure 7: Security page of the share properties</em></p> 
<p><strong>Step 4: Verify access by user</strong></p> 
<p>Verify the user account has access to open and view files in the volumes as in <em>Figure 8</em>. Using the credentials of a user from each group, <a href="https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/create-smb-shares.html" rel="noopener" target="_blank">connect to the file shares</a> and verify that their access is correctly applied. Read-only users should only be able to read files, and full-control users should be able to edit and delete files.</p> 
<p><img alt="A screenshot of the Windows Explorer page showing a list of financial record files. A file is opened for editing in a text editor showing that the current user has access to open and edit this file" class="size-full wp-image-28884 aligncenter" height="295" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-8-A-financial-record-file-is-opened-for-editing.png" width="542" /></p> 
<p style="text-align: center;"><em>Figure 8: A financial record file is opened for editing</em></p> 
<p><strong>Step 5: Create the S3 access points</strong></p> 
<p style="padding-left: 40px;">a. In the Amazon FSx console, choose <strong>Volumes </strong>from the navigation pane and choose the <strong>S3</strong> tab of the volume you want to add an access point to. Choose<strong> Create S3 access point </strong>as in <em>Figure 9</em>.</p> 
<p><img alt="A screenshot of the AWS console showing the new S3 Access Point tab on the FSx for ONTAP page. The S3 tab is highlighted and the Cfreate S3 access point button is visible" class="size-full wp-image-28886 aligncenter" height="140" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-9-Create-an-S3-access-point.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 9: Create an S3 access point</em></p> 
<p style="padding-left: 40px;">b. Enter the <strong>Access point name</strong> and <strong>File system user identity</strong>, select your preferred <strong>Network configuration</strong>, and choose <strong>Create S3 access point </strong>as in <em>Figure 10</em>.</p> 
<p><strong> <img alt="A screenshot showing the configuration page of the Create S3 acces point workflow. The properties are filled out according to the instructions in this step" class="size-full wp-image-28888 aligncenter" height="643" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-10-Configure-the-S3-access-point.png" width="459" /></strong></p> 
<p style="text-align: center;"><em>Figure 10: Configure the S3 access point</em></p> 
<p><strong>Step 6: Repeat steps 1 through 5 to create a total of four access points</strong></p> 
<p style="padding-left: 40px;">a. Repeat the preceding steps to create four access points: One for the SupportSvc service account to access the Support volume using the information in the following table. One for the user Alice to access the Finance volume, and one for each volume for the Admin user to access each share as in the following table.</p> 
<p><img alt="4 Access Points" class=" wp-image-28891 aligncenter" height="195" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-11-Acess-Points.png" width="856" /></p> 
<p style="padding-left: 40px;">b. To confirm the access points, go to the Amazon S3 console and choose <strong>Access Points for FSx</strong> in the navigation pane, as in <em>Figure 11</em>.</p> 
<p><img alt="Screenshot showing four access points for FSx on the AWS console." class="size-full wp-image-28892 aligncenter" height="261" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-11-Access-points-for-FSx.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 11: Access points for FSx</em></p> 
<p><strong>&nbsp;</strong></p> 
<p><strong>Step 7: Copy the Access Point alias</strong></p> 
<ul> 
 <li>Copy the <strong>Access Point alias</strong> of full-control user and save to your clipboard as in <em>Figure 12</em>.</li> 
</ul> 
<p style="text-align: center;"><img alt="Screenshot showing the access point alias has been copied using the embedded copy to clipboard button from the FSx for ONTAP S3 access point AWS console web page.&gt;" class="size-full wp-image-28895 aligncenter" height="152" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-12-Copy-the-Access-Point-alias-from-the-console.png" width="364" /></p> 
<p style="text-align: center;"><em>Figure 12: Copy the <strong>Access Point alias </strong>from the console</em></p> 
<p><strong>Step 8: Verify access to the data using the access point: </strong></p> 
<ul> 
 <li>From a CloudShell or a terminal, run the command <code>aws s3 ls s3://&lt;access point alias&gt;/&lt;path to folder&gt;</code></li> 
</ul> 
<p><img alt="Screenshot of a CloudShell console session with the command from the step executed. The output shows the list of expected files has been returned" class="size-full wp-image-28897 aligncenter" height="211" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-13-Use-the-CLI-to-verify-access-to-the-data-through-the-access-point.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 13: Use the CLI to verify access to the data through the access point</em></p> 
<p><strong>Step 9: Ensure the access point has read access to a file in the volume </strong></p> 
<ul> 
 <li>From CloudShell or a terminal, run the command <code>aws s3 cp s3://&lt;access point alias&gt;/&lt;path to folder&gt;/&lt;filename&gt;</code></li> 
</ul> 
<p><img alt="Screenshot of a CloudShell console session with the command from the step executed. The file requested has been downloaded and saved locally" class="size-full wp-image-28899 aligncenter" height="65" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-14-Use-the-CLI-to-the-access-point-has-read-access.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 14: Use the CLI to the access point has read access</em></p> 
<h2>Demonstrate an Amazon Quick Suite use case and how access is allowed or denied</h2> 
<p>With the access points, users, and permissions configured, you’re ready to test the configuration.</p> 
<p><strong>In this section we will configure Amazon Quick to use the S3 access point to populate a knowledge base. Afterward, we’ll query the knowledge base to see what insights can be learned from the data set we’ve discovered. </strong></p> 
<ol> 
 <li>Navigate to the Amazon Quick console in your AWS account as in <em>Figure 15</em>.</li> 
</ol> 
<p><img alt="Screenshot of the Amazon Quick web page console showing the home button highighted" class="size-full wp-image-28901 aligncenter" height="285" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-15-The-Amazon-Quick-web-page-console.png" width="488" /></p> 
<p style="text-align: center;"><em>Figure 15: The Amazon Quick web page console</em></p> 
<ol start="2"> 
 <li>Verify the Region: From the top right corner of the Amazon Quick web page console, click the account name drop down arrow. From the list, select the region name, and check to make sure that the correct region is chosen as in <em>Figure 16</em>.</li> 
</ol> 
<p><img alt="Screenshot showing the Amazon Quick web page console with the user options menu expanded. The region name is highlighted" class="size-full wp-image-28903 aligncenter" height="228" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-16-Select-the-Amazon-Quick-region.png" width="170" /></p> 
<p style="text-align: center;"><em>Figure 16: Select the Amazon Quick region</em></p> 
<p><em><strong>Note</strong>: Amazon Quick Suite role permissions by selecting <strong>Manage Accounts</strong>, then <strong>Permissions</strong>, and selecting<strong> AWS resources</strong>. Verify that the selected role has the appropriate Amazon S3 permissions configured in IAM.</em></p> 
<ol start="3"> 
 <li>Choose <strong>Integrations</strong> from the navigation panel (at the time of this writing, available in Virginia, Oregon, Sydney, and Dublin) as in <em>Figure 17</em>.</li> 
</ol> 
<p><img alt="The Amazon Quick navigation bar highlighting the Integrations button" class="size-full wp-image-28905 aligncenter" height="311" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-17-The-Amazon-Quick-navigation-bar-highlighting-the-Integrations-button.png" width="277" /></p> 
<p style="text-align: center;"><em>Figure 17: The Amazon Quick navigation bar highlighting the Integrations button</em></p> 
<p><em>&nbsp;</em></p> 
<ol start="4"> 
 <li>In <strong>Integrations</strong>, choose <strong>Knowledge bases</strong>, and select <strong>Amazon S3,</strong> as in <em>Figure 18</em>.</li> 
</ol> 
<p><img alt="Set up knowledge bases" class="size-full wp-image-28908 aligncenter" height="234" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-18.png" width="337" /></p> 
<p style="text-align: center;"><em>Figure 18: Integrations, choose Knowledge bases</em></p> 
<ol start="5"> 
 <li>On the <strong>Create integration</strong> page, enter a name for the integration, select the appropriate account. For the <strong>S3 bucket URL</strong>, paste the S3 Access Point alias copied previously. Add <code>S3://</code> as a prefix to the alias as in <em>Figure 19</em>.</li> 
</ol> 
<p><img alt="Create knowledge base" class="size-full wp-image-28910 aligncenter" height="286" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure19.png" width="471" /></p> 
<p style="text-align: center;"><em>Figure 19: Enter integration name</em></p> 
<ol start="6"> 
 <li>Enter a <strong>Name</strong> for the knowledge base. Under <strong>Content</strong>, select <strong>Add all content</strong> and choose <strong>Create</strong>, as in <em>Figure 20</em>.</li> 
</ol> 
<p><img alt="Figure 20 Enter a name for the knowledge base" class="size-full wp-image-28912 aligncenter" height="293" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-20.png" width="464" /></p> 
<p style="text-align: center;"><em>Figure 20: Enter a name for the Knowledge Base</em></p> 
<ol start="7"> 
 <li>Large volumes of data might take some time to fully ingest and synchronize. When complete, the status will change from <strong>Syncing</strong> to <strong>Available</strong>, as shown in <em>Figure 21</em>.</li> 
</ol> 
<p><img alt="Figure 21" class="size-full wp-image-28913 aligncenter" height="121" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-21.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 21: Status is synching/available</em></p> 
<ol start="8"> 
 <li>When the integration is complete, you can choose <strong>Chat agents</strong> from the navigation pane and query the data using natural language, as in <em>Figure 22</em>.</li> 
</ol> 
<p><img alt="Figure 22 Choose chat agents" class="size-full wp-image-28916 aligncenter" height="367" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-22-Choose-chat-agents.png" width="447" /></p> 
<p style="text-align: center;"><em>Figure 22: Choose Chat agents</em></p> 
<h3>Example of a failed integration attempt</h3> 
<p>In this example, Bob attempts to create an access point connected to the Support volume with his user account. Following the steps above, but now using his own credentials instead of the admin user.</p> 
<p>He starts by signing in in to the Amazon console and navigating to the FSx service page, selecting the FSx for ONTAP file system, and clicking the create S3 Access Point button on the S3 tab of the Support volume. He uses his own username to create the Access Point and then copies the S3 Access Point alias to his clipboard (<em>Figure 23</em>).</p> 
<p><img alt="Figure 23" class="size-full wp-image-28918 aligncenter" height="440" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-23.png" width="491" /></p> 
<p style="text-align: center;"><em>Figure 23</em></p> 
<ol> 
 <li>Bob then creates an integration by navigating to the Amazon Quick console, selecting the Integration option, choosing S3, and then entering an access point name, selecting the correct Quick Suite instance account, and filling in the S3 bucket URL with the S3 Access Point alias from the FSx for ONTAP configuration page (<em>Figure 24</em>).</li> 
</ol> 
<p><img alt="Figure 24" class="size-full wp-image-28921 aligncenter" height="208" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-24.png" width="469" /></p> 
<p style="text-align: center;"><em>Figure 24</em></p> 
<ol start="2"> 
 <li>The integration fails with a status of <strong>Completed with issues</strong> and Sync status of <strong>Failed</strong> because Bob’s user account doesn’t have read permissions to the Support volume (<em>Figure 25</em>).</li> 
</ol> 
<p><img alt="Figure 25" class="size-full wp-image-28923 aligncenter" height="122" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-25.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 25</em></p> 
<p>Because Bob isn’t a member of the <strong>SupportAnalysts</strong> security group and he doesn’t have access to the volume with his user account, he needs to sign in using the <strong>SupportSvc</strong> service account, which is a member of the <strong>SupportAnalysts</strong> security group, to create an integration.</p> 
<h3>Example of using an integration with the Quick Suite chat agent</h3> 
<p>Alice has properly configured both the S3 Access point and the Quick Suite knowledge base integration. Using the chat agent, she can query the unstructured data in the Finance volume using natural language.</p> 
<p>Query: Can you read the data in the Alice Financial Knowledge Base?</p> 
<p>Response: Yes, I can access the Alice Financial Knowledge Base! . . .</p> 
<p><img alt="Figure 26" class="size-full wp-image-28925 aligncenter" height="488" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-26.png" width="352" /></p> 
<p style="text-align: center;"><em>Figure 26</em></p> 
<p>The agent will make some recommendations based on its understanding that this is financial transaction data and returns a list of possible queries (<em>Figure 26</em>). Following the prompt, the agent can create useful visualizations in addition to summarized reports for broad sets of data (<em>Figure 27</em>).</p> 
<p><img alt="Figure 27" class="size-full wp-image-28928 aligncenter" height="323" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-27.png" width="624" /></p> 
<p style="text-align: center;"><em>Figure 27:</em></p> 
<p>The agent can also be prompted to dive deeper into the results to gain even more insights (<em>Figure 28</em>).</p> 
<p><img alt="Figure 28" class="size-full wp-image-28930 aligncenter" height="661" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-28.png" width="741" /></p> 
<p style="text-align: center;"><em>Figure 28</em></p> 
<p>Bob can access the Support volume data by signing in using the <strong>SupportSvc</strong> account so that he has access to the <strong>SupportSvc</strong> knowledge base and <strong>SupportSvc</strong> S3 access point. He can then query the data using natural language and can test the limits of the agent (<em>Figure 29</em>).</p> 
<p><img alt="Figure 29" class=" wp-image-28932 aligncenter" height="496" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-29.png" width="457" /></p> 
<p style="text-align: center;"><em>Figure 29</em></p> 
<p>Additional prompts will reveal that there are more data sets that the agent considers relevant, but the prompts must be specific if the query needs to be limited to the call transcripts (<em>Figures 30/31</em>).</p> 
<p><img alt="Figure 30" class="size-full wp-image-28934 aligncenter" height="652" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-30.png" width="291" /></p> 
<p style="text-align: center;"><em>Figure 30</em></p> 
<p><img alt="Figure 31" class="size-full wp-image-28936 aligncenter" height="541" src="https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2026/04/23/Figure-31.png" width="277" /></p> 
<p style="text-align: center;"><em>Figure 31:</em></p> 
<p>Ultimately, you can obtain valuable insights from the pure text data that would otherwise be impossible to gather without using the Quick Suite chat agent. For example, you can ask the chat agent to return a month-by-month analysis of the data.</p> 
<h2>Conclusion</h2> 
<p>In this post, you saw how using Amazon S3 Access Points with <a href="https://aws.amazon.com/fsx/netapp-ontap/" rel="noopener" target="_blank">FSx for NetApp ONTAP</a> file systems in a Windows Active Directory environment gives you a powerful and flexible way to extend your existing enterprise file data to cloud-native AI and analytics services without compromising your security posture. By mapping S3 access points to specific Active Directory identities, you can enforce granular, per-user and per-department access controls at both the IAM and file system levels, helping to ensure that users can only access the data they’re authorized to see. Whether you’re enabling a finance analyst to run AI-powered research on quarterly reports or configuring a service account to safely feed support transcripts into a generative AI pipeline, you can use this dual-layer security model to meet your compliance requirements while unlocking the full potential of your unstructured file data. As your organization continues to migrate enterprise workloads to AWS, <a href="https://aws.amazon.com/s3/features/access-points/" rel="noopener" target="_blank">S3 Access Points for FSx for NetApp ONTAP</a> provide the bridge between your legacy file infrastructure and the modern AI services—such as <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> Knowledge Bases, <a href="https://aws.amazon.com/sagemaker/" rel="noopener" target="_blank">Amazon SageMaker</a>, and <a href="https://aws.amazon.com/quick/" rel="noopener" target="_blank">Amazon Quick Suite</a>—that your teams need to drive faster, smarter business decisions.</p>
