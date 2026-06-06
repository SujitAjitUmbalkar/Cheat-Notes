
# Core AWS Services

| Tool / Service            | Use (Short)                     |
| ------------------------- | ------------------------------- |
| Amazon Web Services (AWS) | Cloud platform by Amazon        |
| EC2                       | Virtual machine/server in cloud |
| S3                        | Store files, images, backups    |
| RDS                       | Managed SQL database            |
| IAM                       | User permissions and roles      |
| VPC                       | Private network inside AWS      |
| Lambda                    | Run code without server         |
| CloudWatch                | Logs and monitoring             |
| Route 53                  | Domain and DNS management       |
| CloudFront                | Fast content delivery           |
| EBS                       | Disk/storage attached to EC2    |
| Elastic Beanstalk         | Deploy apps easily              |
| ECS                       | Run Docker containers           |
| EKS                       | Managed Kubernetes              |
| API Gateway               | Create/manage APIs              |
| SNS                       | Send notifications              |
| SQS                       | Message queue between services  |

---

# Important DevOps / Deployment Terms

| Word              | Meaning                                 |
| ----------------- | --------------------------------------- |
| AMI               | Image/template for EC2                  |
| Security Group    | Firewall rules for EC2                  |
| Key Pair          | Login key for EC2                       |
| Auto Scaling      | Automatically increase/decrease servers |
| Load Balancer     | Distributes traffic across servers      |
| Region            | AWS location (Mumbai, Ohio, etc.)       |
| Availability Zone | Data center inside region               |
| Instance          | Running EC2 server                      |
| CIDR              | IP range notation                       |
| Public IP         | Internet accessible IP                  |
| Private IP        | Internal network IP                     |

---

# CI/CD Related AWS Tools

| Tool         | Use                      |
| ------------ | ------------------------ |
| CodePipeline | Complete CI/CD pipeline  |
| CodeBuild    | Build/test application   |
| CodeDeploy   | Deploy app automatically |
| CodeCommit   | Git repository by AWS    |

---

# Database Terms

| Tool       | Use                           |
| ---------- | ----------------------------- |
| MySQL      | Relational database           |
| PostgreSQL | Advanced SQL database         |
| DynamoDB   | NoSQL database                |
| Aurora     | High-performance AWS database |

---

# Docker / Container Terms

| Tool       | Use                             |
| ---------- | ------------------------------- |
| Docker     | Create containers               |
| Container  | Lightweight packaged app        |
| Image      | Blueprint/template of container |
| Kubernetes | Container orchestration         |
| Pod        | Smallest Kubernetes unit        |

---

# Common Real-World Flow

Spring Boot App → Docker → Push to GitHub → CI/CD → EC2/ECS → RDS Database → CloudWatch Logs

This is the flow teachers usually discuss during deployment classes.

---


# 1] AWS IAM (Identity and Access Management)

## Definition

IAM is an AWS service used to securely control access to AWS resources. It determines **who can access AWS** and **what actions they can perform**.

**IAM = Authentication + Authorization**

* Authentication → Verifies identity (Who are you?)
* Authorization → Determines permissions (What can you do?)

---

## IAM Components

### 1. User

An IAM User represents an individual person or application that needs access to AWS.

Examples:

* Admin
* Developer
* DevOps Engineer

A user can access AWS through:

* AWS Management Console (Username & Password)
* AWS CLI/SDK (Access Keys)

---

### 2. Group

A Group is a collection of IAM users.

Example:

* Developers Group
* Admin Group
* Testers Group

Permissions are assigned to the group instead of assigning them individually to every user.

---

### 3. Policy

A Policy is a JSON document that defines permissions.

It specifies:

* What actions are allowed or denied
* Which AWS resources can be accessed

Example:

* S3 Read Access
* EC2 Full Access
* RDS Read-Only Access

Types of Policies:

1. AWS Managed Policies
2. Customer Managed Policies
3. Inline Policies

---

### 4. Role

An IAM Role provides temporary permissions to AWS services or users.

Unlike users, roles do not have:

* Username
* Password
* Access Keys

Common Use Cases:

* EC2 accessing S3
* ECS pulling images from ECR
* Lambda accessing RDS

