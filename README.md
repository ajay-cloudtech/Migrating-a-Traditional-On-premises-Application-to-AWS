# Migrating-a-Traditional-On-premises-Application-to-AWS

Migrating a Traditional On-premises Application to AWS: A Step-by-Step Journey
![image](https://github.com/user-attachments/assets/048cb763-5529-4dc9-abcc-ee71e8345eb5)


In today’s fast-paced digital landscape, migrating applications and databases to the cloud is no longer a luxury—it’s a necessity. Whether it’s for scalability, cost-efficiency, or improved performance, the cloud offers unparalleled advantages. I had the opportunity to execute a comprehensive migration of a three-tier web application to AWS. Here’s a detailed walkthrough of the key steps, tools, and processes involved in this journey.

## Step 1: Discovery and Assessment

The first step in any migration project is understanding your current environment. Deploy the AWS Application Discovery Service (ADS) agent on the on-premises web servers. ADS agent collects and syncs data about the servers, including OS, memory, disk usage, CPU, networking dependencies and performance metrics (E.g. CPU usage over time). It also provides visibility into Application Dependencies. By analyzing network traffic and server interactions, ADS creates a visual diagram of how applications and servers are connected which helps, identify which servers are part of the same application stack, understand the flow of data between components and avoid breaking critical dependencies during migration. For example, if your web server depends on a specific database server, ADS will highlight this relationship, ensuring both are migrated together.

The data collected by ADS is sent to the AWS Migration Hub, where it is organized and presented in a user-friendly format. This helps you to group servers into logical migration waves based on dependencies, estimate costs by matching on-premises server specifications to equivalent AWS resources (e.g. EC2 types) and identify potential challenges such as servers with high resource utilization or outdated operating systems. Based on the data collected by agent, Migration Hub provides "EC2 instance recommendations" which suggests suitable EC2 instance types (e.g., t3.medium, m5.large) for migrating your on-premises servers to AWS. This helps in right sizing, cost optimizations and migration planning.

On-prem servers detected by agent in Migration Hub
![image](https://github.com/user-attachments/assets/03beb9fc-a6a3-4a98-add5-fabe8c480ef8)

Server hardware specs in Migration Hub
![image](https://github.com/user-attachments/assets/d8e10084-361d-4b72-9cd5-b3c07d3ff907)

EC2 Instance Recommendations CSV file - Downloaded from Migration Hub
![image](https://github.com/user-attachments/assets/641e32eb-87de-405c-9302-ee3ec5012b8a)

Network map of servers and dependencies in Migration Hub
![image](https://github.com/user-attachments/assets/0f8d0801-b341-49ee-9f25-99fb9171391e)

## Step 2: Application Migration with MGN (Rehost)

Application Migration Service (MGN) service designed to simplify and automate the migration of applications to AWS. The first step is to initialize MGN to prepare the service for use. Why? Initializing activates MGN service in your account that helps create necessary backend resources (e.g., replication servers, storage, and networking components) to handle data replication from source servers to AWS. Then, customize the replication server template, for ensuring the migration process aligns with your specific environment and requirements - e.g. network alignment (subnet, VPC, etc.), performance (bandwidth, throughput, etc.), security (configure security groups)  and cost management (select EC2 types, storage, etc.).

Replication Agent: The replication agent installed on on-prem servers, perform a block-level read of all volumes attached to the web servers. This data is then replicated to the replication servers in AWS, which acts as staging area for the migration.

 Monitoring the Replication Process: Monitor the real time replication status in the Migration Hub. Once the initial sync and continuous replication are in place, the web server is marked as Ready for Testing.

Launching Test and Cutover Instances: After the initial replication is complete, launch a Test Instance to validate the migration. This step is crucial for ensuring that the application functions correctly in the new environment. Once the test is successful, proceed to launch the Cutover Instance, which marks the final step in the migration process.

Source server details in Application Migration Service (MGN)
![image](https://github.com/user-attachments/assets/556a8133-e220-4577-9263-7b70019fadc1)

Data replication status in MGN
![image](https://github.com/user-attachments/assets/74b6f184-e834-469d-9f5f-7c57c209cef5)

Migration dashboard in MGN
![image](https://github.com/user-attachments/assets/4f8f60e6-131d-49dc-a72a-bb60e543122f)

## Step 3: Database Migration with AWS DMS (Replatform)

With the discovery phase complete, the next step is to migrate the database. How? Use AWS Database Migration Service (DMS). Why? It is a managed service that ensures minimal downtime and zero data loss during the migration process. DMS supports a wide range of database engines, making it a versatile choice for both homogeneous and heterogeneous migrations. Key steps are

Setting Up the Target Database: Choose a target database that aligns with the structure and requirements of your original database (in my case it was AWS Relational Database Service - RDS, as source DB was MySQL). Once the target DB decision is made, configure the DB in AWS - this will involve configuring the VPC, subnets, database engine, credentials, storage, security groups, and backup options.

 Create Replication Instance: Once the database is ready, create a Replication Instance (which is just a fancy name for managed EC2 instance) in DMS. This instance acts as the backbone of the migration process, and hosts the replication tasks that move data from the source to the target database.

 Continuous Replication with Change Data Capture (CDC): To ensure minimal downtime, I configured the migration to use Continuous Replication with Change Data Capture (CDC). This feature captures changes made to the source database in real-time and replicates them to the target database. For this to work, I had to enable the Binary Log on the source MySQL database, which records all changes made to the database.

 Creating Source and Target Endpoints: Next, create Endpoints for both source and target DBs in DMS. These endpoints act as gateways to the source and target databases. The source endpoint will be configured to connect to the on-premises database server, while the target endpoint connected to the AWS DB instance.

 Launching the Replication Task: With the endpoints in place, create Replication Task in DMS. This task is responsible for moving data from the source to the target database. It specifies source and target endpoints and tables and schemas to migrate. You can monitor the task for migration status and continuous replication status.

Once the task status changes to 'Load complete,' congratulations—your database migration is complete!

Replication Instance in DMS
![image](https://github.com/user-attachments/assets/fb5e67bb-530f-40b7-b8d6-f820b056088e)

Source and Target DB Endpoints in DMS
![image](https://github.com/user-attachments/assets/9aa139c4-9c9c-43de-976d-cc98fd099dba)

Task details in DMS
![image](https://github.com/user-attachments/assets/0b4d98a8-14e6-4b15-9ca2-c5a149a67aa5)

Task status in DMS
![image](https://github.com/user-attachments/assets/672cdd51-90a9-46ec-a83a-674d07ccfd42)

Task statistics in DMS
![image](https://github.com/user-attachments/assets/94269ace-c39c-4541-be36-9c440b029f05)

## Step 4: Post-Migration Configuration

With the application and database now running in AWS, the final step is to reconfigure the application to use the new database. This involves updating the application’s configuration files to point to the Amazon RDS instance.

Finalizing the Cutover: Once everything is configured and tested, finalize the cutover in MGN. This step terminates the replication resources and uninstalls the replication agent from the source servers. The migration is now complete, and the application is fully operational in AWS.

Application migration confirmation with final cutover in MGN
![image](https://github.com/user-attachments/assets/37917f10-1ece-463d-a3a8-70bbd1d669f3)

## Conclusion

Migrating an application and its database to AWS is a complex but rewarding process. By leveraging AWS services like Application Discovery Service, Database Migration Service, and Application Migration Service, I was able to execute the migration with minimal downtime and zero data loss. The key to success lies in thorough planning, careful execution, and continuous monitoring throughout the process.

This migration not only improved the application’s performance and scalability but also set the stage for future modernization efforts. If you’re considering a similar migration, I highly recommend exploring these AWS services and following a structured approach to ensure a smooth transition.
