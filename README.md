## Three-tier Web App (WordPress) Architecture On AWS

### Introduction
---
 This design document provides a design for a 3-tier web application architecture leveraging AWS (Amazon Web Services) cloud services. This architecture design aims to deliver a scalable, highly available, and secure web application environment that separates presentation, application logic, and data layers.
 By following the proposed 3-tier architecture, organizations can benefit from improved scalability, increased fault tolerance, and enhanced security for their web applications.
 
 ---
 ### Architecture Over View
 
![enter image description here](https://github.com/ahmedflqn/AWSCLOUDASSIGNMENT/blob/main/Architecture%20diagram/WordPress%20architecture%20diagram%20(4).png?raw=true)
 
 The Three-tier Architecture consists of the following layers:
 
 - Presentation tier: Describe the components responsible for user interaction, such as web servers, load balancers, and content delivery networks (CDNs). This architecture will consist of Cloudfront as our CDN and Elastic Load Balancer.
 - Application tier: responsible for implementing application logic, Processing data, and facilitating the communication between the presentation layer and the data layer. It will consist of EC2 instances in an Auto scaling group.
 - Data tier: Focuses on managing and storing data required by the application and provides the necessary functionality for data access, storage, and retrieval. We will use RDS in a multi-AZ deployment for the database and EFS as our storage.
 
 ### Architecture components
 **First our infrastructure** :
 will consist of a VPC with  2  public  and 4 private  subnets  spread  across  two   Availability  Zones. Internet  gateway,  with  a  default  route  on  the  public  subnets.  Pair  of  NAT  gateways  (one  in  each  AZ),  and  default  routes  for  them  in  the  private  subnets. Site to Site VPN connection which consist of VPN Gateway on our VPC and Customer Gateway in the Customer office to create a secure connection between them.
 
 **Second our AWS Services**:
 
 - Route 53 Allows us to register a domain name for our web app and acts as a DNS service, allowing us to manage the DNS records associated with our domain. We will use Route 53 Geolocations routing policy to route users based on their location so if a user lives in USA he will be routed to the IP which contains the English version of our App.
 - Cloudfront is our content delivery network it caches and delivers static and dynamic content, including web pages, images, videos, and other media files, from edge locations worldwide for faster delivery and lower latency and improving performance by reducing load on our application and Also improve security
 - Application Load Balancer Will ensure high availability and scalability by distributing incoming traffic
 - Autoscaling group to scale based on the capacity you need
 - EC2 It is our computing power and will host the application servers in the application tier
 - Internet Gateway to enable communication between resources in VPC and the internet
 - NAT Gateway in each availability Zone to enable EC2  instances in a private subnet to access the internet
 -  Amazon RDS to host our Web App database in a Multi-AZ deployment 
 - Amazon EFS  Will be our Web App shared file storage. EC2 can access shared App data via EFS Mount Target in every availability Zone 

---
 

 **High availability**
 
 Our architecture consists of two availability zones which will allow us to distribute our EC2, NAT Gateway across different locations to eliminate one point of failure
 
We will deploy our EC2 instances behind an Application Load Balancer, The Load Balancer distributes incoming traffic across these instances so if one instance becomes unavailable it will direct traffic to a healthy instance.

Our RDS will be deployed in Multi-AZ which use synchronous replication to ensure that the database remains available if one of the database fails and will reduce downtime and ensure data availability.

EFS offers an SLA of four-nine availability (99.99%). 

**Scalability**

AWS Autoscaling group to automatically add or remove instances based on predefined scaling policies.
Load Balancer to evenly distribute traffic across multiple instances evenly ensure application can scale by adding more instances.

Our RDS can vertically scale by using larger instance types. and we can create RDS Read Replica to scale Reads if it's required.

EFS automatically scales its capacity and throughput as you add or remove files.

**Securtiy**

AWS WAF attached on Cloudfront to filter malicious traffic, including cross-site scripting and SQL injections via rules.

AWS Sheild for DDos Protection

AWS IAM to apply fine-grained permission to AWS resources and services.

For data protection
data in transit Use SSL/TLS encryption to protect data transmitted between client and web app.
Enabling HTTPS for secure communication.
Encrypt data at rest in RDS, EBS, and EFS.

For Networking 
We will deploy only Nat Gateway in the public subnets, and for our App instances and RDS will be deployed in private subnets. We will access our App instance through AWS Systems Manager.
Security Groups:
Load Balancer: Allow only inbound HTTP, HTTPS from anywhere. 
App instances: Allow HTTPS from our VPC CIDR to enable Systems manager to access our EC2, Also allow HTTPS,HTTP from Load Balancer security group
RDS: Allow inbound from our App security group on port 3306 that is used for database connections.
EFS: Allow inbound from our App security group on port 2049

**Cost optimization**

We will use AWS Trusted Advisor to identify underutilized resources that can be downsized or terminated.
Analyze our workloads to adjust the capacity of our EC2 instances and RDS Instances
We will use Reserved instances for cost savings
Implement cost allocation tags to track and categorize costs by project, application, or team. This enables better visibility and cost management.
Implement auto-scaling across our application tiers to dynamically adjust resource capacity based on demand.

**High Performance**

Cloudfront will improve performance, latency, and delivery by caching content in edge locations.
Route 53 provides global load balancing and intelligent routing to direct user requests to the nearest available web server.
Load Balancer and Auto scaling to improve performance by distributing traffic adjusts number of instances based on utilization which ensures the application can handle variable loads.
RDS we can use Read Replica if we need to scale our read which will reduce the load on our primary RDS instance and improve performance.
RDS can scale automatically if detects that you are running out of space.
EFS with General Purpose Mode which will be Suitable for most workloads, it offers a balance of throughput, IOPS, and latency. It automatically scales with the size of the file system.

**CI/CD for the application development**

![enter image description here](https://github.com/ahmedflqn/AWSCLOUDASSIGNMENT/blob/main/Architecture%20diagram/CICD.png?raw=true)

Continuous integration CI
Developers will push code to the code repository  AWS CodeCommit
testing and building server will check the code as soon it's pushed AWS CodeBuild
get the results to check if the test failed or passed so we can find bugs and fix them early

continuous delivery CD
CodeDeploy to automate deployment to the server when the build is passed
ensure deployments happen often and quick, Define rollback and rollforward strategies

**Observability**

Metrics: data point that represents the behavior, performance, or state of a particular AWS resource or service
Logs: It's a way to provide visibility into usage, errors and activity of system information
Traces: Representation of a single request as it passes through the distributed application system

**Creating an observability strategy for a cloud-based application**

Observability Is the ability to measure a system's current state based on the data it generates.
The observability of a system has a significant impact on its operating and development costs. Observable systems yield meaningful, actionable data to their operators, allowing them to achieve favorable outcomes

Our strategy:

*1-* Identify the objective of our strategy
Each business outcome has a set of Key Performance Indicators (KPIs) associated with it, they’re definition of what can be measured to evaluate the successful achievement of business outcomes. 

*2-* Determine what to observe 
Identify critical services, functions, data flows etc. that need monitoring
Determine relevant metrics like latency, errors, throughput etc.

*3-* Instrument code 
Add SDKs or open-tracing libraries to auto-collect metrics, logs and traces
Add manual instrumentation for custom metrics

*4-* Collect and aggregate 
logs Configure applications to send logs to CloudWatch Logs
Aggregate logs from multiple sources in one place

*5-* Set up Metrics and Alarms 
Set appropriate threshold values to trigger notifications or automated actions when metrics breach predefined thresholds. Define actions to be taken when the threshold is crossed

*6-* Analyze metrics and traces
 Visualize time-series metrics and events in CloudWatch Dashboards/Insight
Analyze traces to debug performance issues

*7-* Evolving your observability strategy over time
Regularly review and refine your observability strategy based on feedback, lessons learned, and changing application requirements. 


**AWS consumption calculator**

![enter image description here](https://github.com/ahmedflqn/AWSCLOUDASSIGNMENT/blob/main/Consumption%20Calculator/Consumption%20calculator.PNG?raw=true)

**Short-term and long-term recommendations**

Short-term

1-  Continuously monitor and optimize your AWS resources to ensure cost efficiency.

2- Implement a caching layer using services like Amazon ElastiCache (Redis or Memcached) 

3- Set up comprehensive monitoring and logging using AWS CloudWatch, AWS X-Ray

4- Create RDS read replica 


Long-term

1- Adopt containerization using services like Amazon Elastic Container Service (ECS) or Amazon Elastic Kubernetes Service (EKS).

2- Upgrade your RDS to Aurora for higher performance, scalability, and durability

3- Deploy in Multiple Regions for Disaster Recovery to get a fault-tolerant application






 






















 
 


 
