
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

