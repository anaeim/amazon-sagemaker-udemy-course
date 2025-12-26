
# AWS SageMaker IAM, Roles, Users, and Domains – Explained

## 1. IAM (Identity and Access Management) – the foundation

IAM answers **“who can do what on which AWS resources.”**

### Core IAM concepts relevant to SageMaker
- **IAM Users** → People or services that log in
- **IAM Roles** → Permission lists (containers) that can be assumed
- **IAM Policies** → Rules that define allowed/denied actions

---

## 2. IAM Users

### What is an IAM User?
An **IAM User** represents a **human or application** that needs to authenticate to AWS.

**Key characteristics**
- Long-term credentials
  - Console password
  - Access keys (CLI/SDK)
- No permissions by default
- Permissions granted via:
  - Policies
  - IAM Groups

### In SageMaker context
IAM Users are typically:
- ML engineers
- Data scientists
- Administrators

They:
- Log into AWS Console
- Access SageMaker Studio
- Assume IAM roles

> **Best practice:** IAM Users should not directly access AWS services. Use roles instead.

**Q.** What does it mean to add policies to an IAM role?<br>
A. Policies (rule) define what the role is allowed (or denied) to do.<br>
When something assumes the role, it can perform only the actions allowed by those policies.

---

## 3. IAM Roles

### What is an IAM Role?
An **IAM Role** is a temporary list (set) of permissions that:
- Has no long-term credentials
- Is assumed using AWS STS
- Can be assumed by users or AWS services

### Why roles are critical in SageMaker
SageMaker workloads must access:
- S3
- ECR
- CloudWatch
- KMS

They do this by **assuming IAM roles**.

### SageMaker Execution Role
Used by:
- Training jobs
- Processing jobs
- Endpoints
- Studio notebooks

Example trust policy:
```json
{
  "Effect": "Allow",
  "Principal": { "Service": "sagemaker.amazonaws.com" },
  "Action": "sts:AssumeRole"
}
```

---

## 4. SageMaker Domain

### What is a SageMaker Domain?
A **SageMaker Domain** is a managed workspace for SageMaker Studio. <br>
A SageMaker Domain is a logical boundary for SageMaker Studio.

It defines:
- Authentication
- Networking (VPC)
- Storage (EFS)
- Default execution roles

### Domain structure
```
SageMaker Domain
├── User Profiles
├── Apps (Studio, Jupyter, KernelGateway)
└── Shared EFS Storage
```

---

## 5. SageMaker User Profiles

### What is a User Profile?
A **User Profile**:
- Represents one person in Studio
- Is not an IAM User
- Is attached (mapped) to an IAM execution role

Each profile has:
- Execution role
- Home directory in EFS
- App configurations

### IAM User vs User Profile

| IAM User | SageMaker User Profile |
|--------|------------------------|
| AWS identity | SageMaker identity |
| Console login | Studio login |
| Global scope | Domain-scoped |
| Assumes roles | Uses execution role |

---

## 6. How everything works together

### End-to-end flow
```
Human
↓
IAM User
↓
Assume Role
↓
SageMaker Domain
↓
User Profile
↓
Execution Role
↓
AWS Services (S3, ECR, Logs)
```

---

## 7. Hierarchy Overview

```
AWS Account
├── IAM
│   ├── Users
│   └── Roles
├── SageMaker
│   ├── Domain
│   │   └── User Profiles
│   │       └── Execution Roles
└── AWS Services (S3, ECR, CloudWatch)
```

---

## 8. Security Best Practices

- Least privilege
- Role-based access
- Temporary credentials
- Separation of human and service access

---

## 9. Simple Mental Model

- **IAM User** → Who you are
- **IAM Role** → What you can do
- **Domain** → Where you work
- **User Profile** → Your workspace
- **Execution Role** → Your AWS access badge