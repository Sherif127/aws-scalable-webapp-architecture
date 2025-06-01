# 🏗️ AWS Architecture: Scalable Web Application

## 📚 Content:
- [🔍 High-level Overview](#high-level-overview)
- [🧰 Key AWS Services Used](#key-aws-services-used)
- [📝 Notes](#notes)
- [🔐 IAM Roles](#iam-roles)
- [🛡️ Security Groups](#security-groups)
- [🔄 Project Flow](#project-flow)

---

## 🔍 High-level Overview

![Architecture](https://github.com/user-attachments/assets/684c65be-37e9-4f91-9fe2-ec92589f3373)

This architecture represents a **highly available**, **scalable**, and **secure** web application hosted on AWS.  
It is designed with separation between frontend, backend, and database layers, and includes auto scaling, monitoring, and failover mechanisms.

---

## 🧰 Key AWS Services Used

- **EC2:** For hosting frontend and backend servers (split across private subnets).
- **Application Load Balancer (ALB):** Distributes traffic between instances on both web and app layers.
- **Auto Scaling Group (ASG):** Automatically scales EC2 instances based on load.
- **Amazon RDS:** Managed relational database with Multi-AZ & Read Replica.
- **Amazon VPC:** Isolated networking environment with public and private subnets.
- **NAT Gateway:** Allows private EC2 instances to access the internet securely.
- **Bastion Host:** For secure SSH access to private EC2s.
- **CloudWatch:** For monitoring system health and metrics.
- **SNS:** Sends email notifications for events and alarms.

---

## 📝 Notes

- **NAT Gateway** is used to allow EC2 instances in private subnets to access the internet (e.g., for updates or package installs).
- **Frontend and Backend servers** are deployed in separate **Auto Scaling Groups** in different Availability Zones for fault tolerance.
- **Application ports are customizable**, and in this deployment, we’re using port **8080** instead of the default 5000.
- **Bastion Host** is deployed in a public subnet to enable SSH access to internal resources.
- **RDS Read Replica** ensures better performance for read-heavy operations and helps offload the primary DB.

---

## 🔐 IAM Roles

| Component    | IAM Role |
|--------------|----------|
| ALB          | `AWSElasticLoadBalancingServiceRolePolicy` |
| Auto Scaling | `AWSServiceRoleForAutoScaling` |
| CloudWatch   | `CloudWatchReadOnlyAccess` |

---

## 🛡️ Security Groups

| Component     | Inbound Traffic                                  | Outbound Traffic                                 |
|---------------|--------------------------------------------------|--------------------------------------------------|
| **ALB (Web Tier)** | `0.0.0.0/0` : **80**, **443**               | Web EC2 subnet range : **80**, **443**           |
| **Web EC2**       | ALB IP range : **80**, **443**            | IP of ALB (App Tier): **8080**, `0.0.0.0/0 via NAT`       |
| **ALB (App Tier)**| Web EC2 subnet range : **8080**                     | App EC2 subnet range : **8080**                         |
| **App EC2**       | App ALB IP range : **8080**                  | RDS subnet range : **3306**, `0.0.0.0/0 via NAT` |
| **RDS (MySQL)**   | App EC2 subnet range : **3306**              | -                                                |

---

## 🔄 Project Flow

1. A user accesses the application via a public DNS linked to the Web ALB.
2. The request enters through the **Internet Gateway** into the **Public Subnet** where the **ALB** resides.
3. The **Web ALB** routes traffic to the **frontend EC2 instances** within private subnets and managed by an Auto Scaling Group.
4. The **frontend servers** interact with the **App ALB** on port **8080**, which routes requests to the **backend servers**.
5. The **backend EC2 instances** query the **Amazon RDS** database hosted in private subnets.
6. **Multi-AZ deployment** ensures high availability for the primary DB, while **Read Replica** serves read requests.
7. **CloudWatch** monitors system metrics and triggers alarms when needed.
8. **SNS** is integrated with CloudWatch to notify the admin via email about any critical system events.
9. If needed, the **bastion host** provides SSH access to internal EC2 instances through a secure connection.
10. **Private EC2 instances** use the **NAT Gateway** to access the internet for updates or outbound traffic.
---

