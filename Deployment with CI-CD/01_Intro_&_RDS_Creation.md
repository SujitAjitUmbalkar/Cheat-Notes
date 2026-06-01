## Short Notes on Amazon Web Services

* AWS is a cloud computing platform provided by Amazon.
  It offers servers, storage, databases, networking, and security services online.

* AWS follows a pay-as-you-go model.
  Users pay only for the resources they use.
* AWS provides scalable infrastructure 

* It helps developers deploy applications without buying physical hardware.
  Applications can run from anywhere using internet-based cloud servers.

* AWS provides scalable infrastructure.
  Resources can increase or decrease based on application traffic.

* EC2 is used for virtual servers in AWS.
  Developers host websites, APIs, and applications on EC2 instances.

* RDS is AWS’s managed relational database service.
  It supports MySQL, PostgreSQL, Oracle, and other databases.

* S3 is cloud storage service in AWS.
  It stores files, backups, images, and application data.

* Lambda allows serverless computing in AWS.
  Code runs without managing servers manually.

* AWS Free Tier helps beginners learn cloud services.
  It provides limited free usage for selected services.

* AWS is widely used in DevOps and modern software development.
  Many companies use it for hosting scalable applications.


---

## RDS Creation Concepts (Short Notes)

* **Engine Type** → Selects the database system like MySQL or PostgreSQL.
  Different engines support different features and SQL behavior.

* **Version** → Chooses the specific version of database software.
  Newer versions may provide better security and performance.

* **Templates** → AWS provides presets like Free Tier or Production.
  It automatically configures settings based on your use case.

* **DB Instance Identifier** → Unique name given to the RDS instance.
  Used to identify the database inside AWS.

* **Master Username** → Admin username for database access.
  It is used to login and manage the database.

* **Master Password** → Password for the admin account.
  Required while connecting applications to RDS.

* **DB Instance Class** → Defines CPU, RAM, and processing power.
  Larger instance classes provide better performance.

* **Storage Type** → Type of storage disk used by database.
  SSD storage gives faster performance than HDD.

* **Allocated Storage** → Amount of disk space assigned to database.
  Used for storing tables, records, logs, and backups.

* **Availability Zone** → Physical AWS data center where DB runs.
  Helps improve reliability and reduce downtime.

* **Multi-AZ** → Creates a standby copy in another zone.
  Used for automatic failover during failures.

* **VPC** → Private network environment for AWS resources.
  Controls communication between EC2 and RDS securely.

* **Subnet Group** → Group of subnets where RDS can be deployed.
  Helps AWS place DB across different network locations.

* **Public Access** → Allows internet connection to database if enabled.
  Usually disabled for better security in production.

* **Security Group** → Acts like firewall rules for RDS access.
  Controls which IPs or servers can connect to database.

* **Port** → Communication endpoint for database connections.
  Example: MySQL uses port 3306.

* **Database Name** → Initial database created inside RDS.
  Applications use this DB for storing data.

* **Backup Retention** → Number of days AWS stores automatic backups.
  Helps restore data after accidental loss.

* **Monitoring** → Tracks database health and performance metrics.
  Useful for identifying CPU or memory issues.

* **Encryption** → Protects stored database data using encryption.
  Improves security and data privacy.

* **Maintenance Window** → Scheduled time for AWS updates and patches.
  AWS performs maintenance during this period.

* **Deletion Protection** → Prevents accidental deletion of database.
  Extra safety feature for important production DBs.

---

## Inbound Rules

Inbound rules control incoming traffic to AWS resources like Amazon EC2 or Amazon RDS.

They define:

* who can connect
* which port is allowed
* what type of traffic can enter

Example:

* Allow MySQL traffic on port 3306 from your IP.

---

## Outbound Rules

Outbound rules control outgoing traffic from AWS resources.

They define:

* where the resource can send data
* which ports/protocols are allowed outside

Example:

* Allow database server to send responses to applications or internet services.

## Steps to Edit Inbound/Outbound Rules in AWS

1. Open AWS Console
2. Go to Amazon RDS or Amazon EC2
3. Select your resource
4. Open **Connectivity & Security** section
5. Click the **Security Group** link
6. Security Group page will open
7. Go to:

   * **Inbound Rules** tab OR
   * **Outbound Rules** tab
8. Click **Edit Rules**
9. Add/Edit/Delete rules
10. Click **Save Rules**

---
Security groups act like firewalls for AWS resources.