Roles are the preferred and secure way of granting permissions.

---

### 5. Access Keys

Used for programmatic access through:

* AWS CLI
* SDKs
* Applications

Consist of:

* Access Key ID
* Secret Access Key

Access keys should be rotated regularly and never shared publicly.

---

## IAM Security Best Practices

1. Never use the Root Account for daily work.
2. Enable Multi-Factor Authentication (MFA).
3. Follow the Principle of Least Privilege.
4. Use IAM Roles instead of Access Keys whenever possible.
5. Create Groups and assign permissions through groups.
6. Rotate credentials periodically.
7. Remove unused users and permissions.

---

## User vs Role

| User                     | Role                              |
| ------------------------ | --------------------------------- |
| Permanent Identity       | Temporary Identity                |
| Used by Humans           | Used by AWS Services/Applications |
| Has Password/Access Keys | No Permanent Credentials          |

---

## Real-World Example

A Spring Boot application is deployed on an EC2 instance.

Flow:
Developer → IAM User → Deploy Application

EC2 Instance → IAM Role → Access S3 Bucket

This avoids storing AWS credentials inside the application.

---

## Quick Revision

IAM = Identity and Access Management

User      → Individual identity
Group     → Collection of users
Policy    → Permission document (JSON)
Role      → Temporary permissions
AccessKey → CLI/SDK access

Best Practices:

* Use Roles
* Enable MFA
* Least Privilege Principle
* Avoid Root User
* Use Groups for permission management

---

# 2] AWS EC2 (Elastic Compute Cloud)

## Definition

Amazon EC2 (Elastic Compute Cloud) is a service that provides virtual servers in the cloud.

It allows users to launch, manage, and scale servers without purchasing physical hardware.

**EC2 = Virtual Machine (VM) in AWS**

Common Uses:

* Hosting Spring Boot applications
* Running websites
* Deploying APIs
* Running Docker containers
* Hosting databases (for learning/testing)

---

## EC2 Components

### 1. Instance

An EC2 Instance is a virtual server.

Examples:

* t2.micro
* t3.micro
* t3.small
* m5.large

Each instance provides:

* CPU
* RAM
* Storage
* Network

---

### 2. AMI (Amazon Machine Image)

AMI is a template used to launch an EC2 instance.

It contains:

* Operating System
* Software
* Configuration

Examples:

* Amazon Linux
* Ubuntu
* Windows Server

Think of AMI as a "server blueprint."

---

### 3. Instance Types

AWS provides different instance sizes.

Categories:

#### General Purpose

Balanced CPU and RAM

Examples:

* t2.micro
* t3.micro

#### Compute Optimized

High CPU performance

Examples:

* c5.large

#### Memory Optimized

High RAM

Examples:

* r5.large

---

### 4. EBS (Elastic Block Store)

EBS is the storage attached to EC2.

Features:

* Persistent storage
* Data remains after reboot
* Can create snapshots
* Can increase size later

Think of EBS as the hard disk of your EC2 server.

---

### 5. Security Group

Acts as a virtual firewall for EC2.

Controls:

* Incoming Traffic (Inbound Rules)
* Outgoing Traffic (Outbound Rules)

Examples:

Port 22 → SSH

Port 80 → HTTP

Port 443 → HTTPS

Port 8080 → Spring Boot Application

---

### 6. Key Pair

Used for secure login into EC2.

Consists of:

* Public Key (stored by AWS)
* Private Key (.pem file)

Example:

```bash
ssh -i mykey.pem ec2-user@public-ip
```

Never share the private key.

---

### 7. Elastic IP

A static public IP address.

Purpose:

* Fixed IP for applications
* IP remains same after restart

Without Elastic IP:

* Public IP may change after stop/start.

---

## EC2 States

1. Pending
2. Running
3. Stopping
4. Stopped
5. Rebooting
6. Terminated

Important:

* Terminated instances cannot be recovered.

---

## Scaling EC2

### Vertical Scaling

Increase instance size.

Example:

```text
t2.micro → t3.small
```

More CPU and RAM.

---

### Horizontal Scaling

Add multiple EC2 instances.

Example:

```text
1 Server → 5 Servers
```

