## 1. IAM (Identity and Access Management)

IAM is the global AWS security system.

It answers:
> Who can do what on which AWS resources?

### What IAM does
- Manages users, groups, roles, and policies
- Controls permissions across all AWS services
- Uses JSON policies to allow or deny actions

### In SageMaker context
IAM defines:
- Who can create notebooks, training jobs, endpoints
- What S3 data SageMaker can read/write
- Whether SageMaker can access other AWS services

IAM exists outside SageMaker and applies to the whole AWS account.

---

## 2. IAM Role (in SageMaker)

A role is a specific IAM identity that SageMaker assumes.

It answers:
> What permissions does SageMaker have when it runs something?

> a list of permissions that a use can do.

### What a SageMaker role does
- Grants SageMaker permission to access S3, CloudWatch, ECR, etc.
- Is assumed automatically by SageMaker services

### Common SageMaker roles
- Notebook execution role
- Training job role
- Endpoint (inference) role

Policies are attached to roles, not directly to SageMaker jobs.

---

## 3. SageMaker Domain

A SageMaker Domain is a managed ML workspace environment.

It answers:
> How do users access SageMaker Studio and collaborate?

### What a Domain does
- Hosts SageMaker Studio
- Manages users, notebooks, and apps
- Integrates with IAM, VPC, and user profiles

### Key concepts inside a Domain
- Domain: top-level environment
- User Profile: individual user
- Execution Role: IAM role attached to users/apps

---

## How They Work Together

IAM provides security policies.
Roles grant permissions to SageMaker services.
Domains organize users and Studio environments.

---

## Comparison Table

| Concept | What it is | Scope | Purpose |
|------|-----------|------|--------|
| IAM | AWS-wide security system | Entire AWS account | Define permissions |
| IAM Role | Permission container | AWS services | Let SageMaker access resources |
| SageMaker Domain | ML workspace | SageMaker Studio | User & app management |

---

## Analogy

- IAM = Security office
- Role = Access badge
- Domain = ML department
- User Profile = Employee desk


## What is a VPC?

A **VPC (Virtual Private Cloud)** is a logically isolated virtual
network in AWS where you define: - IP address ranges - Subnets
(public/private) - Route tables - Security rules (security groups &
NACLs)

Think of it as **your own private data center in AWS**.

------------------------------------------------------------------------

## Why use a VPC with SageMaker?

By default, SageMaker runs **outside your VPC** and accesses data over
the internet.\
Using a VPC allows you to:

-   Keep data **private**
-   Prevent internet access
-   Access **private resources** (RDS, Redshift, EC2)
-   Meet **security & compliance** requirements

------------------------------------------------------------------------

## How SageMaker uses a VPC

When you attach SageMaker to a VPC, AWS creates **Elastic Network
Interfaces (ENIs)** in your subnets.

These ENIs allow SageMaker to: - Read/write data from private
resources - Communicate securely inside your VPC

------------------------------------------------------------------------

## SageMaker resources that support VPC

You can attach a VPC to: - Notebook Instances - Studio Domains -
Training Jobs - Processing Jobs - Endpoints (real-time inference)

------------------------------------------------------------------------

## Key VPC components used by SageMaker

### 1. Subnets

-   Private subnets are recommended
-   Must have sufficient IP addresses
-   Can be in multiple Availability Zones

### 2. Security Groups

Control traffic: - Inbound (usually restricted) - Outbound (allow S3,
ECR, CloudWatch)

### 3. VPC Endpoints (Very Important)

To avoid internet access, create: - S3 Gateway Endpoint - ECR Interface
Endpoint - CloudWatch Logs Endpoint - STS Endpoint

This lets SageMaker access AWS services **without NAT or internet**.

------------------------------------------------------------------------

## Example: Secure SageMaker Training in a VPC

1.  Training job runs in private subnet
2.  No internet gateway
3.  Access S3 via VPC endpoint
4.  Logs sent to CloudWatch
5.  Model artifacts stored securely in S3

Result: **Fully private ML workflow**

------------------------------------------------------------------------

## When should you use VPC with SageMaker?

Use it if you: - Handle sensitive data (PII, PHI) - Need compliance
(HIPAA, SOC, PCI) - Access private databases - Want zero internet
exposure

------------------------------------------------------------------------

## Simple analogy

**SageMaker without VPC** → Working in a public café\
**SageMaker with VPC** → Working in a locked office with controlled
access

-----

# how to attach a VPC to a sagemaker studio?
To attach a VPC to Amazon SageMaker Studio, you do it when creating the SageMaker Studio Domain (or by updating the domain). Studio runs inside the VPC you choose.


Key concept (important)

- 👉 SageMaker Studio is VPC-bound at the Domain level, not per notebook.
- One Studio Domain = one VPC
- All users and apps in that domain use the same VPC
- You cannot “attach later” without updating or recreating the domain