
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


# AWS IAM (Identity and Access Management)

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