Used with Load Balancer.

---

## Real-World Deployment Flow

```text
Developer
    |
    v
GitHub
    |
    v
EC2 Instance
    |
Spring Boot Application
    |
RDS Database
```

A Spring Boot JAR is commonly deployed on an EC2 instance.

---

## Best Practices

1. Use IAM Roles instead of Access Keys.
2. Open only required ports in Security Groups.
3. Take EBS Snapshots regularly.
4. Use Elastic IP only when required.
5. Stop unused instances to reduce cost.
6. Enable monitoring through CloudWatch.

---

## Interview Questions

### What is EC2?

EC2 is a service that provides scalable virtual servers in AWS.

### What is AMI?

A template containing OS and configurations used to launch EC2 instances.

### What is Security Group?

A virtual firewall controlling inbound and outbound traffic.

### Difference Between EBS and S3?

EBS:

* Block Storage
* Attached to EC2

S3:

* Object Storage
* Independent storage service

---

## Quick Revision

EC2 = Virtual Server in AWS

Instance       → Virtual Machine
AMI            → Server Template
EBS            → Hard Disk
Security Group → Firewall
Key Pair       → Login Credentials
Elastic IP     → Static Public IP

Ports:
22   → SSH
80   → HTTP
443  → HTTPS
8080 → Spring Boot

Scaling:
Vertical   → Bigger Server
Horizontal → More Servers

---

# AWS S3 (Simple Storage Service)

## Definition

Amazon S3 (Simple Storage Service) is AWS's object storage service used to store and retrieve files from anywhere on the internet.

S3 is highly durable, scalable, and cost-effective.

Common Uses:

* Store images and videos
* Application file uploads
* Static website hosting
* Backups
* Log storage
* Data archiving

---

## Basic Concepts

### 1. Bucket

A Bucket is a container that stores objects (files).

Example:

```text
my-app-files
employee-documents
springboot-backups
```

Rules:

* Bucket names must be globally unique.
* Created in a specific AWS Region.

---

### 2. Object

An Object is a file stored inside a bucket.

Examples:

```text
resume.pdf
profile.jpg
video.mp4
backup.zip
```

Each object contains:

* Data
* Metadata
* Unique Key

---

### 3. Object Key

The unique name/path of an object.

Example:

```text
images/profile.jpg
documents/resume.pdf
```

Think of it as the file path inside S3.

---

## S3 Storage Classes

### 1. S3 Standard

Used for frequently accessed data.

Examples:

* Website images
* Application files

Features:

* High availability
* Low latency

---

### 2. S3 Standard-IA

IA = Infrequent Access

Used when data is accessed occasionally.

Examples:

* Monthly reports
* Old documents

Lower storage cost than Standard.

---

### 3. S3 Glacier Instant Retrieval

Used for archival data that is rarely accessed.

Examples:

* Historical records
* Compliance data

Very low storage cost.

---

### 4. S3 Glacier Deep Archive

Cheapest storage class.

Used for long-term backups.

Examples:

* Old company backups
* Legal archives

---

## Bucket Versioning

Versioning keeps multiple versions of the same file.

Example:

```text
resume.pdf (v1)
resume.pdf (v2)
resume.pdf (v3)
```

Benefits:

* Recover deleted files
* Restore older versions

---

## Lifecycle Rules

Automatically move files between storage classes.

Example:

```text
After 30 Days
    ↓
Standard → Standard-IA

After 180 Days
    ↓
Glacier
```

Helps reduce storage cost.

---

## Bucket Policies

Bucket policies control who can access bucket contents.

Example:

* Public Read
* Private Access
* Specific IAM Users

Uses JSON policies similar to IAM.

---

## Pre-Signed URL

A temporary URL that allows secure access to a private file.

Example:

```text
Private S3 File
        |
Pre-Signed URL
        |
User Downloads File
```

Commonly used in Spring Boot applications.

---

## Static Website Hosting

S3 can host static websites.

Supports:

* HTML
* CSS
* JavaScript

Example:

```text
index.html
style.css
app.js
```

No server required.

---

## Security Features

### Encryption

Protects stored data.

Types:

1. SSE-S3
2. SSE-KMS
3. Client-Side Encryption

