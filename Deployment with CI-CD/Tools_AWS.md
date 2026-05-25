
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