---

### Access Control

Using:

* IAM Policies
* Bucket Policies
* ACLs (less commonly used)

---

## Real-World Example

Spring Boot Application:

```text
User Uploads Image
        |
Spring Boot API
        |
        v
      S3 Bucket
        |
Store Image URL in RDS
```

Instead of storing images in database, store them in S3.

---

## S3 vs EBS

| S3                  | EBS                 |
| ------------------- | ------------------- |
| Object Storage      | Block Storage       |
| Stores Files        | Stores Disk Blocks  |
| Independent Service | Attached to EC2     |
| Unlimited Scaling   | Limited to Instance |

---

## Best Practices

1. Enable Versioning.
2. Use Lifecycle Rules.
3. Keep Buckets Private by Default.
4. Encrypt Sensitive Data.
5. Use Pre-Signed URLs for secure downloads.
6. Store backups in Glacier classes.

---

## Interview Questions

### What is S3?

AWS object storage service used to store files.

### What is a Bucket?

A container used to store objects in S3.

### What is Versioning?

Maintains multiple versions of the same file.

### What is a Pre-Signed URL?

A temporary URL used to access private S3 objects securely.

### Difference Between S3 and EBS?

S3 stores objects/files, whereas EBS provides block storage attached to EC2.

---

## Quick Revision

S3 = Object Storage

Bucket        → Container
Object        → File
Key           → File Path
Versioning    → Multiple File Versions
Lifecycle     → Automatic Cost Optimization
Bucket Policy → Access Control
Pre-Signed URL→ Temporary Secure Access

Storage Classes:
Standard
Standard-IA
Glacier Instant Retrieval
Glacier Deep Archive

Common Uses:

* File Uploads
* Image Storage
* Backups
* Static Websites
* Logs

```
```


### **Amazon S3 (Simple Storage Service)**

Amazon S3 is a highly scalable, secure, and durable object storage service provided by AWS. It allows users to store and retrieve virtually any amount of data from anywhere on the internet, acting as a foundational building block for modern cloud infrastructure.

* **How It Works:** Data is stored as "objects" (the files themselves) within distinct containers called "buckets." Each object is assigned a unique key identifier along with customizable metadata for easy management and retrieval.
* **Core Benefits:**
* **Unmatched Durability:** Engineered to provide 99.999999999% (11 9's) data durability, making the likelihood of data loss mathematically almost impossible by storing copies across multiple physical facilities.
* **Elastic Scalability:** Storage capacity automatically expands or shrinks seamlessly based on exact demand without any performance degradation.
* **Robust Security:** Provides comprehensive encryption capabilities, detailed access logs, and fine-grained access controls to secure sensitive information.


* **Primary Use Cases:** S3 is the industry standard for creating reliable data backups, archiving long-term compliance records, hosting static websites, and building massive data lakes to power machine learning and analytics.




### **Amazon EC2 (Elastic Compute Cloud)**

Amazon EC2 हे AWS मधील एक अत्यंत लोकप्रिय आणि महत्त्वाचे service आहे, जे cloud मध्ये resizable compute capacity म्हणजेच virtual servers पुरवते. सोप्या भाषेत सांगायचे तर, तुम्ही internet द्वारे cloud मध्ये स्वतःचा एक virtual computer भाड्याने घेऊ शकता आणि त्यावर तुमचे applications run करू शकता.

* **How It Works:** तुम्ही तुमच्या गरजेनुसार CPU, RAM, आणि storage निवडून काही मिनिटांत एक Instance (virtual machine) तयार करू शकता. यावर तुम्ही तुमच्या आवडीचा Operating System (जसे की Linux किंवा Windows) आणि software stack configure करू शकता.
* **Core Benefits:**
* **Scalability:** ट्रॅफिक वाढल्यास तुम्ही Auto Scaling चा वापर करून तुमच्या servers ची संख्या स्वयंचलितपणे (automatically) वाढवू शकता किंवा कमी करू शकता.
* **Cost-Effective:** यामध्ये pay-as-you-go मॉडेल असते, म्हणजेच तुम्ही जितका वेळ instance active ठेवता फक्त तितक्याच वेळेचा charge तुम्हाला द्यावा लागतो.
* **Full Control:** तुम्हाला तुमच्या virtual server चा पूर्ण administrative किंवा root access मिळतो, ज्यामुळे तुम्ही ते तुमच्या प्रोजेक्टच्या गरजेनुसार configure करू शकता.



**Real-world project use:**
एका E-commerce website प्रोजेक्टमध्ये, जेव्हा एखादा मोठा festival sale असतो तेव्हा अचानक येणाऱ्या प्रचंड traffic ला handle करण्यासाठी Amazon EC2 instances चा वापर Auto Scaling सोबत केला जातो, ज्यामुळे server वर लोड न येता website crash न होता smooth चालते आणि नंतर ट्रॅफिक कमी झाल्यावर जास्तीचे instances automatic delete होतात.

Cloud computing च्या जगात स्वतःचे application किंवा backend deploy करण्यासाठी आणि infrastructure वर पूर्ण नियंत्रण मिळवण्यासाठी EC2 हा एक अत्यंत पायाभूत, लवचिक आणि विश्वासार्ह पर्याय मानला जातो जो कोणत्याही आधुनिक web application ला यशस्वीरीत्या चालवू शकतो.



### **Amazon RDS (Relational Database Service)**

Amazon RDS is a managed service that makes it easy to set up, operate, and scale a relational database in the cloud. It automates time-consuming administration tasks such as hardware provisioning, database setup, patching, and backups.

* **How It Works:** You choose a database engine (like MySQL, PostgreSQL, SQL Server, or Oracle), and AWS provisions the infrastructure, managing the underlying operating system and database software while giving you standard database access.
* **Core Benefits:**
* **Automated Management:** Handles routine database maintenance tasks like automated backups, software patching, and automatic failure detection and recovery.
* **High Availability:** Offers Multi-AZ deployments that automatically replicate data to a standby instance in a different availability zone to ensure disaster recovery.
* **Scalability:** You can seamlessly scale your database's compute and storage resources with just a few clicks or an API call without any significant downtime.



**Real-world project use:**
In a banking web application, Amazon RDS is utilized to securely manage structured financial transactions, user accounts, and balance records. It ensures strict data consistency and automatic failover capabilities, guaranteeing that customer financial data is always accurate and highly available even during localized hardware failures.

Understanding these core cloud services is essential for building scalable software, and leveraging managed databases like RDS significantly reduces operational overhead by allowing you to focus entirely on writing business logic rather than dealing with server maintenance. If you want to explore how these cloud components integrate with your specific backend architectures or need details on any other services, just let me know and we can dive deeper into the technical implementations.


# AWS Rare but Important Service: AWS Systems Manager (SSM)

## What is SSM?

AWS Systems Manager helps you **manage, monitor, and automate EC2 instances and servers** from a central place.

---

## Why Use It?

* Connect to EC2 without SSH
* Run commands on multiple servers
* Store configuration and secrets
* Automate maintenance tasks
* Patch operating systems automatically

---

## Important Features

### 1. Session Manager ⭐

Login to EC2 directly from AWS Console.

```text
AWS Console
     ↓
Session Manager
     ↓
EC2 Instance
```

No SSH key required.

---

### 2. Run Command

Execute commands on many servers at once.

Example:

```bash
sudo yum update -y
```

Runs on multiple EC2 instances.

---

### 3. Parameter Store

Stores:

* Database passwords
* API keys
* Environment variables

Example:

```text
/db/password
/api/key
```

---

### 4. Patch Manager

Automatically updates OS patches.

---

## Interview One-Liner

**SSM is a service used to manage, automate, patch, and securely access EC2 instances without SSH.**

---

## Short Notes

```text
AWS Systems Manager (SSM)

• Manage EC2 from one place
• Session Manager = Login without SSH
• Run Command = Execute commands remotely
• Parameter Store = Store secrets/configs
• Patch Manager = Automatic updates
• Improves security and automation
```

### Real Industry Usage

Many companies **disable SSH access completely** and use **SSM Session Manager** to access EC2 instances securely. This is a common DevOps practice.


# AWS Rare but Important Service: AWS EventBridge

## What is EventBridge?

**EventBridge** is AWS's event bus service.

It listens for events and automatically triggers actions.

---

## Real Example

```text
User uploads file to S3
         ↓
EventBridge detects event
         ↓
Triggers Lambda
         ↓
Processes file
```

---

## Why Use It?

* Event-driven architecture
* Automates workflows
* Connects AWS services
* Reduces manual coding

---

## Important Components

### 1. Event

Something happens.

Examples:

* EC2 started
* S3 file uploaded
* RDS backup completed

---

### 2. Rule

Defines when to react.

Example:

```text
IF file uploaded to S3
THEN trigger Lambda
```

---

### 3. Target

Action to perform.

Targets can be:

* Lambda
* ECS
* SQS
* SNS
* Step Functions

---

## Common Use Cases

### Auto Notification

```text
EC2 Stopped
      ↓
EventBridge
      ↓
SNS Email
```

---

### Automated Deployment

```text
Code Commit
      ↓
EventBridge
      ↓
CodePipeline
```

---

## Interview One-Liner

**EventBridge captures events from AWS services and routes them to targets for automated processing.**

---

## Short Notes

```text
AWS EventBridge

• Event-driven service
• Event = Something happened
• Rule = Condition
• Target = Action
• Connects AWS services
• Triggers Lambda, ECS, SQS, SNS etc.
• Used for automation and integrations
```

### Quick Memory Trick

```text
Event Happens
      ↓
EventBridge Catches
      ↓
Rule Matches
      ↓
Target Executes
```

**Rare Service Learned:** ✅ EventBridge

**Next rare service suggestion:** AWS Step Functions (very useful in microservices and interviews).



# AWS Rare but Important Service: AWS Step Functions

## What is Step Functions?

**Step Functions** lets you build workflows by connecting multiple AWS services in a sequence.

Think of it as an orchestrator that controls what happens first, next, and last.

---

## Real Example

Order Processing System:

```text
Order Placed
     ↓
Validate Payment
     ↓
Reserve Inventory
     ↓
Generate Invoice
     ↓
Send Email
```

Instead of writing complex code, Step Functions manages the flow.

---

## Why Use It?

* Orchestrates multiple services
* Handles retries automatically
* Error handling built-in
* Visual workflow designer
* Reduces application complexity

---

## Key Concepts

### 1. State Machine ⭐

The workflow definition.

Example:

```text
Start
 ↓
Task 1
 ↓
Task 2
 ↓
End
```

---

### 2. Task State

Performs actual work.

Examples:

* Lambda execution
* ECS task
* DynamoDB operation
* SQS message send

---

### 3. Choice State

Works like an `if-else`.

```text
Payment Success?
    ↓ Yes
 Ship Order

    ↓ No
 Cancel Order
```

---

### 4. Retry & Error Handling

If a task fails:

```text
Task Failed
     ↓
Retry 3 Times
     ↓
Still Failed
     ↓
Error Workflow
```

No need to code retry logic manually.

---

## Common Use Cases

### ETL Pipeline

```text
Read Data
   ↓
Transform Data
   ↓
Store Data
```

### Microservices Workflow

```text
Service A
   ↓
Service B
   ↓
Service C
```

### Approval Process

```text
Request
   ↓
Manager Approval
   ↓
Execute Action
```

---

## Interview One-Liner

**AWS Step Functions is a workflow orchestration service used to coordinate multiple AWS services through state machines.**

---

## Short Notes

```text
AWS Step Functions

• Workflow orchestration service
• State Machine = Workflow
• Task State = Execute work
• Choice State = If-Else logic
• Built-in Retry & Error Handling
• Visual workflow design
• Commonly used with Lambda, ECS, SQS
• Useful in microservices and ETL pipelines
```

### Memory Trick

```text
Many Services
      ↓
Step Functions
      ↓
One Workflow


AWS Lambda
Serverless compute service
Run code without managing servers
Pay only when code executes
Triggered by S3, API Gateway, SQS, SNS, EventBridge

Keywords: Function, Trigger, Event, Execution Role

--- 
AWS VPC (Very Important)
Private network inside AWS
Control networking and security

Sub-components

VPC
Subnet
Internet Gateway
NAT Gateway
Route Table
Security Group
NACL


Auto Scaling
Automatically adds/removes EC2 instances
Maintains application availability


## AWS Fargate

### What is it?

AWS Fargate is a **serverless compute engine for containers** that works with ECS and EKS.

### Why use it?

* No need to manage EC2 servers.
* AWS automatically handles infrastructure.
* Focus only on your application and containers.

### Features

* Serverless container execution.
* Automatic scaling.
* Pay only for CPU and memory used.
* Better security through task isolation.

### Use Case

Deploy a Spring Boot application in a Docker container on ECS without creating or managing EC2 instances.

### Important Terms

* Task
* Task Definition
* Cluster
* CPU & Memory Allocation

### Interview One-Liner

> AWS Fargate allows us to run Docker containers on ECS/EKS without provisioning or managing EC2 instances.

### Short Notes

* Serverless Containers
* Works with ECS & EKS
* No EC2 management
* Auto Scaling
* Pay-as-you-go
* Ideal for microservices and containerized applications

## AWS App Runner

### What is it?

AWS App Runner is a fully managed service that deploys web applications and APIs directly from source code or Docker images.

### Why use it?

* No need to manage servers, load balancers, or scaling.
* Quick deployment of Spring Boot, Node.js, Python, and other applications.
* Automatically handles HTTPS, scaling, and monitoring.

### Features

* Deploy from GitHub or ECR.
* Automatic scaling.
* Built-in load balancing.
* Secure HTTPS endpoint.

### Use Case

Deploy a Spring Boot REST API directly from a Docker image stored in ECR with minimal AWS configuration.

### Important Terms

* Service
* Source Repository
* Auto Scaling
* Instance Configuration

### Interview One-Liner

> AWS App Runner is a fully managed service that automatically builds, deploys, and scales web applications and APIs.

### Short Notes

* Fully Managed
* Deploy from GitHub/ECR
* Auto Scaling
* Built-in HTTPS
* No Infrastructure Management
* Faster than setting up ECS + Load Balancer for simple applications



## AWS Service: AWS Lambda

### What is AWS Lambda?

AWS Lambda is a **serverless compute service** that lets you run code without managing servers.

You upload your code, and Lambda automatically executes it when triggered.

### Key Features

* No server management
* Automatic scaling
* Pay only for execution time
* Supports Java, Python, Node.js, C#, Go, etc.

### Common Triggers

* S3 file upload
* API Gateway request
* DynamoDB changes
* SQS messages
* Scheduled events (cron jobs)

### Example

1. User uploads an image to S3.
2. S3 triggers Lambda.
3. Lambda resizes the image.
4. Resized image is saved back to S3.

### Benefits

* Cost-effective for small workloads
* Scales automatically
* Easy event-driven architecture

### Interview One-Liner

**AWS Lambda is a serverless service that runs code in response to events without requiring server management.**


## AWS Service: AWS Step Functions

### What is AWS Step Functions?

AWS Step Functions is a service used to **orchestrate multiple AWS services and workflows**.

It allows you to define a sequence of steps and manage their execution automatically.

### Key Features

* Visual workflow designer
* Error handling and retries
* Integration with Lambda, ECS, SQS, SNS, DynamoDB, etc.
* State machine-based execution

### Example

Online Order Processing:

1. Receive order
2. Process payment
3. Check inventory
4. Ship product
5. Send notification

Step Functions coordinates all these steps.

### Benefits

* Reduces complex application logic
* Built-in fault tolerance
* Easy monitoring of workflow execution

### Interview One-Liner

**AWS Step Functions is a workflow orchestration service that coordinates multiple AWS services into automated business processes.**


### AWS App Runner

**Purpose:** Deploy web applications and APIs without managing servers.

**Uses:**

* Host Spring Boot applications
* Deploy REST APIs
* Run containerized applications

**Key Features:**

* Automatic scaling
* HTTPS by default
* CI/CD integration
* Simple deployment from source code or container image

**Example:**
Deploy a Spring Boot application directly from a GitHub repository, and App Runner automatically builds, deploys, and scales it.

**Short Note:**
AWS App Runner is a fully managed service that makes it easy to deploy and run web applications and APIs without managing infrastructure.
